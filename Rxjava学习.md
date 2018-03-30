# Rxjava  学习

RxJava是一个基于事件订阅的异步执行的一个类库

#### **基础**

just(T...): 将传入的参数**依次发送**出来。

from(T[]) / from(Iterable<? extends T>) : 将传入的数组或 Iterable 拆分成具体对象后，**依次发送**出来。

也就是依次将事件发送给监听者



#### **线程切换**

**Schedulers.io(): **I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。行为模式和 newThread() 差不多，区别在于 io() 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。不要把计算工作放在 io() 中，可以避免创建不必要的线程。
**Schedulers.computation():** 计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 Scheduler 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU。



subscribeOn 订阅事件发生的进程

observerOn  Subscriber 所在的线程

observeOn() 的多次调用，程序实现了线程的多次切换。

subscribeOn() 的位置放在哪里都可以，但它是只能调用一次的。?????  

doOnSubscribe？？？？



线程切换的原理

```java
Observable.create(onSubscribe)  
    .subscribeOn(Schedulers.io())  
    .doOnSubscribe(new Action0() {  
        @Override  
        public void call() {  
            progressBar.setVisibility(View.VISIBLE); // 需要在主线程执行  
        }  
    })  
    .subscribeOn(AndroidSchedulers.mainThread()) // 指定主线程  
    .observeOn(AndroidSchedulers.mainThread())  
    .subscribe(subscriber);   
```



#### 变换

map(): 事件对象的直接变换

flatMap():？？？  网络嵌套

lift():

compose():











throttleFirst(): 应用？





doOnnext  doOnError？？   在subscribe调用之前执行数据的处理

sample  定时发射





## Flowable  

后面必须要跟subscrible 

而Flowable的异步缓存池有个固定容量，其大小为128。





#### SINGLE



#### RXANDROID



### BUFFER

```
* 2.buffer(count,skip) :有这么一种情况 buffer(count) 等价于 buffer(count,count).
 * 详细说一下： buffer(2)输出为 【1,2】【3,4】【5,6】 那么buffer(2,3)就要这么算：首先从第一项开始缓存2个数据，【1,2】，此时的skip=3，意思是第二个集合的第一项是第skip+1项的数据，也就是第四个,集合长度是2，结果就是【4,5】，
 * 此时可用的数据是4,5,6，再计算第三个集合的时候，第三个集合的第一项就应该是第skip+1项的数据，然而没有第四个数据，就没有第三个集合了。
```



skip  跳过

take  最多接受





### 基础用法

Observable   ->subscribe

​	.subscribe(new Observer ->)

![](https://upload-images.jianshu.io/upload_images/3994917-21e4dcc1b5e3196a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

 但是Observeable和Flowable都支持Cousumer





Contact 

两个Obserable的泛型应该一致；

如果一个Obserable 获取成果则调用 onNext；如果希望调用另外一个Obserable，则可以使用onComplete





uncX包装的是有返回值的方法，用于Observable的变换、组合等等；ActionX用于包装无返回值的方法，用于subscribe方法的闭包参数。Func1有两个入参，前者是原始的参数类型，后者是返回值类型；而Action1只有一个入参，就是传入的被消费的数据类型。





在Obserable的subscribe方法中其实已经完成了数据的异步操作，也就是说数据其实已经在这时候处理完了



Action0 - >Action



Action1 - > Consumer

Action2 -> BiConsumer



Func - > Function

Func2 - >BiFunction







###应用场景：

#### 1.数据类型转换为另外一个数据类型

map

#### 2.数据类型转换为另外一个Obserable

#### 3.先后嵌套与请求

Flatmap

#### 4. 先后或操作

contact （如果前面的满足，后面的则不在执行）

####5.不区分顺序，数据合并

zip

#### 6.心跳

Flowable.interval(1, TimeUnit.SECONDS)  默认新开线程

#### 7.数据去重

distinct

#### 8.过滤掉不满足的

filter

#### 9.



merge和zip

1.方法的参数不一样，zip有一个合并函数，merge没有，所以zip发射数据是合并函数的返回值，merge则是交错排列多个源Observable发射的数据。
2.merge的终止不会受任何一个Observable的发射完成而终止，zip则只要有一个Observable的发射完成而终止发射
（merge和zip中只要有一个错误通知终止，就都终止）





























