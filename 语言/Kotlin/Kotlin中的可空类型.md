Java的NullPointException是经常遇到的异常，也是最让人头疼的一个异常。Kotlin为了解决这个问题，引进了可空类型，将运行时可能发生异常提前到编译期发现。  
Kotlin中有可空类型，这种类型表示取值可能为空；而一般类型，则取值不能为空。区别是类型后面有一个？，表示这个类型是可空的。  
举个栗子：  
```
var s?=null
var s="Hello World"
var s1=null //编译器提示错误，因为s1是不可空的类型
```
为了应对可空的判断，Kotlin提供了几种操作符。  
## ?.  
在Java里，一段代码可能如下：  
```Java
int length(String s){
    if(s==null){
        return 0;
    }else{
        return s.length();
    }
}
```
而在Kotlin中，这段代码就是下面这个样子的：  
```Kotlin
fun length(s: String?):Int?{
    return s?.length
}
```
当对一个可空的类型使用?.等价于  
```
if(s==null){
    return null
}else{
    return s.length
}
```
这样得到的结果就是Int?，结果也是一个可能为空的类型。
**?.的返回类型需要注意，是一个可空类型**
## ?:  
Java中的三目运算符?:的使用如下：  
```Java
int length(String s){
    return s==null?-1:s.length();
}
```
Kotlin中也有?：运算符，使用情况类型，  
```Kotlin
fun length(s:String?):Int{
    return s?.length ?: -1
}
```
?:操作符等价于下面这段代码：  
```
if(s?length==null) {
    return -1;
}else{
    return s?.length
}
```
可以发现?:其实就是Java中的三目运算符。  
## !!  
如果在某种情况下，明确能知道一个可空类型不可能为空，那么可以使用!!进行说明，比如：  
```Kotlin
fun length(s:String):Int{
    return s!!.length
}
```
因为你自己确保了这个可空类型不为空，那么如果为空，那不好意思了，你就会碰到空指针异常了，Exception in thread "main" kotlin.KotlinNullPointerException  。  
所以说，Kotlin中虽然有了可空类型，但也不是就没有空指针异常哦。  

## 总结  
?.   ?:   !!   三种操作符要熟练使用，使用起来比Java写很多判断null的代码清爽多了，赶紧用起来吧，小伙伴们!
