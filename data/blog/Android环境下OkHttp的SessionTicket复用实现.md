---
date: 2019-10-15
title: Android环境下OkHttp的SessionTicket复用实现
tags: [SSL, Session-Resumption, SessionTicket, OkHttp, Android]
---

在文章[SSL Handshake Session Resumption](/2019/10/10/SSL-Handshake-Session-Resumption.html)中介绍了SessionTicket的复用技术原理，从中我们得知SessionTicket复用是在客户端缓存的SessionTicket，服务端只是验证客户端传过去的SessionTicket是否有效，因此不同的客户端实现库，对SessionTicket的缓存机制有差异。本文介绍Android系统上OKHttp客户端的SessionTicket缓存实现。

## 概述

Android系统上作为客户端通过HTTPS请求服务端资源，客户端由3部分构成：Android系统运行时、HTTPClient、SSL库。本文参考的源代码，对应的Android系统运行时为[api-level-28](https://github.com/AndroidSDKSources/android-sdk-sources-for-api-level-28)，HTTPClient为[OKHttp-latest](https://github.com/square/okhttp)，SSL库为[conscrypt](https://source.android.google.cn/devices/architecture/modular-system/conscrypt)(conscrypt底层SSL库是[BoringSSL](https://boringssl.googlesource.com/boringssl/)，谷歌基于OpenSSL fork出来的实现)。

Android系统基于上述SSL库，默认的行为是启动SessionTicket，SessionTicket缓存在内存中，支持SessionTicket缓存到文件。要使用SessionTicket缓存到文件的功能，设置全局属性```org.conscrypt.Conscrypt.setClientSessionCache(SSLContext context, SSLClientSessionCache cache)```，其中```SSLClientSessionCache```选择```org.conscrypt.FileClientSessionCache.Impl```。

## 默认开启SessionTicket复用

### 1. RealConnection.kt: 初始化 SSL Socket

```kotlin
  fun connect(
    connectTimeout: Int,
    readTimeout: Int,
    writeTimeout: Int,
    pingIntervalMillis: Int,
    connectionRetryEnabled: Boolean,
    call: Call,
    eventListener: EventListener
  ) {
        ...

        if (route.requiresTunnel()) {
          connectTunnel(connectTimeout, readTimeout, writeTimeout, call, eventListener)
          if (rawSocket == null) {
            // We were unable to connect the tunnel but properly closed down our resources.
            break
          }
        } else {
          connectSocket(connectTimeout, readTimeout, call, eventListener)
        }
        establishProtocol(connectionSpecSelector, pingIntervalMillis, call, eventListener)
        eventListener.connectEnd(call, route.socketAddress, route.proxy, protocol)
        ...
  }
```

```kotlin

private fun establishProtocol
    connectTls(connectionSpecSelector)

```

```kotlin
private fun connectTls(connectionSpecSelector: ConnectionSpecSelector)
      // Configure the socket's ciphers, TLS versions, and extensions.
      val connectionSpec = connectionSpecSelector.configureSecureSocket(sslSocket)
      if (connectionSpec.supportsTlsExtensions) {
        Platform.get().configureTlsExtensions(sslSocket, address.protocols)
      }
```

### 2. Platform.kt

```kotlin
  companion object {
    @Volatile private var platform = findPlatform()

    /** Attempt to match the host runtime to a capable Platform implementation. */
    private fun findPlatform(): Platform {
      val android10 = Android10Platform.buildIfSupported()

      if (android10 != null) {
        return android10
      }

      val android = AndroidPlatform.buildIfSupported()

      if (android != null) {
        return android
      }
```

### 3. AndroidPlatform.kt

```kotlin
/** Android 5+. */
class AndroidPlatform : Platform() {
  private val socketAdapters = listOfNotNull(
      StandardAndroidSocketAdapter.buildIfSupported(),
      ConscryptSocketAdapter.buildIfSupported(),
      DeferredSocketAdapter("com.google.android.gms.org.conscrypt")
  ).filter { it.isSupported() }

  override fun configureTlsExtensions(
    sslSocket: SSLSocket,
    protocols: List<@JvmSuppressWildcards Protocol>
  ) {
    // No TLS extensions if the socket class is custom.
    socketAdapters.find { it.matchesSocket(sslSocket) }
        ?.configureTlsExtensions(sslSocket, protocols)
  }
```

### 4. ConscryptSocketAdapter.kt

```kotlin
  override fun configureTlsExtensions(
    sslSocket: SSLSocket,
    protocols: List<Protocol>
  ) {
    // No TLS extensions if the socket class is custom.
    if (matchesSocket(sslSocket)) {
      // Enable session tickets.
      Conscrypt.setUseSessionTickets(sslSocket, true)

      // Enable ALPN.
      val names = Platform.alpnProtocolNames(protocols)
      Conscrypt.setApplicationProtocols(sslSocket, names.toTypedArray())
    }
  }
```

### 5. org.conscrypt.OpenSSLSocketImpl.java

```java
public abstract void setUseSessionTickets(boolean useSessionTickets);
```

### 6. org.conscrypt.ConscryptEngineSocket extends OpenSSLSocketImpl

```java
@Override
public final void setUseSessionTickets(boolean useSessionTickets) {
    engine.setUseSessionTickets(useSessionTickets);
}
```

## 在SSL握手时复用SessionTicket

### 1. RealConnection.kt

```kotlin
private fun connectTls(connectionSpecSelector: ConnectionSpecSelector){
    ...

          // Force handshake. This can throw!
      sslSocket.startHandshake()
      // block for session establishment
      val sslSocketSession = sslSocket.session
      val unverifiedHandshake = sslSocketSession.handshake()

    ...
}
```

### 2. sslSocket 对象就是 OpenSSLSocketImpl，实现类是 org.conscrypt.ConscryptEngineSocket

```java
public final void startHandshake() throws IOException {
    ...

    if (state == STATE_NEW) {
    state = STATE_HANDSHAKE_STARTED;
    engine.beginHandshake();
    in = new SSLInputStream();
    out = new SSLOutputStream();

    ...
}
```

### 3. ConscryptEngine.java

```java
    @Override
    public void beginHandshake() throws SSLException {
        synchronized (ssl) {
            beginHandshakeInternal();
        }
    }

   private void beginHandshakeInternal() throws SSLException {
       ...

               try {
            // Prepare the SSL object for the handshake.
            ssl.initialize(getHostname(), channelIdPrivateKey);

            // For clients, offer to resume a previously cached session to avoid the
            // full TLS handshake.
            if (getUseClientMode()) {
                NativeSslSession cachedSession = clientSessionContext().getCachedSession(
                        getHostname(), getPeerPort(), sslParameters);
                if (cachedSession != null) {
                    cachedSession.offerToResume(ssl);
                }
            }

            maxSealOverhead = ssl.getMaxSealOverhead();
            handshake();
        
       ...
   }

    private ClientSessionContext clientSessionContext() {
        return sslParameters.getClientSessionContext();
    }
```

### 4. org.conscrypt.SSLParametersImpl 

构造函数中传递了org.conscrypt.ClientSessionContext clientSessionContext对象

### 5. org.conscrypt.ClientSessionContext.java

```java
    /**
     * Gets the suitable session reference from the session cache container.
     */
    synchronized NativeSslSession getCachedSession(String hostName, int port,
            SSLParametersImpl sslParameters) {
        ...
        NativeSslSession session = getSession(hostName, port);
        if (session == null) {
            return null;
        }

        ...

        if (session.isSingleUse()) {   
            removeSession(session);
        }
        return session;

        ...
    }

    /**
     * Finds a cached session for the given host name and port.
     *
     * @param host of server
     * @param port of server
     * @return cached session or null if none found
     */
    private NativeSslSession getSession(String host, int port) {
        ...

        //先从内存读
        synchronized (sessionsByHostAndPort) {
            List<NativeSslSession> sessions = sessionsByHostAndPort.get(key);
            if (sessions != null && sessions.size() > 0) {
                session = sessions.get(0);
            }
        }

        ...

        //内存没有，则从持久化存储读

        // Look in persistent cache.  We don't currently delete sessions from the persistent
        // cache, so we may find a multi-use (aka TLS 1.2) session after having received and
        // then used up one or more single-use (aka TLS 1.3) sessions.
        if (persistentCache != null) {
            byte[] data = persistentCache.getSessionData(host, port);
            ...
        }

        ...
    }
```

### 6. ClientSessionContext.java

保存session时候，只有multi-use的session才会存储到文件，single-use的session只存储到内存

```java
    @Override
    void onBeforeAddSession(NativeSslSession session) {
        String host = session.getPeerHost();
        int port = session.getPeerPort();
        if (host == null) {
            return;
        }

        HostAndPort key = new HostAndPort(host, port);
        putSession(key, session);

        // TODO: Do this in a background thread.
        if (persistentCache != null && !session.isSingleUse()) {
            byte[] data = session.toBytes();
            if (data != null) {
                persistentCache.putSessionData(session.toSSLSession(), data);
            }
        }
    }
```

### 7. FileClientSessionCache.java

持久化存储，一个 FileClientSessionCache 对应一个目录，最多存储 **12** 个SessionTicket，超过之后按照 LRU 算法删除一个。

```java
/**
 * File-based cache implementation. Only one process should access the
 * underlying directory at a time.
 */
@Internal
public final class FileClientSessionCache {
    private static final Logger logger = Logger.getLogger(FileClientSessionCache.class.getName());

    public static final int MAX_SIZE = 12; // ~72k
...
        @Override
        public synchronized void putSessionData(SSLSession session, byte[] sessionData) {
            String host = session.getPeerHost();
            if (sessionData == null) {
                throw new NullPointerException("sessionData == null");
            }

            String name = fileName(host, session.getPeerPort());
            File file = new File(directory, name);

            // Used to keep track of whether or not we're expanding the cache.
            boolean existedBefore = file.exists();

            FileOutputStream out;
            try {
                out = new FileOutputStream(file);
            } catch (FileNotFoundException e) {
                // We can't write to the file.
                logWriteError(host, file, e);
                return;
            }

            // If we expanded the cache (by creating a new file)...
            if (!existedBefore) {
                size++;

                // Delete an old file if necessary.
                makeRoom();
            }

            boolean writeSuccessful = false;
            try {
                out.write(sessionData);
                writeSuccessful = true;
            } catch (IOException e) {
                logWriteError(host, file, e);
            } finally {
                boolean closeSuccessful = false;
                try {
                    out.close();
                    closeSuccessful = true;
                } catch (IOException e) {
                    logWriteError(host, file, e);
                } finally {
                    if (!writeSuccessful || !closeSuccessful) {
                        // Storage failed. Clean up.
                        delete(file);
                    } else {
                        // Success!
                        accessOrder.put(name, file);
                    }
                }
            }
        }
...
```

## Reference

[Conscrypt](https://source.android.google.cn/devices/architecture/modular-system/conscrypt)
