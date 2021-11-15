测试换行1  
测试换行2
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
监视新增和删除：防盗感应  
监听数组方法：  

## Vue
[简易版本vue的实现](https://www.cnblogs.com/aaron---blog/p/10577662.html)

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
  1.定义不同绑定类型（各种指令、事件或插值文本）的更新方式。  
  2.递归查找宿主节点中不同绑定类型中的值，即为数据要绑定的key。  
  3.初始化：根据要绑定的数据的value，执行相关更新方式。  
  4.创建要绑定的数据的Watcher实例，并将更新方式传给它。  


## 其它
> 如果一个数组成员不严格等于undefined，默认值不会生效
```
[x = 1] = [undefined] // x:1
[y = 1] = [null] // y:null
```
***
> 赋值是倒着赋的，取值是正着取的
```
var a = {n: 1};
var b = a;
a.x = a = {n: 2};
console.log(a.x) 	
console.log(b.x)
```
所以最终的{n:2}是赋给b.x。
***
html、css、js设置缓存？
`?v=xxx`
***