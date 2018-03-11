## 对象分配


> 所有的JS对象都是通过堆来进行分配的。  

> 使用process.memoryUsage()查看使用情况
  [Node.js v6.11.3文档](https://nodejs.org/dist/latest-v6.x/docs/api/process.html#process_process_memoryusage)  
  heapTotal and heapUsed refer to V8's memory usage. external refers to the memory usage of C++ objects bound to JavaScript objects managed by V8.



```
> process.memoryUsage()
{ rss: 27541504, 
  heapTotal: 9437184, 
  heapUsed: 5897048,
  external: 8935 }
  // 单位 字节
  // rss (resident set size) 常驻进程内存 所有内存占用
  // heapTotal 已申请堆内存
  // heapUsed 已使用堆内存
  // external c++对象绑定到js的内存
```

### 内存限制

> 内存限制主要原因是v8的垃圾回收制度。1.5GB内存做一次小的回收需要50MS，做一次非增量性回收需要1S以上，并且这会使JS线程暂停。因此限制内存。

![img](http://upload-images.jianshu.io/upload_images/1214547-f76a4eba8d3b0487.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### V8的堆组成
V8的堆由一系列区域组成：
> 新生代区：大多数对象的创建被分配在这里，这个区域很小，但垃圾回收非常频繁，独立于其它区。

> 老生代指针区：包含大部分可能含有指向其它对象指针的对象。大多数从新生代晋升（存活一段时间）的对象会被移动到这里。

> 老生代数据区：包含原始数据对象（没有指针指向其它对象）。Strings、boxed numbers以及双精度unboxed数组从新生代中晋升后被移到这里。

> 大对象区：这里存放大小超过其它区的大对象。每个对象都有自己mmap内存。大对象不会被回收。

> 代码区：代码对象（即包含被JIT处理后的指令对象）存放在此。唯一的有执行权限的区域（代码过大也可能存放在大对象区，此时它们也可被执行，但不代表大对象区都有执行权限）。

> Cell区、属性Cell区以及map区：包含cell、属性cell以及map。每个区都存放他们指向的相同大小以及相同结构的对象。

> v8在64位系统下只能使用1.4GB内存，在32位系统下只能使用0.7GB内存。  


##### 如何解除内存限制？
> 利用堆外内存: 使用Buffer类。Buffer 性能相关部分由C++实现。[Buffer教程](http://www.runoob.com/nodejs/nodejs-buffer.html)

> 在启动node时，传递--max-old-space-size=4096 (调整老生代内存限制，单位为mb。--max-new-space-size 已经不可用了)

> 使用stream处理大文件 [stream教程](http://www.runoob.com/nodejs/nodejs-stream.html)

> 官方建议：it is recommended that you split your single process into several workers if you are hitting memory limits. （拆分进程）

## 垃圾回收机制

![img](http://alinode-assets.oss-cn-hangzhou.aliyuncs.com/d9928033-09a4-48f3-b5f1-efec986d77c1.png)

##### V8的垃圾回收有如下几个特点
```
当处理一个垃圾回收周期时，暂停所有程序的执行。(stop-the-world 全停顿)  

在大多数垃圾回收周期，每次仅处理部分堆中的对象，使暂停程序所带来的影响降至最低。(增量标记等算法) 

准确知道在内存中所有的对象及指针，避免错误地把对象当成指针所带来的内存泄露。(标记指针法:在每个指针的末位预留一位来标记这个字代表的是指针或数据。)

在V8中，对象堆被分为两个部分：新创建的对象所在的新生代，以及在一个垃圾回收周期后存活的对象被提升到的老生代。
如果一个对象在一个垃圾回收周期中被移动，那么V8将会更新所有指向此对象的指针。
```

> 没有一种算法能够胜任所有场景，因此现代垃圾回收算法中按对象的存活时间将内存的垃圾回收进行不同的分代，然后分别对不同分代实行不同算法

> v8将内存分为新生代和老生代

##### 主要算法

1. Scavenge算法   (打扫)

    > 特点：牺牲空间换时间  
    
    > 将堆内存一分为二，一个处于使用中，一个处于闲置。  
    检查使用中空间的存活对象，将存活对象复制到闲置空间中。  
    完成复制后闲置空间和使用中空间角色互换。   
    当一个对象经过多次复制依然存活时，它会被认为是生命周期较长的对象，会被晋升到老生代中。   
    晋升的条件有两个，一个是对象是否经历过Scavenge回收，一个是空闲空间的内存占用比超过25%。（为何不是50%呢？）
    
    > 缺点：只能使用一半的堆空间，适合应用在新生代中。

2. Mark-Sweep & Mark-Compact (标记清除 & 标记整理)

    > Mark-Sweep遍历堆中的所有对象，标记活着的对象。  
    在清除阶段清除所有没有被标记的对象。  
    缺点在于内存空间会出现不连续的状态。

    > Mark-Compact与Mark-Sweep的差别在于，在整理过程中，将活着的对象往一端移动，然后清理掉边界外的内存。
    
速度： Scavenge>Mark-Sweep>Mark-Compact

3. Incremental Marking (增量标记)

    > 前三种算法都需要将应用逻辑暂停，待垃圾回收完成后再恢复，即“全停顿”。

    > 新生代较小，全停顿影响不大。但老生代很大，全停顿影响很大。

    > 为降低影响，将标记阶段改为增量标记，也就是将标记拆分为很多小标记，每做完一步就让js逻辑执行一会儿，垃圾回收与JS逻辑交替执行。
    
4. lazy sweeping & incremental compaction 等



## 内存泄漏

    常见原因
    1.缓存
    2.队列消费不及时    
    3.作用域未释放

#### case1 : 缓存
```js
var cache = {};
function set (key, value){
    cache[key] = value;
}
```
#### case2 : 无限增长数组
```js
var arr = [];
function x (value){
    arr.push(value);
}
```

#### case3: 无限重连导致的内存泄漏
```
const net = require('net');
let client = new net.Socket();

function connect() {
    client.connect(26665, '127.0.0.1', function callbackListener() {
    console.log('connected!');
});
}

//第一次连接
connect();

client.on('error', function (error) {
    // console.error(error.message);
});

client.on('close', function () {
    //console.error('closed!');
    //泄漏代码
    client.destroy();
    setTimeout(connect, 1);
});
```

泄漏产生的原因其实也很简单：event.js 核心模块实现的事件发布/订阅本质上是一个js对象结构（在v6版本中为了性能采用了new EventHandles()，并且把EventHandles的原型置为null来节省原型链查找的消耗），因此我们每一次调用 event.on 或者 event.once 相当于在这个对象结构中对应的 type 跟着的数组增加一个回调处理函数。

那么这个例子里面的泄漏属于非常隐蔽的一种：net 模块的重连每一次都会给 client 增加一个 connect事件 的侦听器，如果一直重连不上，侦听器会无限增加，从而导致泄漏。


#### case4: 小测试
```
泄漏了吗？
var run = function () {
  var str = new Array(1000000).join('*');
  var doSomethingWithStr = function () {
    if (str === 'something')
      console.log("str was something");
  };
  doSomethingWithStr();
};
setInterval(run, 1000);
```

```
泄漏了吗？
var run = function () {
  var str = new Array(1000000).join('*');
  var logIt = function () {
    console.log('interval');
  };
  setInterval(logIt, 100);
};
setInterval(run, 1000);
```

```
泄漏了吗？
var run = function () {
  var str = new Array(1000000).join('*');
  var doSomethingWithStr = function () {
    if (str === 'something')
      console.log("str was something");
  };
  doSomethingWithStr();
  var logIt = function () {
    console.log('interval');
  }
  setInterval(logIt, 100);
};
setInterval(run, 1000);
```  
一旦一个变量被任一个闭包使用了，它会在所有的闭包词法环境结束之后才被释放，这会导致内存泄漏。

## 参考：
    
>1. 《深入浅出Node.js》朴灵

>2. [《Asurprising JavaScript memory leak found at Meteor》](http://point.davidglasser.net/2013/06/27/surprising-javascript-memory-leak.html) [译文](http://doc.okbase.net/friskfly/archive/5361.html)

>3. [v8垃圾回收](https://github.com/yjhjstz/deep-into-node)
    
