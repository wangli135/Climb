Dart是一个单线程编程语言。如果任意代码阻塞了程序执行（比如，等待一个IO的耗时操作），程序将会冻结。异步操作允许你的程序不阻塞地执行。Dart使用Future对象表示异步操作。  

# 介绍  

首先看一个会阻塞程序执行的代码：  

```dart
void doLargeWork(){
  
  for(int i=0;i<10000;i++){
    print(i);
  }
  
}

void main(){
  
  doLargeWork();
  doLargeWork();
  doLargeWork();
  doLargeWork();
  
}
```

上面的代码中，假设doLargeWork()方法是一个耗时操作，那么上一个方法不执行完，那么下一个方法是没法执行的。  

# Future是什么  

Future代表了一个可以返回一个结果的异步操作，这个结果也可以是void。当一个返回Future的函数被调用时，两件事情可能发生：  

1. 返回一个未执行完的Future对象  

2. 当操作完成时，Future返回一个值或一个错误 

使用Future有两种方式：  
1. 使用async和await关键字

2. 使用Future API

## Async和Await  

一个async函数是用async关键字标注在body之前的函数。await关键字只在async函数内有效。  

### 处理错误  

异步函数可以内部处理错误：  

```
Future<void> printDailyNewsDigest() async {
  try {
    var news = await gatherNewsReports();
    print(news);
  } catch (e) {
    // Handle error...
  }
}
```

### ### 线性处理  

```dart
main() async {
  await expensiveA();
  await expensiveB();
  doSomethingWith(await expensiveC());
}
```

## Future API 

在async和await关键字加入到1.9之前，只能使用Future API。

使用Future API时，你可以使用then()方法注册一个回调，这个回调用于处理Future得到的结果。

### 处理错误  

catchError()  

```
Future<void> printDailyNewsDigest() =>
    gatherNewsReports()
        .then((news) => print(news))
        .catchError((e) => handleError(e));
```

### 使用then()链式调用  

```
expensiveA()
    .then((aValue) => expensiveB())
    .then((bValue) => expensiveC())
    .then((cValue) => doSomethingWith(cValue));
```

### 等待多个Future的执行结果  

```Dart
Future.wait([expensiveA(), expensiveB(), expensiveC()])
    .then((List responses) => chooseBestResponse(responses))
    .catchError((e) => handleError(e));
```

