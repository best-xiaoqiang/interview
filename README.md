## 跨域
#### JSONP
> JSONP 核心原理：script 标签不受同源策略约束，所以可以用来进行跨域请求，优点是兼容性好，但是只能用于 GET 请求

```
// 后端：
ctx-- request-query-cb(jsonpCb)
ctx-- body=`cb(${data})`

// 前端：
window.jsonpCb = (data){}
<script src="http://xxx?cb=jsonpCb" />
```
cb：前后端商定一个传递回调函数名称的参数  
data：返回数据  
jsonpCb：回调函数函数名称  

> 根据script不受同源策略的约束，通过后端传参调用函数的方式传递数据

[前端实现jsonp](https://juejin.cn/post/6946022649768181774#heading-18)
```
const jsonp = ({ url, params, callbackName }) => {
    const generateUrl = () => {
        let dataSrc = ''
        for (let key in params) {
            if (params.hasOwnProperty(key)) {
                dataSrc += `${key}=${params[key]}&`
            }
        }
        dataSrc += `callback=${callbackName}`
        return `${url}?${dataSrc}`
    }
    return new Promise((resolve, reject) => {
        const scriptEle = document.createElement('script')
        scriptEle.src = generateUrl()
        document.body.appendChild(scriptEle)
        window[callbackName] = data => {
            resolve(data)
            document.removeChild(scriptEle)
        }
    })
}
```

## BFC
[什么是BFC？看这一篇就够了](https://blog.csdn.net/sinat_36422236/article/details/88763187)  
BFC：四边高度为2  
普通div：左右边高度2（可折叠），上下边高度1；上下边宽度向右撑满  
float：上边高度3，下边高度2，离地1；上下边宽度为设置值  
overflow：上下边宽度向右撑满。（float和overflow可能在一行；同行下边界由overflow决定）  
relative：原元素继续占位。新的一层；宽度向右撑满  
absolute：新的一层；上下边宽度为设置值  
margin：高度为1处的向外方向的棍子  

## setInterval和setTimeout
[setInterval和setTimeout](https://www.jianshu.com/p/fc9a08ca2c92)
#### 时间
setInterval： 相邻两次添加队列的时间间隔  
用setTimeout模拟setInterval： 上一次任务执行完成和下一次任务添加到队列的时间间隔

#### setInterval问题
1. 当任务执行时间大于时间间隔，下一次任务会继续等待，并在上一次任务执行完成时立即执行。
2. 同一个定时器在任务队列中只会有一个回调函数，当任务执行时间大于N个时间间隔时，也只有一个任务在等待。

setInterval
整点挂号；  
下个整点上个病人仍在就诊时，依然可以挂号并在候诊室等候；  
候诊室最多允许一人等候，当候诊室已经有人时整点不开放挂号。

用setTimeout模拟setInterval  
上个病人就诊结束后，医生休息一定时长后开放挂号。

## 事件循环机制
[详解JavaScript中的Event Loop（事件循环）机制](https://zhuanlan.zhihu.com/p/33058983)  

单线程：医生  
执行栈：正在进行的诊断  
微任务：进行过相应检查并回来找大夫诊断的人  
宏任务：挂号等待的人  

[microtask queue存储的任务，必须要在当前函数执行栈为空时才会开始调度](https://segmentfault.com/a/1190000019123388)

## LRU
least recent used
![image](https://user-images.githubusercontent.com/27996959/120997182-92c20c80-c7b9-11eb-9491-54449de6dc5c.png)

## 闭包
董事长订的大蛋糕  
部门经理订的包间的烧烤摊  
组长订的桌子上的烤鱼  
每个员工碗里的饭  

作用域链：每个人依次可以吃碗里的饭——>桌上的烤鱼——>包间里的烧烤摊——>董事长订的大蛋糕。  
垃圾回收机制：服务员收拾餐具。  
闭包：虽然董事长走了，但是即使只剩一个员工还在吃，服务员都不会收走董事长订的大蛋糕。  

## new操作符
[new操作符干了什么](https://www.jianshu.com/p/35414c2063e8)
对于`cosnt a = new Foo()`，`new`干了以下事情：
```
const o = new Object() //创建了一个新的空对象o
o.proto = Foo.prototype //让这个o对象的 __proto__指向函数的原型prototype
Foo.call(o) //this指向o对象
a = o //将o对象赋给a对象
```
1.招生  
2.加助理联系方式  
3.培训  
4.毕业  

## 继承
[js继承的6种方式](https://www.cnblogs.com/ranyonsue/p/11201730.html)

```
function Person(name){
  this.name = name
  this.sum = function(){
    alert(this.name)
  }
}
```
寄生组合式继承
```
function content(obj){
  function F(){}
  F.prototype = obj
  return new F()
}
var con = content(Person.prototype)
function Sub(){
  Person.call(this)
}
Sub.prototype = con
con.constructor = Sub //给con补充constructor属性
var sub1 = new Sub()
console.log(sub1.age)
```

## 强制缓存和协商缓存
[深入理解浏览器的缓存机制](https://www.jianshu.com/p/54cc04190252)
![](https://raw.githubusercontent.com/best-xiaoqiang/image/main/qianduan1/cache.png)

## 属性描述符

属性描述符  
* configurable 可配置
* enumerable 可枚举
* value 值
* writable 可写  
或
* configurable 可配置
* enumerable 可枚举
* get
* set

机械手表  
  configurable 关闭就不能打开的一次性后盖  
  enumerable 隐身开关  
  value 显示的值  
  writable 正面玻璃盖  
电子手表  
  get  
  set  

configurable  
  1.在false情况下，如果修改，不管是不是严格模式，都会抛出TypeError的错误  
  2.在这种情况下，我们仍可以将可写性的状态由true改为false（最后盖上玻璃盖）  
  3.delete属性也会被禁止(delete myObject.a;)  

## Object.defineProperty和Proxy区别
区别|Object.defineProperty|Proxy
:---:|:---:|:---:
劫持对象|对对象属性劫持|对整个对象劫持
监听新增和删除|no|yes
监听数组方法|no<br>（需要变异原型方法）|yes
性能|不好<br>（对深层对象劫持要一次性递归）|好<br>（只在getter时才进行对象下一次属性的劫持）
兼容性|不兼容ie8<br>（ES5属性）|Proxy没办法被完全polyfill<br>（ES6属性）

电子表和机械表  
对整个对象劫持：wifi  
兼容性：艳丽广告牌  
监视新增和删除：防盗感应  
监听数组方法：电脑管理  
性能：调表员工  

## CommonJS和ES Module的区别
ES6 模块是编译时输出接口，是静态语法只能写在顶层。  
CommonJS 模块是运行时加载 是动态语法可以写在判断里，  
CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。  
CommonJs 是单个值导出，ES6 Module可以导出多个。  
CommonJs 的 this 是当前模块，ES6 Module的 this 是 undefined。  

插座 电瓶 电动车。
插座只能安在墙上；电瓶是骑车时才使用，可以选在拆下来。  
插座提供的是发电厂传来的电；电瓶只是充的电；  
插座可以有很多个；电池只有一个正负极接口；  
插座无所谓谁用它；电瓶指定的使用者是电动车。  


## Vue
#### .sync
> 当我们想要在父组件和子组件之间对某个属性值进行双向绑定时,有什么便捷的方式？是的只要.sync修饰符即可办到


```
// 父组件
<template>
  <div class="sync-parent">
    我是父组件: {{ text }}
    <Child :text.sync="text" />
  </div>
</template>

import Child from './child.vue'

export default {
  name: 'SyncParent',
  data () {
    return {
      text: 'parent'
    }
  },
  components: {
    Child,
  }
}

// 自组件
<template>
  <div class="child">
    我是子组件: 
    <input type="text" v-model="value" @input="onInput">
  </div>
</template>

export default {
  name: 'child',
  props: {
    text: {
      type: String
    }
  },
  data () {
    return {
      value: this.text
    }
  },
  methods: {
    onInput () {
      // 注意这里，必须是update:xxx的形式xxx即属性prop
      this.$emit('update:text', this.value)
    }
  }
}

作者：前端胖头鱼
链接：https://juejin.cn/post/7026867875990208543
```

## Vue实现
[简易版本vue的实现](https://www.cnblogs.com/aaron---blog/p/10577662.html)

Compile定义了不同绑定方式（指令、事件、插值文本）所对应的更新dom的方式，  
并通过递归解析出不同绑定方式里需要绑定的数据，  
然后根据绑定数据的值和对应绑定方式更新dom的方式，  
去初始化dom。  
同时创建一个以该绑定数据为监视数据、  
以该更新dom方式为其更新方式的Watcher实例。

Dep创建的实例用来存储Watcher实例。  
早在Compile之前，  
Observe就通过Object.defineProperty对数据进行了劫持，  
同时为对应数据添加一个Dep实例，  
当set数据时，通知所有Watcher实例进行更新；  
当get数据时，如果Dep.target不为空则将Dep.target作为Watcher实例存到dep实例中。  
而Compile并创建Watcher实例时，  
Watcher就是通过将自己赋值给Dep.target，  
同时访问对应数据触发数据的get方法，  
从而将某数据的Watcher添加到该数据对应的dep里。


```
//  数据响应
defineReactive(obj,key,val){
  //  解决数据层次嵌套
  this.observe(val);
  const dep = new Dep();
  Object.defineProperty(obj, key,{
    get(){
      //  向管理watcher的对象追加watcher实例
      //  方便管理
      Dep.target && dep.appDep(Dep.target);
      return val;
    },
    set(newVal){
      if(newVal === val){
          return;
      }
      val = newVal;
      // console.log(`${key}更新了：${newVal}`)
      dep.notify();
    }
  })
}
```
#### Observe的作用
监听数据变化。  
  创建一个Dep实例。  
  在数据被请求时：  
    1.判断是否为Dep访问（Dep.target不为空），  
      是则在Dep实例的监视数组中添加Dep.target（一个Watcher的实例）。  
    2.返回值。  
  在数据被改变时：  
    1.设置新值。  
    2.Dep实例通知其监视数组中的所有Watcher实例运行更新方法。  

```
class Dep {
  constructor() {
    //  存储
    this.deps = [];
  }
  //  添加watcher
  appDep(dep){
    this.deps.push(dep);
  }
  //  通知所有的watcher进行更新
  notify(){
    this.deps.forEach((dep) => {
      dep.update();
    })
  }
}
```
#### Dep的作用
管理Watcher实例。  
  Dep本身：  
    传输Watcher实例给对应的Dep实例。  
  Dep实例：  
    1.存储Watcher实例。  
    2.通知Watcher实例更新。  

```
class Watcher {
  constructor(vm,key,cb){
    //  Vue实例
    this.vm = vm;
    //  需要更新的key
    this.key = key;
    //  更新后执行的函数
    this.cb = cb;
    //  将当前watcher实例指定到Dep静态属性target
    //  用来在类间进行通信
    Dep.target = this;
    //  触发getter,添加依赖
    this.vm[this.key];
    Dep.target = null;
  }
  update(){
    this.cb.call(this.vm,this.vm[this.key]);
  }
}
```
#### Watcher的作用
  1.创建对应某属性的实例。  
  2.将自己的实例暂时传递给Dep.target并访问对应属性，  
    以达到通过Dep.target将Watcher实例添加到对应Dep实例的监视数组的目的。  
  3.定义更新方法。  

```
class Compile {
    constructor(el,vm) {
        //  要遍历的宿主节点
        this.$el = document.querySelector(el);
        this.$vm = vm;

        //  编译
        if(this.$el){
            //  转换宿主节点内容为片段Fragment元素
            this.$fragment = this.node2Fragment(this.$el);
            //  执行编译过程
            this.compile(this.$fragment);
            //  将编译完的HTML结果追加至宿主节点中
            this.$el.appendChild(this.$fragment);
        }
    }

    //  将宿主元素中代码片段取出来，遍历，这样做比较高效
    node2Fragment(el){
        const frag = document.createDocumentFragment();
        //  将宿主元素中所有子元素**（搬家，搬家，搬家）**至frag中
        let child;
        //  如果 el.firstChild 为undefined或null则会停止循环
        while(child = el.firstChild){
            frag.appendChild(child);
        }
        return frag;
    }

    compile(el){
        //  宿主节点下的所有子元素
        const childNodes = el.childNodes;
        Array.from(childNodes).forEach((node) => {
            if(this.isElement(node)){
                //  如果是元素
                console.log("编译元素"+node.nodeName)
                //  拿到元素上所有的执行,伪数组
                const nodeAttrs = node.attributes;
                Array.from(nodeAttrs).forEach((attr) => {
                    //  属性名
                    const attrName = attr.name; 
                    //  属性值
                    const exp = attr.value;     
                    //  如果是指令
                    if(this.isDirective(attrName)){
                        //  q-text
                        //  获取指令后面的内容
                        const dir = attrName.substring(2);
                        //  执行更新
                        this[dir] && this[dir](node,this.$vm,exp);
                    }
                    //  如果是事件
                    if(this.isEvent(attrName)){
                        //  事件处理
                        let dir = attrName.substring(1);    //  @
                        this.eventHandler(node,this.$vm,exp,dir);
                    }
                })
            }else if(this.isInterpolation(node)){
                //  如果是插值文本
                this.compileText(node);
                console.log("编译文本"+node.textContent)
            }
            //  递归子元素，解决元素嵌套问题
            if(node.childNodes && node.childNodes.length){
                this.compile(node);
            }
        })
    }
    //  是否为节点
    isElement(node){
        return node.nodeType === 1;
    }
    //  是否为插值文本
    isInterpolation(node){
        return node.nodeType === 3 && /\{\{(.*)\}\}/.test(node.textContent);
    }
    //  是否为指令
    isDirective(attr){
        return attr.indexOf("q-") == 0;
    }
    // 是否为事件
    isEvent(attr){
        return attr.indexOf("@") == 0;
    }

    //  v-text
    text(node,vm,exp){
        this.update( node, vm, exp, "text");
    }
    textUpdater(node,value){
        node.textContent = value;
    }

    //  双向绑定
    //  v-model
    model(node,vm,exp){
        //  指定input的value属性，模型到视图的绑定
        this.update(node,vm,exp,"model");
        //  试图对模型的响应
        node.addEventListener('input',(e) => {
            vm[exp] = e.target.value;
        })
    }
    modelUpdater(node,value){
        node.value = value;
    }

    //  v-html
    html(node,vm,exp){
        this.update(node,vm,exp,"html")
    }
    htmlUpdater(node,value){
        node.innerHTML = value;
    }
    
    //  更新插值文本
    compileText(node){
        let key = RegExp.$1;
        this.update( node, this.$vm, key, "text");
    }
    //  事件处理器
    eventHandler(node,vm,exp,dir){
        let fn = vm.$options.methods && vm.$options.methods[exp];
        if(dir && fn){
            node.addEventListener(dir,fn.bind(vm));
        }
    }

    //  更新函数 - 桥接
    update(node,vm,exp,dir){
        const updateFn = this[`${dir}Updater`];
        //  初始化
        updateFn && updateFn(node,vm[exp]);
        //  依赖收集
        new Watcher(vm,exp,function(value){
            updateFn && updateFn(node,value);
        })
    }
}
```
#### Compile的作用
查找并添加更新方式。  
  1.定义不同绑定类型（各种指令、事件或插值文本）的更新dom方式。  
  2.递归查找宿主节点中不同绑定类型中的值，即为数据要绑定的key。  
  3.初始化：根据要绑定的数据的value，执行相关更新方式。  
  4.创建要绑定的数据的Watcher实例，并将更新方式传给它。  


## 其它
#### 默认值
> 如果一个数组成员不严格等于undefined，默认值不会生效
```
[x = 1] = [undefined] // x:1
[y = 1] = [null] // y:null
```

#### 赋值和取值
> 赋值是倒着赋的，取值是正着取的
```
var a = {n: 1};
var b = a;
a.x = a = {n: 2};
console.log(a.x) 	
console.log(b.x)
```
所以最终的{n:2}是赋给b.x。

#### html、css、js设置缓存？
`?v=xxx`

#### concat
> 参数可以是具体的值，也可以是数组对象。可以是任意多个。如果要进行 concat() 操作的参数是数组，那么添加的是数组中的元素，而不是数组
```
[1,2,3].concat(4,5,6)   // [1,2,3,4,5,6]
[1,2,3].concat(4,[5,6]) // [1,2,3,4,5,6]
[1,2,3].concat([4,5,6]) // [1,2,3,4,5,6]

[1, [2, [3]]].flat(2)  // [1, 2, 3]
// 实现扁平化flat
function flatten(arr){
  while(arr.some(item => Array.isArray(item))){
    arr = [].concat(...arr)
  }
  return arr
}
```
[数组扁平化](https://juejin.cn/post/6946022649768181774#heading-8)

#### 字符串模版
[字符串模版](https://juejin.cn/post/6946022649768181774#heading-12)
```
function render(template, data) {
  const reg = /\{\{(\w+)\}\}/; // 模板字符串正则
  if (reg.test(template)) { // 判断模板里是否有模板字符串
    const name = reg.exec(template)[1]; // 查找当前模板里第一个模板字符串的字段
    template = template.replace(reg, data[name]); // 将第一个模板字符串渲染
    return render(template, data); // 递归的渲染并返回渲染后的结构
  }
  return template; // 如果模板没有模板字符串直接返回
}
```