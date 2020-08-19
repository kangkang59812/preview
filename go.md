1. channel [1](https://my.oschina.net/renhc/blog/2246871)
2. 垃圾回收 [1](https://www.jianshu.com/p/ebf03d9605d0)
#### 基础知识

如果是在函数外部定义，那么将在当前包的所有文件中都可以访问。名字的开头字母的大小写决定了名字在包外的可见性。如果一个名字是大写字母开头的（译注：必须是在函数外部定义的包级名字；包级函数名本身也是包级名字），那么它将是导出的，也就是说可以被外部的包访问

#### 变量声明 

```go
var sign int //未初始化，默认值0
sign=5

var sign =5 //根据类型自动判断

//只能用在函数体内
new_sign :=5 //：=左边必须至少有一个 new_sign之前必须没出现过

// 常用于全局变量,全局变量是允许声明但不使用的
var(
	sign1 int
  signt string
)

//常量，不允许被修改，数据类型只可以是布尔型、数字型（整数型、浮点型和复数）和字符串型
const identifier [type] = value
//iota自增
const (
            a = iota   //0
            b          //1
            c          //2
            d = "ha"   //独立值，iota += 1
            e          //"ha"   iota += 1
)
//  k=3<<2, l=3<<3
const (
    i=1<<iota
    j=3<<iota
    k
    l
)
```

#### 派生类型

- (a) 指针类型（Pointer） `var p *int`
- (b) 数组类型 `var a []int ` 作为参数传入函数，传的是指针，函数内改动**会**影响原数组
- (c) 结构化类型(struct)   作为参数传入函数，函数内改动**不会**影响原数组
- (d) Channel 类型 `var a chan int`
- (e) 函数类型 `var fun(string) int`
- (f) 切片类型
- (g) 接口类型（interface）`var a error`
- (h) Map 类型 `var m map[string] int`

#### 循环

```
// []中要么都加5，要么都不加或者 var s = 。。； for时要带索引i，ss是真实内容
var s []int = []int{1, 2, 3, 4, 5}
	for i, ss := range s {
		fmt.Println(i, ss)
	}
```

#### 切片

```go
slice:=[]int{1,2,3,4,5} //切片
//数组
array:=[5]int{4:1} //第五位赋值，其他位0
//切片
slice:=[]int{4:1}

slice := []int{1, 2, 3, 4, 5}
newSlice := slice[1:3]
/*
对于底层数组容量是k的切片slice[i:j]来说
长度：j-i   len(slice) 3
容量:k-i    cap(slice) 4
*/  
newSlice := slice[1:2:3] // len=2-1  cap=3-1

newSlice=append(newSlice,10,20)
//append函数会智能的增长底层数组的容量，目前的算法是：容量小于1000个时，总是成倍的增长，一旦容量超过1000个，增长因子设为1.25，也就是说每次会增加25%的容量
// slice和原数组共享数据，如果发生扩容，slice就会指向一个新建的数组
```



#### 协程

https://juejin.im/post/5c6a507fe51d45086925e22c


