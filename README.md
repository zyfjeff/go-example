## Introduce
这是来自于[go by example](https://gobyexample.com/)的例子，花了几天的时间写完了这些例子，感觉对我的帮助很大，对于
初学者来说，我的建议还是先找本go的书从头到尾看一下，然后再来看这些例子，每个例子都手敲一遍，对你的帮助还是很大的。
在敲这些例子的过程中，有一些疑问，也有一些知识的扩充，因此总结了本文。

## 你不知道的打印输出
在go中fmt包功能很强大，里面提供了Print，Println等打印方法，支持类似于C语言的格式化输出，最重要的是fmt包可以识别任意类型
功能很强大。下面是利用fmt来打印一个自定义的结构体。

```
package main
import "fmt"


type custom_struct struct{
    name string
    age int
}


func main() {
    man := custom_struct{"zhang",10}
    vars := []string{"first","second","last"}
    fmt.Println(man)
    fmt.Println(vars)
}
```
上面的代码可以直接打印custom_struct，还可以直接打印数组等，功能还是很强大的，但是却很少有人知道，go语言在内部也有一套
print，println方法用来答应你输出，用的人很少，这些打印方法一般用于go内部debug使用，不支持打印自定义类型。

## 大容量的常量
在go语言中，数字可以直接写成指数形式，例如`3e20`，而且go中的常量可以接收任意精度的常量表达的值，例如下面这个例子。

```
package main
import "fmt"


func main() {
    const n = 5000
    const d = 3e100 / n
    fmt.Println(d)
}
```
常量d可以接收任意精度的值，除此之外常量还有另外一个特点就是无类型，可以通过显式的转换赋予其类型，或者通过上下文赋予
常量类型，例如赋值操作，函数调用，那么常量的类型就变成了赋值操作符左侧的类型，或者变成了函数调用的形参类型了。

## 无所不能的for
在go中，for循环可以是C中的while循环，for循环，一个关键字完成所有的循环操作，还支持类型python中的range的循环操作。

```
func main() {
    i := 1
    nums := []int{2,3,4}
	//while循环
    for i <= 3 {
        fmt.Println(i)
        i = i + 1
    }
	
	//普通的for循环
    for j := 7; j <= 9; j++ {
        fmt.Println(j)
    }

    //基于range的循环
    for i,num := range nums {
        if num == 3 {
            fmt.Println("index:",i)    
        }    
    }

	//死循环
    for {
        fmt.Println("loop")
        break
    }
}
```

## if还可以这样子
if语句大体上来看，和C中的if在使用上差别不大，只不过go语言中的if支持在条件的前面加上一个赋值表达式，刷新了我的三观，并且
在go中没有像C中的那个问号表达式了，下面是带赋值表达式的if

```
func main() {
    //这里是可以前置一个赋值表达式的
    if num := 9; num < 0 {
        fmt.Println(num,"is negative")
    } else if num < 10 {
        fmt.Println(num,"has 1 digit")
    } else {
        fmt.Println(num,"has multiple digits")
    }
}
```

## 不一样的switch
自从学习了golang，不断刷新我三观，这和我之前接触的C/C++，Python差别不是一般的大啊，在go中switch也同时兼备了正常和不正常的地方
正常使用的情况和C中的switch基本一样，但是go中的switch没有穿透的概念，意思就是说，匹配到指定的项后就结束停止了，不再向下。go
中的switch是不需要显示的break的。除此之外在和C中的switch别无差别，但是这只是go switch正常的一面，不正常的一面则是go的switch
支持多条件匹配,支持表达式的匹配。

```

    switch time.Now().Weekday() {
    case time.Saturday,time.Sunday:     //用逗号分割的多个条件，匹配任意一个都可
        fmt.Println("it's the weekend")
    default:
        fmt.Println("it's a weekday")
    }

    //类似与if/else的感觉
    t := time.Now()
    switch {
    case t.Hour() < 12:     //为true就执行下面的语句,为false就直接跳到default分支
        fmt.Println("it's before noon")
    case true:              //继续执行,如果这里是false，那么就跳转指向default分支
        fmt.Println("continue")
    default:
        fmt.Println("it's after noon")
    }
```

## arrays和slice不是一回事
go 中的arrays和C中的数组是一样的概念，而slice和python的列表是一个概念，只不过go中的slice必须是同一类型的而已。arrays是固定长度
而slice则不是固定长度，可以通过append方法向其追加元素，但是在go中却没有单独给slice配备一个关键字，这导致有的时候不知道怎么区分
arrays和slice。

```
arrays := [5]int{1,2,3,4,5} //这是一个数组，指明长度了
slices := []int{1,2,3}      //这是一个slice
slices2 := make([]int,3)    //这样也可以是一个slice
```
那么在go中slice是如何实现的呢?在讨论这个问题的时候，让我们来先看看go中的数组的一些实现细节，因为slice就是对array的抽象。
go中的数组和C中的数组类似，但还是有一些不一样的地方，比如go的数组变量代表的就是整个数组，而不是像像C那样是一个指针，指向
数组的首元素，因此在go中赋值和传递一个数组变量会导致对数组中的全部元素进行拷贝。数组在定义的时候需要指定长度，也可以让
编译器帮我们计算，通过如下代码实现:

```
b := [...]string{"test","dwqd"}
```

slice不需要指定长度，这是和数组最明显的区别，那么这个slice内部到底是什么样子的呢，是和c++中的vector一样吗? slice其实就是对
array的一个包装和抽象，每个slice内部都会对应一个数组，这个slice维护一个指针指向这个数组的首元素，维护这个数组的长度len,
还会记录数组的最大长度capacity。那么这个len和capacity是什么关系呢? 和C++的vector中的capacity一个意思吗?，恰恰不是。限于
篇幅这里我不再过多介绍slice的内部实现细节了，给大家推荐一篇博客[go-slice-and-internals](https://blog.golang.org/go-slices-usage-and-internals)


## 多返回值的奇妙体验
限于我之前一直写C，常常需要返回多个值的时候，通常都是借助与函数参数来实现，好在go提供了多返回值的能力。这样就极大了简化了
代码，在返回结果的同时，也返回出现的error，但是需要记住的是，只能返回两个值哦，千万不要贪多。

## 受限的指针
之所以选择学习go，主要是感觉go是一个类C的语言把，熟悉的struct，熟悉的指针，但是go中的指针是受限的，不支持算术运算因为需要
支持垃圾回收。通过指针作为函数参数，在函数内部修改会影响到原始的变量，默认情况下是值拷贝的，在函数内部修改不会影响原始变量。

## 像class的struct
go中的struct也是用来自定义数据类型的，需要需要以面向对象的方法来编程，就需要将方法和数据封装在一起，C中通过在struct中包含
函数指针来实现，go则不然，原生支持，直接给struct定义方法。

```
package main

import "fmt"

type rect struct {
    width,height int
}

//给rect自定义的数据类型，定义了两个方法
func (r *rect) area() int {
    return r.width * r.height
}

func (r rect) perim() int {
    r.width = 100;
    return 2 * r.width + 2*r.height
}


func main() {
    r := rect{width: 10,height: 5}
    fmt.Println("area: ",r.area()) //r 是一个值类型，可是area的参数类型是指针啊，这居然可以调用 
    fmt.Println("perlim: ",r.perim())
    fmt.Println(r)

    rp := &r
    fmt.Println("area: ",rp.area())
    fmt.Println("perlim: ",rp.perim())

}
```
在上面的例子中给rect定义了两个方法，一个是指针作为第一个参数，另一个则是值作为参数，但是在使用的时候居然可以不加区别的使用。
这其实都是go在背后帮我们做了转换，但是不影响最终的效果，指针作为参数的，在函数内部操作会直接影响到原有的变量。


