---
date: 2020-4-15
title: GameAnywhere代码结构
tags: [游戏, GameAnywhere, 云游戏]
---

[GameAnywhere](https://github.com/chunying/gaminganywhere) 是一个开源的云游戏平台，由 **Chun-Ying Huang** 在2013年开发，最开始是毕业论文研究使用，近期随着云游戏风口正盛，这个项目关注度又有起色，本文介绍GameAnywhere代码构成。GameAnywhere 的License为 [BSD3](https://github.com/chunying/gaminganywhere/blob/master/LICENSE)，可以修改代码后闭源，但是项目依赖的开源组件较多，需要注意 License。

## 代码构成

### 目录结构

```html
├─bin
│  ├─config
│  │  └─common
│  ├─data
│  ├─log
│  └─mod
├─deps.pkg.win32      #依赖组件的windows包
│  ├─bin
│  ├─NvCodec
│  └─NVENC
├─deps.posix          #依赖组件的unix包
│  └─lib
├─deps.src            #依赖组件的源码
│  └─patches
├─deps.win32          #依赖组件的Windows头文件与库
│  ├─bin
│  ├─include
│  │  ├─include
│  │  ├─lib
│  │  ├─libavcodec
│  │  ├─libavdevice
│  │  ├─libavfilter
│  │  ├─libavformat
│  │  ├─libavutil
│  │  ├─libpostproc
│  │  ├─libswresample
│  │  ├─libswscale
│  │  ├─live555
│  │  ├─NVENC
│  │  └─SDL2
│  └─lib
├─docs
└─ga
    ├─android   # android客户端
    ├─client    # windows客户端
    ├─core      # 公共依赖部分
    ├─module    # 各个功能模块
    │  ├─asource-system   # apple 平台的音频抓取
    │  ├─ctrl-sdl         # 键盘输入
    │  ├─encoder-audio    # 音频编码
    │  ├─encoder-mfx
    │  ├─encoder-nvenc    # NVIDIA 硬编码
    │  ├─encoder-video    # FFmpeg 软编码
    │  ├─encoder-x264     # 编码x264库
    │  ├─filter-rgb2yuv   # rgb转yuv库，编码前预处理
    │  ├─server-ffmpeg    # FFmpeg库
    │  ├─server-live555   # live555 项目，承载 rtsp 协议
    │  └─vsource-desktop  # DirectX GDI 抓图
    ├─server
    │  ├─event-driven     # 键盘鼠标输入windows平台实现
    │  ├─event-posix      # 键盘鼠标输入posix平台实现
    │  └─periodic         # 可执行文件 Main 函数，依赖 ga/core
    └─vs2010    #Visual Studio工程
        ├─BasicUsageEnvironment
        ├─CloudGameDM_Installer
        ├─encoder-nvenc
        ├─ga-client
        ├─ga-hook
        ├─ga-server-event-driven
        ├─ga-server-periodic
        ├─groupsock
        ├─ipch
        ├─libga
        ├─liveMedia
        ├─module-asource-system
        ├─module-ctrl-sdl
        ├─module-encoder-audio
        ├─module-encoder-video
        ├─module-encoder-x264
        ├─module-filter-rgb2yuv
        ├─module-server-ffmpeg
        ├─module-server-live555
        ├─module-vsource-desktop
        ├─module-vsource-desktop-d3d
        ├─module-vsource-desktop-dfm
        └─UsageEnvironment
```

### 服务端

#### Main: server\ga\server\periodic\ga-server-periodic.cpp

Server 端的功能：图像采集、图像预处理、声音采集、键盘输入、音视频编码、封包、传输功能，这些功能分别由对应GameAnywhere定义的模块实现。

```c
//
if (load_modules() < 0) {
  write_log("error, load_modules error ! \n");
  return -1;
}
if (init_modules() < 0) {
  write_log("error, init_modules error ! \n");
  return -1;
}
if (run_modules() < 0) {
  write_log("error, run_modules error ! \n");
  return -1;
}

```

GameAnywhere 模块类型定义: server\ga\core\ga-module.h

```c
/**
 * Enumeration for types of a module.
 */
enum ga_module_types {
	GA_MODULE_TYPE_NULL = 0,	/**< Not used */
	GA_MODULE_TYPE_CONTROL,		/**< Is a controller module */
	GA_MODULE_TYPE_ASOURCE,		/**< Is an audio source module */
	GA_MODULE_TYPE_VSOURCE,		/**< Is an video source module */
	GA_MODULE_TYPE_FILTER,		/**< Is a filter module */
	GA_MODULE_TYPE_AENCODER,	/**< Is an audio encoder module */
	GA_MODULE_TYPE_VENCODER,	/**< Is a video encoder module */
	GA_MODULE_TYPE_ADECODER,	/**< Is an audio decoder module */
	GA_MODULE_TYPE_VDECODER,	/**< Is a video decoder module */
	GA_MODULE_TYPE_SERVER		/**< Is a server module */
};
```

GameAnywhere 模块定义: server\ga\core\ga-module.h

```c
/**
 * Data strucure to represent a module.
 */
typedef struct ga_module_s {
	HMODULE	handle;		/**< Handle to a module */
	int type;		/**< Type of the module */
	char *name;		/**< Name of the module */
	char *mimetype;		/**< MIME-type of the module */
	int (*init)(void *arg);		/**< Pointer to the init function */
	int (*start)(void *arg);	/**< Pointer to the start function */
	//void * (*threadproc)(void *arg);
	int (*stop)(void *arg);		/**< Pointer to the stop function */
	int (*deinit)(void *arg);	/**< Pointer to the deinit function */
	int (*ioctl)(int command, int argsize, void *arg);	/**< Pointer to ioctl function */
	int (*notify)(void *arg);	/**< Pointer to the notify function */
	void * (*raw)(void *arg, int *size);	/**< Pointer to the raw function */
	int (*send_packet)(const char *prefix, int channelId, AVPacket *pkt, int64_t encoderPts, struct timeval *ptv);	/**< Pointer to the send packet function: sink only */
	void * privdata;		/**< Private data of this module */
}	ga_module_t;
```

模块生命周期动作: server\ga\core\ga-module.h

```c
EXPORT ga_module_t * ga_load_module(const char *modname, const char *prefix);
EXPORT void ga_unload_module(ga_module_t *m);
EXPORT int ga_init_single_module(const char *name, ga_module_t *m, void *arg);
EXPORT void ga_init_single_module_or_quit(const char *name, ga_module_t *m, void *arg);
EXPORT int ga_run_single_module(const char *name, void * (*threadproc)(void*), void *arg);
EXPORT void ga_run_single_module_or_quit(const char *name, void * (*threadproc)(void*), void *arg);
// module function wrappers
EXPORT int ga_module_init(ga_module_t *m, void *arg);
EXPORT int ga_module_start(ga_module_t *m, void *arg);
EXPORT int ga_module_stop(ga_module_t *m, void *arg);
EXPORT int ga_module_deinit(ga_module_t *m, void *arg);
EXPORT int ga_module_ioctl(ga_module_t *m, int command, int argsize, void *arg);
EXPORT int ga_module_notify(ga_module_t *m, void *arg);
EXPORT void * ga_module_raw(ga_module_t *m, void *arg, int *size);
EXPORT int ga_module_send_packet(ga_module_t *m, const char *prefix, int channelId, AVPacket *pkt, int64_t encoderPts, struct timeval *ptv);
```


#### 图像采集模块: server\ga\module\vsource-desktop

此模块提供**桌面级别**图像采集功能，未提供**进程级别**图像采集。提供```DirectX```与```GDI```两种图像采集实现。

```c
ga_module_t *
module_load() {
	static ga_module_t m;
	bzero(&m, sizeof(m));
	m.type = GA_MODULE_TYPE_VSOURCE;
	m.name = strdup("vsource-desktop");
	m.init = vsource_init;
	m.start = vsource_start;
	m.stop = vsource_stop;
	m.deinit = vsource_deinit;
	m.ioctl = vsource_ioctl;
	return &m;
}
```

#### 图像预处理模块: server\ga\module\filter-rgb2yuv\filter-rgb2yuv.cpp

此模块提供将抓取到的RGB格式图片转换为YUV格式。

```c
ga_module_t *
module_load() {
	static ga_module_t m;
	bzero(&m, sizeof(m));
	m.type = GA_MODULE_TYPE_FILTER;
	m.name = strdup("filter-RGB2YUV");
	m.init = filter_RGB2YUV_init;
	m.start = filter_RGB2YUV_start;
	m.stop = filter_RGB2YUV_stop;
	m.deinit = filter_RGB2YUV_deinit;
	//m.threadproc = filter_RGB2YUV_threadproc;
	return &m;
}
```

#### NVIDIA硬编码模块: server\ga\module\encoder-nvenc\encoder-nvenc.cpp

```c
ga_module_t *
module_load() {
	static ga_module_t m;
	struct RTSPConf *rtspconf = rtspconf_global();
	//
	bzero(&m, sizeof(m));
	m.type = GA_MODULE_TYPE_VENCODER;
	m.name = strdup("nvenc-video-encoder");
	m.mimetype = strdup("video/H264");
	m.init  = nvenc_init;
	m.deinit= nvenc_deinit;
	m.start = nvenc_start;
	m.ioctl = nvenc_ioctl;
	m.stop  = nvenc_stop;

	return &m;
}
```

#### 软编码模块(ffmpge实现): server\ga\module\encoder-video\encoder-video.cpp

```c
ga_module_t *
module_load() {
	static ga_module_t m;
	//struct RTSPConf *rtspconf = rtspconf_global();
	char mime[64];
	//
	bzero(&m, sizeof(m));
	m.type = GA_MODULE_TYPE_VENCODER;
	m.name = strdup("ffmpeg-video-encoder");
	if(ga_conf_readv("video-mimetype", mime, sizeof(mime)) != NULL) {
		m.mimetype = strdup(mime);
	}
	m.init = vencoder_init;
	m.start = vencoder_start;
	//m.threadproc = vencoder_threadproc;
	m.stop = vencoder_stop;
	m.deinit = vencoder_deinit;
	//
	m.raw = vencoder_raw;
	m.ioctl = vencoder_ioctl;
	return &m;
}
```

#### 软编码模块(x264实现): server\ga\module\encoder-x264\encoder-x264.cpp

```c
ga_module_t *
module_load() {
	static ga_module_t m;
	//
	bzero(&m, sizeof(m));
	m.type = GA_MODULE_TYPE_VENCODER;
	m.name = strdup("x264-video-encoder");
	m.mimetype = strdup("video/H264");
	m.init = vencoder_init;
	m.start = vencoder_start;
	//m.threadproc = vencoder_threadproc;
	m.stop = vencoder_stop;
	m.deinit = vencoder_deinit;
	//
	m.raw = vencoder_raw;
	m.ioctl = vencoder_ioctl;
	return &m;
}
```

#### 音频采集模块: server\ga\module\asource-system\asource-system.cpp

在windows操作系统上，使用微软的 [IAudioClient API](https://docs.microsoft.com/en-us/windows/win32/api/audioclient/nn-audioclient-iaudioclient) 采集声音。

```c
ga_module_t *
module_load() {
	static ga_module_t m;
	bzero(&m, sizeof(m));
	m.type = GA_MODULE_TYPE_ASOURCE;
	m.name = strdup("asource-system");
	m.init = asource_init;
	m.start = asource_start;
	//m.threadproc = asource_threadproc;
	m.stop = asource_stop;
	m.deinit = asource_deinit;
	return &m;
}
```

#### 音频编码模块: server\ga\module\encoder-audio\encoder-audio.cpp

使用 ffmpeg 封装的API编码音频，编码格式支持 LAME/OPUS 。

```c
ga_module_t *
module_load() {
	static ga_module_t m;
	struct RTSPConf *rtspconf = rtspconf_global();
	char mime[64];
	bzero(&m, sizeof(m));
	m.type = GA_MODULE_TYPE_AENCODER;
	m.name = strdup("ffmpeg-audio-encoder");
	if(ga_conf_readv("audio-mimetype", mime, sizeof(mime)) != NULL) {
		m.mimetype = strdup(mime);
	}
	m.init = aencoder_init;
	m.start = aencoder_start;
	//m.threadproc = aencoder_threadproc;
	m.stop = aencoder_stop;
	m.deinit = aencoder_deinit;
	return &m;
}
```

#### 媒体传输模块(ffmpge-server): server\ga\module\server-ffmpeg\server-ffmpeg.cpp

此模块不再使用。

```c
ga_module_t *
module_load() {
	static ga_module_t m;
	//
	bzero(&m, sizeof(m));
	m.type = GA_MODULE_TYPE_SERVER;
	m.name = strdup("ffmpeg-rtsp-server");
	m.init = ff_server_init;
	m.start = ff_server_start;
	m.stop = ff_server_stop;
	m.deinit = ff_server_deinit;
	m.send_packet = ff_server_send_packet;
	//
	encoder_register_sinkserver(&m);
	//
	return &m;
}
```

#### 音视频传输模块(live555): server\ga\module\server-live555\server-live555.cpp

当前使用此模块。Live555 存在性能优化空间，网上资料较多，可[参考](https://blog.csdn.net/yuanchunsi/article/details/78687516?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)。

```c
ga_module_t *
module_load() {
	static ga_module_t m;
	//
	bzero(&m, sizeof(m));
	m.type = GA_MODULE_TYPE_SERVER;
	m.name = strdup("live555-rtsp-server");
	m.init = live_server_init;
	m.start = live_server_start;
	m.stop = live_server_stop;
	m.deinit = live_server_deinit;
	m.send_packet = live_server_send_packet;
	//
	encoder_register_sinkserver(&m);
	//
	return &m;
}
```

#### 控制指令输入模块

此模块支持键盘、鼠标事件输入，基于 [SDL](https://www.libsdl.org/) 实现。

```c
ga_module_t *
module_load() {
	static ga_module_t m;
	bzero(&m, sizeof(m));
	m.type = GA_MODULE_TYPE_CONTROL;
	m.name = strdup("control-SDL");
	m.init = sdlmsg_replay_init;
	m.deinit = sdlmsg_replay_deinit;
	return &m;
}
```

### Android 客户端

客户端由C/C++ Native库与Java App组成。

- C/C++ Native: 提供触控指令传输、媒体流接收、解码、播放功能。
- Java App: 提供触控指令采集、以及云游戏App功能。

#### C/C++ Native 库 API (主入口): server\ga\android\jni\src\libgaclient.h

Native 库 JNI 函数定义。

1. 初始化

```c
JNIEXPORT jboolean JNICALL Java_org_gaminganywhere_gaclient_GAClient_initGAClient(
		JNIEnv *env, jobject thisObj, jobject weak_this);
```

2. 配置参数

```c
JNIEXPORT void JNICALL Java_org_gaminganywhere_gaclient_GAClient_resetConfig(
		JNIEnv *env, jobject thisObj);
JNIEXPORT void JNICALL Java_org_gaminganywhere_gaclient_GAClient_setProtocol(
		JNIEnv *env, jobject thisObj, jstring proto);
JNIEXPORT void JNICALL Java_org_gaminganywhere_gaclient_GAClient_setHost(
		JNIEnv *env, jobject thisObj, jstring host);
JNIEXPORT void JNICALL Java_org_gaminganywhere_gaclient_GAClient_setPort(
		JNIEnv *env, jobject thisObj, jint port);
JNIEXPORT void JNICALL Java_org_gaminganywhere_gaclient_GAClient_setObjectPath(
		JNIEnv *env, jobject thisObj, jstring objpath);
JNIEXPORT void JNICALL Java_org_gaminganywhere_gaclient_GAClient_setRTPOverTCP(
		JNIEnv *env, jobject thisObj, jboolean enabled);
JNIEXPORT void JNICALL Java_org_gaminganywhere_gaclient_GAClient_setCtrlEnable(
		JNIEnv *env, jobject thisObj, jboolean enabled);
JNIEXPORT void JNICALL Java_org_gaminganywhere_gaclient_GAClient_setCtrlProtocol(
		JNIEnv *env, jobject thisObj, jboolean tcp);
JNIEXPORT void JNICALL Java_org_gaminganywhere_gaclient_GAClient_setCtrlPort(
		JNIEnv *env, jobject thisObj, jint port);
JNIEXPORT void JNICALL
Java_org_gaminganywhere_gaclient_GAClient_setBuiltinAudioInternal(
		JNIEnv *env, jobject thisObj, jboolean enable);
JNIEXPORT void JNICALL
Java_org_gaminganywhere_gaclient_GAClient_setBuiltinVideoInternal(
		JNIEnv *env, jobject thisObj, jboolean enable);
JNIEXPORT void JNICALL
Java_org_gaminganywhere_gaclient_GAClient_setAudioCodec(
		JNIEnv *env, jobject thisObj,
		/*jstring codecname,*/ jint samplerate, jint channels);
JNIEXPORT void JNICALL
Java_org_gaminganywhere_gaclient_GAClient_setDropLateVideoFrame(JNIEnv *env, jobject thisObj, jint ms);
```

3. 发送键盘鼠标事件

```c
JNIEXPORT void JNICALL Java_org_gaminganywhere_gaclient_GAClient_sendKeyEvent(
		JNIEnv *env, jobject thisObj, jboolean pressed, jint scancode, jint sym, jint mod, jint unicode);
JNIEXPORT void JNICALL Java_org_gaminganywhere_gaclient_GAClient_sendMouseKey(
		JNIEnv *env, jobject thisObj, jboolean pressed, jint button, jint x, jint y);
JNIEXPORT void JNICALL Java_org_gaminganywhere_gaclient_GAClient_sendMouseMotion(
		JNIEnv *env, jobject thisObj, jint x, jint y, jint xrel, jint yrel, jint state, jboolean relative);
JNIEXPORT void JNICALL Java_org_gaminganywhere_gaclient_GAClient_sendMouseWheel(
		JNIEnv *env, jobject thisObj, jint dx, jint dy);
```

4. 媒体解码与播放

```c
JNIEXPORT void JNICALL Java_org_gaminganywhere_gaclient_GAClient_sendKeyEvent(
		JNIEnv *env, jobject thisObj, jboolean pressed, jint scancode, jint sym, jint mod, jint unicode);
JNIEXPORT void JNICALL Java_org_gaminganywhere_gaclient_GAClient_sendMouseKey(
		JNIEnv *env, jobject thisObj, jboolean pressed, jint button, jint x, jint y);
JNIEXPORT void JNICALL Java_org_gaminganywhere_gaclient_GAClient_sendMouseMotion(
		JNIEnv *env, jobject thisObj, jint x, jint y, jint xrel, jint yrel, jint state, jboolean relative);
JNIEXPORT void JNICALL Java_org_gaminganywhere_gaclient_GAClient_sendMouseWheel(
		JNIEnv *env, jobject thisObj, jint dx, jint dy);
```

GameAnywhere 对视频使用软解码(猜测出于兼容性考虑)，相比较硬解码性能较差，可替换为硬解码。

5. 媒体流连接请求

```c
JNIEXPORT jboolean JNICALL Java_org_gaminganywhere_gaclient_GAClient_rtspConnect(
		JNIEnv *env, jobject thisObj);
JNIEXPORT void JNICALL Java_org_gaminganywhere_gaclient_GAClient_rtspDisconnect(
		JNIEnv *env, jobject thisObj);
```

#### 媒体流传输客户端(Live555 客户端): server\ga\android\jni\src\rtspclient.cpp

媒体流传输客户端是一个RTSP客户端，依赖 Live555 的库 (BasicUsageEnvironment, UsageEnvironment, groupsock, liveMedia)。

RTSP 客户端启动一个线程，通过 ```BasicTaskScheduler.SingleStep()``` 方法不断从服务端读取音视频数据，然后解码、播放。

读数据: 

```c
void *
rtsp_thread(void *param) {
	RTSPClient *client = NULL;
	BasicTaskScheduler0 *bs = BasicTaskScheduler::createNew();
	...
	while(rtspParam->quitLive555 == 0) {
		bs->SingleStep(1000000);
	}
	...
}
```

如果是视频数据，回调 ```play_video``` 函数：

```c
static void
play_video(int channel, unsigned char *buffer, int bufsize, struct timeval pts, bool marker) {
	struct decoder_buffer *pdb = &db[channel];
	int left;
	//
	if(bufsize <= 0 || buffer == NULL) {
		rtsperror("empty buffer?\n");
		return;
	}
#ifdef ANDROID
	if(rtspconf->builtin_video_decoder != 0) {
		//////// Work with built-in decoders
		if(video_codec_id == AV_CODEC_ID_H264) {
			if(android_decode_h264(rtspParam, buffer, bufsize, pts, marker) < 0)
				return;
		} else if(video_codec_id == AV_CODEC_ID_VP8) {
			if(android_decode_vp8(rtspParam, buffer, bufsize, pts, marker) < 0)
				return;
		}
		image_rendered = 1;
	} else {
	//////// Work with ffmpeg
#endif
  ...
}
```

如果是音频数据，回调 ```play_audio``` 函数:

```c
static void
play_audio(unsigned char *buffer, int bufsize, struct timeval pts) {
#ifdef ANDROID
	if(rtspconf->builtin_audio_decoder != 0) {
		android_decode_audio(rtspParam, buffer, bufsize, pts);
	} else {
	////////////////////////////////////////
#endif
...
}
```

#### 媒体流解码与播放: server\ga\android\jni\src\android-decoders.cpp

具体的媒体流解码逻辑在 ```android-decoders.cpp``` 中实现。 包括配置解码器参数，启动解码器，解码一帧视频。

```c
int android_prepare_audio(RTSPThreadParam *rtspParam, const char *mime, bool builtinDecoder);
int android_decode_audio(RTSPThreadParam *rtspParam, unsigned char *buffer, int bufsize, struct timeval pts);
int android_config_h264_sprop(RTSPThreadParam *rtspParam, const char *sprop);
int android_decode_h264(RTSPThreadParam *rtspParam, unsigned char *buffer, int bufsize, struct timeval pts, bool marker);
int android_decode_vp8(RTSPThreadParam *rtspParam, unsigned char *buffer, int bufsize, struct timeval pts, bool marker);
```

#### 指令传输: server\ga\android\jni\src\controller.cpp

Java App层发送的指令，被保存到队列中，由 ```controller.cpp``` 实现的线程将队列中指令发送给服务端。指令通道是一条独立通道。

#### Java App

Java App 相对简单，提供界面与采集用户输入。

## Reference

[GameAnywhere On Github](https://github.com/chunying/gaminganywhere)

[GameAnyhwere Offical Site](http://www.gaminganywhere.org/)