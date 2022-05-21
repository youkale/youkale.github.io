---
title: 如何提高golang的可读性
date: 2019-03-30 18:55:00
tag: golang
---

## 1. 尽早返回

反例:

```go
//UserCtrl
func UserInfo(userId string){
  user.UserInfo(userId)
  ....
  ....
  //resp result ...
}

//UserService
func UserInfo(userId string){
  if len(userId) > 0 { 
  	//do query database 
    .....
  } 
}
// repo
func queryUserInfo(userId string){
  if len(userId) > 0{
    //select * from user where user_id = ?
  }
}
```

从这个例子来看，在service层和数据库查询，我们都进行了userId的判断. 因为当我们经常会忘记，我们是否在上一层入参的时候进行了userId为空的判断. 为了避免空指针，我们不得已一层层进行判断.

假如我们尽早地返回，那么就可以避免后续的层层判空

推荐写法:

```go
//userCtrl
func UserInfo(userId string){
  if len(userId) == 0 {
    //resp some error
  }
  user.UserInfo(userId)
}
```



## 2. 写好分支语句

反例:

```go
func xxx(lang string){
  if lang == "java"{
    doA()
  } else {
    doB()
  }
  
  if lang == "java"{
    doC()
  }else{
    doD()
  }
}
```

推荐写法:

```go
func java(lang string){
  doA()
  doC()
}

func other(lang string){
  doB()
  doC()
}
```



if都需要包含else

反例:

```go
if cmd == "1" {
  if status == 0 {
    doSome()
  }
}else {
  if tag == 1 {
    doSomeB()
  }
}
```



推荐写法:

```go
if cmd == "1" {
  if status == 0 {
    
  }else{
    
  }
}else {
  if tag == 1{
    
  }else {
    
  }
}
```

避免啰嗦的条件

```go
if isDone() == true {
  do()
}
```

推荐:

```go
if isDone() {
  doSome()
}
```

使用switch语句

反例:

```go
if lang == "java"{
  
}else if lang == "c#" {
  
}else if lang == "clojure"{
  
}
```

推荐写法:

```
switch lang:
case "java":
case "c#":
case "clojure":
```

减少逻辑表达式:

逻辑表达式是门电路的表达式，总会有人不能记得他的先后执行顺序。

反例:

```go
if lang == "java" || lang == "c#" && lang == "clojure" {
  doSome()
} 
```

如果非得使用逻辑表达式，推荐使用括号，显示地说明调用的顺序.

```go
if (lang == "java" || lang == "c#") && lang == "clojure" {
  doSome()
} 
```



使用正序的逻辑

反例:

```go
if !isUserInfoSaved() {
  doA()
}else {
  doB()
}

if !isNotStop() {
  doSome()
}
```



用一个符合人类思考顺序方式来写分支，减少阅读代码时的时间

```go
if isUserInfoSaved() {
  doB()
}else {
  doA()
}

if isStart(){
  doSome()
}
```



## 3.合理使用局部变量

可能我们知道定义一个变量会开辟一块新的内存，有时候觉得自己重复使用一个变量，会让性能"好一些", 于是我们就会写出下面的代码

反例:

```go
var name
if login {
  name = "user"
  doSome(name)
}else {
  name = "guest"
  doSome(name)
}
```

其实在栈上开辟内存的成本很低，编译器会对代码进行逃逸分析，而且执行完这个方法后，内存就会被回收掉，所以不用担心这个性能问题.

推荐写法:

```go

if login {
	//使用局部变量
  name := "user"
  doSome(name)
}else {
  name := "guest"
  doSome(name)
}
```

有的时候我们会想耍个酷，那么牛逼的调用一行代码就写完了，可是这时候阅读起来是非常痛苦的一件事情.

反例:

```go
saveXXX(queryRole(),queryOrder(),saveXXX(queryUserInfo(genUserId())))
```

推荐写法：

我们把参数通过一个中间变量存起来，这样会很明显地说明，我们都干了些什么.

```go
userInfo := queryUserInfo(genUserId)
u := saveXXX(userInfo)
saveXXX(queryRole(),queryOrder(),u)
```



## 4.循环

  循环本身就不好读，假如在循环中包含continue,break之类的，让原本的代码更难读 

反例:

```go
for i,itm := range Users {
  if item.Name != "admin" {
    continue
  }else {
    doSome()
  }
}
```

推荐写法:

```go
for i,itm := range Users {
  if item.Name == "admin" {
    doSome()
  }
}
```

使用i,j之类的下标，本身就比较相似，一不小心就会造成了下标越界,推荐使用foreach

反例:

```go
for i:=0 ; i< len(users); i++ {
   for j:=0 ; j < len(users[i].children); j++ {
     // doSome
   }
}
```

推荐写法

```go
for _,itm := range users {
  for _, c := range item{
    //dosome
  }
}
```



假如非得使用下标操作，也要避免使用i,j之类的变量

```go
for uidx :=0 ; uidx< len(users); uidx++ {
   childen := users[uidx].children  //使用中间变量,避免臃肿
   for chidx:=0 ; chidx < len(childen); chidx++ {
       // chidx 和 uidx 能避免混淆，能在使用的过程中避免出错。
   }
}
```



## 5.面条代码, 过程式代码

面条代码过程式代码

```go
type UserInf struct {
  UserName   string
  UserId     string
  Role       *Role
  Alias     string
}

type Role struct {
  Id string
  Name string
}

func save(inf *UserInf){
  Role(inf)
  inf.Alias = genAlias()
  update(inf)
}

func update(inf *UserInf){
  //update ....
}

func Role(inf *UserInfo) {
  queryUser(inf)
  inf.Role = queryRole(inf.UserId)
}

func queryUser(inf *UserInf){
  //select * from user where user_id = ?
  inf.UserName = ...
  inf.xxx = ...
}
```

思考: 为什么要使用纯函数?


## 6. 控制代码的长度

人的左脑关心的是逻辑，右脑做的是快照(可能是伪科学)，实际情况中，假如我们一眼能看完，是不是剩下的就是在想逻辑，而不是一边读代码，一边想逻辑，这样能让我们大脑一次性把看到的代码缓存起来，然后专注于想逻辑。简短的代码，也可以避免bug，所以方法建议都控制在100行之内。

## 7. 命名

这个放最后来写的原因是这个命名本来就很难，命名得好就会让代码清晰可读，命名不好，就会误导导致需要大量的注视来注释代码，本来维护代码已经是一件痛苦的事情了，假如在修改了代码后，注释没有同步修改，反而会引起误导。因为母语不是英文，很多同学跟我一样也都很痛苦，这里可以上网找找相关命名的资料，本人水平有限也只能是大概地举几个例子

```
bool isStart // 服务启动的状态，最好使用正向的表达，是否启动，不推荐使用  bool isNotStop
func ComputeUserScore() //计算用户积分，假如这是一个耗时的操作，推荐在方法名上就表示出来，不推荐使用GetUserScore,
func DownloadFile() //下载文件，不推荐使用GetFile
```
