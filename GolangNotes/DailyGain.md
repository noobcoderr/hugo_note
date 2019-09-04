##1、当方法的接收者是指针时，方法可以改变结构体的值

```go
type Person struct {
   Name string
   Age int
}

func (p Person) SetName(name string) {
   p.Name = name
}

func main() {
   p := &Person{}
   p.SetName("zhangsan")
   fmt.Println(p.Name)
}
```

其结果是空，因为接受者不是指针，改为指针即func (p *Person) SetName，即可打印出"zhangsan"

## 2、interface接口存储的是具体实现者的值

接口interface 是 一种类型，是一种 具有一组方法 的 类型，跟其他类型如int，string等医院，该类型的变量具有的特性就是，必须实现该接口的所有方法。

```go
type Person struct {
   Name string
   Age int
}

type Human interface {
   Eat()
   Drink()
   Play()
}

func (p *Person) Eat() {
}
func (p *Person) Drink() {
}
func (p *Person) Play() {
}

func Praise(h Human) {
}
func main() {
   p := Person{Name:"Tom",Age:18}
   Praise(&p)
}
```

Praise()函数接收Human接口类型参数，而我们Person结构体实现了该接口的所有方法，即实现了Human接口，可以当做Human类型的变量的具体值(实参)。

interface的重要用途就体现在函数的参数中，如果多种类型实现了某个interface，那么这些类型的值都可以直接使用该interface的变量存储。如上例子，当又有一个结构体Tom实现了Human接口，那么当实例化出一个Tom后，也能当做实参传入Praise()。

> [理解 Go interface 的 5 个关键点](https://sanyuesha.com/2017/07/22/how-to-understand-go-interface/)

##3、Golang传参都是值传递/传值/传副本

这里面有2个概念容易弄混淆，那就是引用类型和传引用。

首先明确一个概念就是，golang内函数传参都是值传递。不论传的是普通变量还是指针,我只对你要传的值进行一次复制，传进去的是原值的副本。

与值传递对应的是引用传递，简要说明就是传递进函数时，传的不是副本，而是本身。

代码更直观,首先是基本类型int

```go
func main() {
   a := 1
   fmt.Println("a的原地址是:",&a)
   modify(a)
   fmt.Println(a)
}

func modify(a int) {
   fmt.Println("实际传入的a的地址是:",&a)
   a = 3
}
```

```
a的原地址是: 0xc000094000

实际传入的a的地址是: 0xc00009c000

1
```

这个很好理解，传进去的值的地址都变了，那内部改变对外部变了也产生不了什么影响了。那来看看传递指针的时候是什么样的

```go
func main() {
	a := 1
	p := &a //p存储的是a的地址,&p则存放的是指针p的存放地址
	fmt.Println("存储p的地址是:",&p)
	fmt.Println("p指向的地址是:",p)
	modify(p)
	fmt.Println(a)
}

func modify(a *int) {
	fmt.Println("实际传入的存储p的地址是:",&a)
	fmt.Println("实际传入的p所指向的地址是",a)
	*a = 3
}
```

```go
存储p的地址是: 0xc000088010
p指向的地址是: 0xc00008e000
实际传入的存储p的地址是: 0xc000088020
实际传入的p所指向的地址是 0xc00008e000
3
```

看到了吧，就算您是指针，我函数也只要一份你的备份，但是关键的一点是，这个备份所指向的地址跟原指针所指向的值是一样的，那么我在函数内部做修改影响到的还是这个地址内的值即内外一致。

现在可以理解值传递的意思了。值传递就是我不管你要传递的参数是啥类型的，我只需要一份你的副本，你要是个普通的int类型，我就复制一份你的数值传进去，你要传递一个指针，那我也只是复制一份指针变量的值传进去，函数内部对这个数值进行修改能不能影响原值这个我不管，那得看你传递的是什么类型的值了。

那这个类型又有哪些呢？可以分为非引用类型和引用类型，非引用类型包括int、string、struct，这一类型的值在传递的时候，函数接收一份副本，在内部进行修改其实是修改的副本的值，对原值产生不了影响，如果想要对原值产生影响，那么可以传递这些类型的指针进去，函数接收一份指针的副本，在内部进行修改的时候，还是会导向该指针指向的值，即可对原值产生影响。再说引用类型，包括map、chan、slice，这些类型的值在传递的时候，不需要传递该类型的指针，函数接收一份该类型的副本，在函数内部进行修改即可以影响原值，这是为什么呢？为啥int、string、struct传递的时候得传其指针才可以在内部修改，而这三种却不用呢？

因为他们在通过make()创建时创建出的本来就是个指针类型，比如map和chan

```go
// makemap implements a Go map creation make(map[k]v, hint)
// If the compiler has determined that the map or the first bucket
// can be created on the stack, h and/or bucket may be non-nil.
// If h != nil, the map can be created directly in h.
// If bucket != nil, bucket can be used as the first bucket.
func makemap(t *maptype, hint int64, h *hmap, bucket unsafe.Pointer) *hmap {
    //省略无关代码
}
```

```go
func makechan(t *chantype, size int64) *hchan {
    //省略无关代码
}
```

不只是`make()`，直接`mp := map[int]int`时创建的可以推测也是指针类型`*hmap`，有可能这种方式最后生成时也是调用的`make()`方法来创建。所以当我们传递这些引用类型的时候，其实传递的是指针类型，那么自然可以在内部改变外部的值了。

说说`slice`,它其实是一种元素和结构体指针的混合类型，当我们对其进行`slice[index]`索引取值/赋值时，其实改的是该索引对应的地址内的值。

```go
type SliceHeader struct {
	Data uintptr
	Len  int
	Cap  int
}

type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

即本质上，这个引用类型，他本身就是指针类型，或者内部字段为指针类型，那么也就可以理解为什么我们只传递该类型的副本就可以在函数内部修改外部的值了。

所以如果想要在函数内部修改全局变量的值的方法可以为：类型本身作为指针、类型里面有指针类型的字段(slice)

参考文章

[Go语言参数传递是传值还是传引用？](https://www.flysnow.org/2018/02/24/golang-function-parameters-passed-by-value.html)

##4、for的三重陷阱







参考文章

[Go for 三重陷阱](https://blog.csdn.net/u011304970/article/details/72674786)





