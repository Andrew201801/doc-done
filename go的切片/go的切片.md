# Go的切片（进阶版）

带着设计的思维去学习！

[TOC]

## 引入：为什么要引入切片？

因为数组真的不好用啊，啊sir！你看看它，声明的时候就要说明容量，容量到了还不能自动扩容，那剩下的不就只有一个按下标遍历的功能了吗？

所以我们想要的是什么呢？是不需要声明容量，并且自动扩容，支持增删等操作的一个数据结构，其他语言里就有这样的东西，比如`Python`里的`List`，`C++`里的`Vector`，`JAVA`里的`ArrayList`。

如果我们自己实现，需要怎么做呢？带你回忆一下数据结构课教的东西

动态数组这个数据结构包含什么东西呢？首先，动态数组也是数组，需要开辟一块连续内存来存放东西。而与数组不同的是，动态数组在于容量到了之后的操作，即自动扩容——当动态数组到达容量后，他的容量会自动增加。那么，我们需要一个`cap`变量来存储当前容量，一个`len`变量来存储当前用了多少空间。相当于是做一道上汤娃娃菜，拿出一根火腿，整根火腿的长度就是它的容量`cap`，而一道菜一般用不了那么多，需要切下一部分来做菜，切下的那部分的长度就是`len`。当你做很多道菜的时候，一根火腿不够用了，你需要再买几根，这就是自动扩容。

因此，切片需要包括三个变量：

* 开头指针，指向了切片开头的地址

* `len`，标明了使用到的空间长度

* `cap`，表示总空间长度

切片操作主要包括

* 增，见下文切片新增元素
* 删，`go`没有实现这个操作，需要通过重复切片来实现，见下文
* 查改，涉及切片的遍历，切片的遍历与数组的遍历一样，都可以使用`for`或`for range`遍历

## 详细介绍

### 1.创建

####  1.1凭空创建

凭空创建主要包含两种方法，直接声明和`make`

直接声明：

`var a []T`：T为变量类型，可以在声明时直接赋值

```go
var a []int						//只声明，未分配空间，不能通过下标方式使用
a[0] = 1						//将报错
fmt.Println(nil == a)           //将输出true,说明只声明时，a相当于nil

var b = []int{}                  //声明并分配空间，len=cap=0
b[0] = 1                        //将报错
fmt.Println(nil == a)           //将输出false,分配空间后，b不等于nil

var c = []string{"Andrew","Myth"} //声明并分配空间，len=cap=2
c[0] = "Hi"                       //不会报错
```

make方式：

`var a = make([]T,len,cap)`：T为变量类型，cap可省略

```go
var a = make([]int,3,5)	//此时，len=3,cap=5，且a的前三个元素自动初始化为0
a = append(a,1)			//在a中添加元素，a=[0 0 0 1],len=4,cap=5
var b = make([]int, 3)	//此时，len=cap=3,全初始化为0
```

#### 1.2从已有的数组创建

`a[low : high : max]`：`low`是初始地址，`high`是`len`的最高位地址，`max`是`cap`的最高位地址，其中，`high`和`max`所对应的位置均不包含。

```go
a := [6]int{12,43,23,56,75,66}	//a是个长度为6的数组，a[0]=12	

//从a的下标1切到3,且不包含3，最大容量到下标5,此时，b=[43 23],len=2,cap=4,此时，与凭空创建不同的是，此时b没有用到的两个位置(cap-len=2)是有元素的，他们是56和75
b := a[1:3:5]

b[3] = 111		//将会报错，因为b[3]虽然有东西，但len=2,只能访问b[0]和b[1]
b = b[0:4:4]	//在b上重复切片，屁股往后挪，b=[43 23 56 75],len=4,cap=4
b[3] = 111		//不会报错了
```

其中`low`默认为`0` ，`high`默认为原数组长度，`max`也默认为原数组长度，以下是使用默认值的省略写法

```go
​```a := [6]int{12,43,23,56,75,66}	//a是个长度为6的数组,a[0]=12	
b := a[1:3]						//b=[43 23],len=2,cap=5与上例不同
c := a[:3]						//c=[12,43,23],len=3,cap=6
d := a[3:]						//d=[56,75,66],len=3,cap=3
e := a[1::5]					//将会报错
f := a[:]						//f=[12 43 23 56 75 66],len=cap=6
```

当重复切片时，`high`默认为原切片`len`，但可以比原切片`len`大，只是必须`<=`原切片`cap`小，`max`默认为原切片`cap`，以下是使用默认值的省略写法

```go
a := [6]int{12,43,23,56,75,66}
b := a[1:3:5]		//原切片b=[43 23],len=2,cap=4
c := b[:]			//c=[43 23],len=2,cap=4
d := b[1:]			//头往后挪，整体压缩d=[23],len=1,cap=3
e := b[0:4]			//头不动,屁股往后多切e=[43 23 56 75],len=4,cap=4
f := b[0:5]			//将会报错，屁股挪太多了
```

### 2.切片的增加操作：

#### 2.1`append`操作

切片的增加，使用的是`append`函数，如下

```go
a := []int{1}			//a=[1]
b := append(a,2)		//b=[1,2]
c := append(a,2,3,4)	//c=[1,2,3]
d := append(a,c...)		//append的另一种用法，将C解开后，添加到a中

//与上面的写法等价
d := a
for i:=0;i <len(c);i++{
    d=append(d,c[i])
}
```

聪明你的一定发现了，我在介绍`append`的时候忽略了对于`len`和`cap`的关注，因为如果新增后`cap`装不下了，就必须自动扩容，而这比较复杂，值得专门讲一下

#### 2.2切片自动扩容

再次把自己代入到`Go`设计者的角色里，如果是我们来设计这个功能，要怎么设计呢？数据结构课还是讲过，当容量到达原来的上限时，直接把原来的容量翻倍，重新申请一块内存，并把数据复制过去。(注意，**此时内存位置发生了变化**！！)

```go
a := []int{}
for i:=0;i<4;i++{
   a=append(a,i)
   fmt.Printf("len=%d cap=%d 内存位置:%p a=%v\n",len(a),cap(a),a,a)
}

//输出为
//len=1 cap=1 内存位置:0xc00000a098 a=[0]
//len=2 cap=2 内存位置:0xc00000a0e0 a=[0 1]
//len=3 cap=4 内存位置:0xc00000e1e0 a=[0 1 2]
//len=4 cap=4 内存位置:0xc00000e1e0 a=[0 1 2 3]
```

但这样会有一些问题：

* 如果一次添加的很多，翻倍也不够怎么办？

  先翻倍，然后一直+2，直到符合要求

  ```go
  func main() {
  	a := []int{30, 31}		//cap=2，翻倍后=4
  	fmt.Println("a的容量为",cap(a))
      
  	b := append(a,1,2,3)		//添加3个元素，至少需要2+3=5个cap
  	fmt.Println("添加后b的容量为",cap(b))
      	
  	c := append(a,1,2,3,4)	//添加4个元素，至少需要2+2=6个cap
  	fmt.Println("添加后c的容量为",cap(c))
      
  	d := append(a,1,2,3,4,5)	//添加5个元素，至少需要2+5=7个cap
  	fmt.Println("添加后d的容量为",cap(d))
  }
  
  //输出为
  //a的容量为 2
  //添加后b的容量为 6
  //添加后c的容量为 6
  //添加后d的容量为 8
  ```

* `cap`已经很大了,比如1024，再翻倍要很久才能填满，浪费空间

  那就不翻倍，一次添加原来的256（即1024的1/4），这是大多数教程的说法。我也翻源码看了，这种说法在一定范围内是正确的，比如下例：

  ```go
  a := make([]int,1024)
  b := make([]int,1)
  a = append(a,b...)
  fmt.Println(cap(a))		//将输出1280，即1024+256=1280
  ```

  但也有一些例外：

  ```Go
  a := make([]int,1024)
  b := make([]int,257)
  a = append(a,b...)
  fmt.Println(cap(a))		//输出1696,1696=1024+256+416,这个416哪里来的？
  ```

  对此，我分别遍历了a和b的大小，记录合并后cap跳变的瞬间，但我并没有从中找到规律，希望大家帮忙看看：

  ```shell
  //a固定为1024,b从0遍历到1024,cap跳变的瞬间：
  a的初始容量为1024 b的初始容量为0 添加后a的容量为 1024
  a的初始容量为1024 b的初始容量为1 添加后a的容量为 1280
  
  a的初始容量为1024 b的初始容量为256 添加后a的容量为 1280
  a的初始容量为1024 b的初始容量为257 添加后a的容量为 1696
  
  a的初始容量为1024 b的初始容量为576 添加后a的容量为 1696
  a的初始容量为1024 b的初始容量为577 添加后a的容量为 2048
  
  a的初始容量为1024 b的初始容量为976 添加后a的容量为 2048
  a的初始容量为1024 b的初始容量为977 添加后a的容量为 2560
  
  //a从0遍历到1024,b也从0遍历到1024,只记录cap(a)+cap(b)>=1024且cap跳变的瞬间：
  //有7000多行，太大了，文件链接如下：
  //假装有链接
  ```

  那大家说一次加256的说法哪里来的呢？我查找了slice.go的源码（注释是我自己加的）：

  ```go
  //自动扩容的函数，可以看到，这里传入了一个cap！！就是这个cap搞的鬼
  func growslice(et *_type, old slice, cap int) slice {
      //前面还有代码，省略
      newcap := old.cap
      doublecap := newcap + newcap
      //之前计算过一个cap,传进来后和旧cap的两倍(doublecap)比较,如果,传进来的cap比较大,就用传进来的值作为newcap。
      //例如上面举过的例子，a=[30 31]，新增3个元素，此时旧cap的两倍(doublecap)=4，而传进来的cap应该等于6,6>4，因此newcap=6。
      //对于cap(a)=1024,cap(b)=257,合并到a后cap=1696,应该也是外面传进来的这个cap
      //但是我没有找到外面传进来前怎么计算的
      if cap > doublecap {
         newcap = cap						
      } else {
         if old.cap < 1024 {
            //旧空间没有太大，直接翻倍
            newcap = doublecap
         } else {
            //256的出处
            for 0 < newcap && newcap < cap {
               newcap += newcap / 4		
            }
            //防止溢出
            if newcap <= 0 {
               newcap = cap
            }
         }
      }
  ```
  留一个小问题，底下这个代码的输出是什么：

  ```go
  var a =[]int{}
  b := append(a,1)
  c := append(a,1,2)
  d := append(a,1,2,3)
  fmt.Print(cap(b),cap(c),cap(d))
  ```



**PS**:学完了`append`，结合重复切片，就可以得到删除切片元素的方法

```go
var a = []string{"A","B","C","D","E"}
a = append(a[:2],a[3:]...)
fmt.Print(a)			//输出[A B D E]
//要删除下标为x的元素，则
x := 2
a=append(a[:x],a[x+1:]...)
```

**PPS**:关于三个点的语法问题

```go
//三个点可以用在：
var a = [...]int{1,2}		//自动推导数组长度
append(a,b...)			//解开切片
func append(slice []Type, elems ...Type) []Type	//接受不定长参数
```

关于解开切片，我很好奇切片解开后返回的是什么数据结构，但我尝试过都失败了，有人能说说嘛**(✪ω✪)**

```go
var a =[]string{"A","B","C","D","E"}
b := a...					//报错
fmt.Printf("%v",a...)	//报错
```

### 3.切片的传参及复制操作

#### 3.1传参

之前我们学到过，数组作为参数传递给函数，传递参数的形式是值传递，也就是说函数内部会自己开一块空间，复制这个传进来的数组，之后的操作都在自己的空间进行，不会修改原来的数组:

```go
func change(a [3]int){
   a[0] = 1
}
func main() {
   var a = [...]int{9,99,999}
   change(a)
   fmt.Print(a)		//a还是[9 99 999]
}
```

但我们使用切片，就可以改变原始的内容，因为切片里存的是指针，传过去的时候，函数内部也会自己开一块空间，复制这个传进来的切片，但复制前后，切片内部存的指针指向的是同一块地址，因此函数内部使用自己复制出来的切片进行操作时，也会影响到原始的切片

```go
func change(a []int){
   a[0] = 1
}
func main() {
   var a = []int{9,99,999}
   change(a)
   fmt.Print(a)		//a变成了[1 99 999]
}
```

#### 3.2复制

同样，简单赋值时，复制的也是指针：

```go
var a = []int{9,99,999}
b := a
b[0] = 1
fmt.Print(a)	//a变成了[1 99 999]
```

但有时，我们不想让复制后的指针修改原来的值，此时我们需要使用到copy函数，将原切片底层数组的值复制到新的地方，之后的操作就在新的地方进行了：

```go
var a = []int{9,99,999}
b := make([]int,3)
copy(b,a)
b[0] = 1
fmt.Print(a)	//a还是[9 99 999]
```

## 参考文献：

[1.Go语言基础之切片]( https://www.liwenzhou.com/posts/Go/06_slice/#autoid-2-5-0)

[2.golang 切片扩容的探讨](https://studygolang.com/articles/16052)

[3.Golang中的三个点...有什么用?](https://studygolang.com/articles/26231)