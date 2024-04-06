---
tags: [Go, 编程语言]
date: 2014-08-23
title: GO语言学习总结2

---
### GO语言学习总结（variable, data type）
### 1. variable
#### 1.1 变量使用关键字 var 定义。变量是强类型的。

{% highlight go %}
package main

import "fmt"

var i int
var c, python, java bool

func main() {
    fmt.Println(i, c, python, java)
}
{% endhighlight %}

#### 1.2 定义变量时候可以不指定类型，而是通过赋值获得类型

{% highlight go %}
package main

import "fmt"

var i, j int = 1, 2
var c, python, java = true, false, "no!"

func main() {
    fmt.Println(i, j, c, python, java)
}
{% endhighlight %}

#### 1.3 函数内临时变量定义，通过 := 定义

{% highlight go %}
package main

import "fmt"

func main() {
    var i, j int = 1, 2
    k := 3
    c, python, java := true, false, "no!"      
    fmt.Println(i, j, k, c, python, java)
}
{% endhighlight %}

### 2. data type
#### 2.1 所有的数据类型

{% highlight go %}
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // alias for uint8

rune // alias for int32
     // represents a Unicode code point

float32 float64

complex64 complex128
{% endhighlight %}

#### 2.2 所有的数据类型必须显示转换，否则会编译错误

{% highlight go %}
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)
{% endhighlight %}

#### 2.3 常量

- 常量可以是数字，字符串，字符，布尔类型
- 常量的申明与变量相同，只是在前面加上**const**关键字
- 常量不能使用 := 赋值

{% highlight go %}
package main

import "fmt"

const Pi = 3.14

func main() {
    const World = "世界"
    fmt.Println("Hello", World)
    fmt.Println("Happy", Pi, "Day")
    const Truth = true
    fmt.Println("Go rules?", Truth)
}
{% endhighlight %}

### 3. struct
struct与C语言几乎一样，使用**tpye**和**struct**关键字定义。  

{% highlight go %}
package main

import "fmt"

type Vertex struct {
    X int
    Y int
}

func main() {
    fmt.Println(Vertex{1, 2})
}
{% endhighlight %}

#### 3.1 使用“点”访问结构体内的变量

{% highlight go %}
package main

import "fmt"

type Vertex struct {
    X int
    Y int
}

func main() {
    v := Vertex{1, 2}
    v.X = 4
    fmt.Println(v.X)
}
{% endhighlight %}

> Java中没有struct还真是不完美。定义一个结构体要多写好多代码。

#### 3.2 构造时候指定变量名称赋值

{% highlight go %}
package main

import "fmt"

type Vertex struct {
    X, Y int
}

var (
    p = Vertex{1, 2}  // has type Vertex
    q = &Vertex{1, 2} // has type *Vertex
    r = Vertex{X: 1}  // Y:0 is implicit
    s = Vertex{}      // X:0 and Y:0
)

func main() {
    fmt.Println(p, q, r, s)
}
{% endhighlight %}

> Go 支持指针