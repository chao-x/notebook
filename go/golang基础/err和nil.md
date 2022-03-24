### Golang 进一步理解*nil*和*err*

### 示例1

我们先来看下面的代码：

```go
func (e *Error) Error() string {
	switch e.errCode {
	case 1:
		return "file not found"
	case 2:
		return "time out"
	case 3:
		return "permission denied"
	default:
		return "unknown error"
	}
}

func checkError(err error) {
	if err != nil {
		panic(err)			//这里会panic
	}
}


func main(){
	var err *Error
	checkError(err)
}
```

在我们一般的理解中，我们声明里err变量指针，并没有为他初始化申请内存，此时它为nil。在checkError函数中，if语句应该不成立，不会触发panic函数，但实际中，并不是如此。



### 官方实现

我们可以看源码，在golang官方的实现中，

```go
// nil is a predeclared identifier representing the zero value for a
// pointer, channel, func, interface, map, or slice type.
var nil Type // Type must be a pointer, channel, func, interface, map, or slice type

// Type is here for the purposes of documentation only. It is a stand-in
// for any Go type, but represents the same type for any given function
// invocation.
type Type int
```

这里说明了必须是nil是预先声明的标识符，他表示了指针，管道，函数，接口，哈希，接口类型的零值。

这里的Type仅用于文档目的，他可以表示任何go的类型，但是如果给定了功能调用，他就表示该类型。

并且nil必须是指针，管道，函数，接口，哈希，接口类型之一，他们都是引用类型，底层都为指针。

我们可以简单的理解nil为是空指针。



### 修改示例1

为了达到我们最初的目的，我们修改了程序

```go
func (e *Error) Error() string {
	switch e.errCode {
	case 1:
		return "file not found"
	case 2:
		return "time out"
	case 3:
		return "permission denied"
	default:
		return "unknown error"
	}
}

func checkError(err error) {
	if err != (*Error) (nil) {
		panic(err)			//此时不会触发panic
	}
}


func main(){
	var err *Error
	checkError(err)
}
```

不会触发的原因是因为当err与nil做比较时，err为接口类型(error接口)，如果err要与nil相等，则其Type和Value都得相等。

在之前的程序中，err的Type和Value为（Error，nil），而nil的Type和Value为（nil，nil），自然是不相等的。当我们为nil强制类型转换后，其就与err相等。



### 示例2

```go
type Plane struct {
        Num int
}

func (this *Plane) Fly1(){
        fmt.Println("Fly1......")
}

func main(){

        test(nil)		//不会panic
}

func test(pl *Plane) {
        pl.Fly1()
}
```

这里我们传入了nil，但是却可以**成功调用test函数**，因为**调用方法并不需要申请内存**。

> 这里有待进一步深入

但我们调用其成员变量时，明显就不可以了。

```go
func (this *Plane) Fly2(){        
  fmt.Println("Fly2......Num:", this.Num) 	//panic
} 

func test(pl *Plane) {       
  pl.Fly2() 
}
```

> [本示例参考](https://blog.csdn.net/lanyang123456/article/details/102616712)