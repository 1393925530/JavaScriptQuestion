# JavaScript 易错点

## 1.['1', '2', '3'].map(parseInt) what & why ?
* 首先map函数的第一个参数callback：
```JavaScript
var new_array = arr.map(function callback(currentValue[, index[, array]]) { // Return element for new_array }[, thisArg])
```
callback一共接受三个参数，第一个参数currentValue当前被处理的元素，第二个参数index代表该元素的索引。
* parseInt用来解析字符串，使字符串成为指定基数的整数。
```JavaScript
parseInt(string, radix)
```
```JavaScript
parseInt('123', 5) //将'123'看作5进制数，返回十进制数38 => 1 * 5 ^ 2 + 2 * 5 ^ 1 + 3 * 5 ^ 0
```
* 模拟运行
1. parseInt('1', 0) //radix为0时，且string参数不以'0x'和'0'开头时，按照基数为10处理。返回1。
2. parseInt('2', 1) //1进制表示的数，最大值小于2，无法解析，返回NaN。
3. parseInt('3', 2) //2进制表示的数，最大值小于3，无法解析，返回NaN。
* map函数返回数组，最后结果为[1, NaN, NaN]
### 变形
```JavaScript
[10, 10, 10, 10, 10].map(parseInt) //[10, NaN, 2, 3, 4]
```
1. parseInt('10', 0) //10
2. parseInt('10', 1) //NaN
3. parseInt('10', 2) //2进制只可能出现0和1，结果为2 => 1 * 2 ** 1 + 0 * 2 ** 0
```JavaScript
let unary = fn => val => fn(val)
let parse = unary(parseInt)
console.log(['1.1', '2', '0.3'].map(parse)) //[1, 2, 0]
```
* 将parseInt放进unary的意思是返回一个只带一个参数的parseInt函数出来。
* parseInt第二个参数默认为十进制。
```JavaScript
let unary = fn => (val, radix) => fn(val, radix)
let parse = unary(parseInt)
console.log(['1.1', '2', '0.3'].map(parse)) //[1, NaN, 0]
```
* 将parseInt两个参数都放进unary，运行结果和 ['1.1', '2', '0.3'].map(parseInt) 相同
## 写 React / Vue 项目时为什么要在列表组件中写 key，其作用是什么？
>vue和react都采用diff算法来对比新旧虚拟节点，从而更新节点。在交叉对比中，当新节点跟旧节点**头尾交叉对比**没有结果时，会根据新节点的key去对比旧节点数组中的key，从而找到相应旧节点(这里对应的是key => index的**map映射**)。如果未找到就认为是一个新增节点。如果没有key，就会采用**遍历查找**的方式找到对应的旧节点。前者是map映射，后者是遍历查找。相对而言，**map映射**的速度更快。
## 什么是防抖和节流？有什么区别？如何实现？
1. 防抖
>触发高频事件后n秒内只会执行一次，如果n秒内高频事件再次被触发，则重新计算时间。
* 思路：
>每次触发事件时都取消之前的延时调用方法。
```JavaScript
function debounce (fn) {
  let timeout = null //创建一个标记用来存放定时器的返回值
  return function () {
    clearTimeout(timeout) //当用户输入时把前一个定时器清除
    timeout = setTimeout(() => {
      fn.apply(this, arguments)
    }, 500)
  }
}
function sayHi () {
  console.log('防抖成功')
}
var inp = document.getElementById('inp')
inp.addEventListener('input', debounce(sayHi)) //防抖
```
2. 节流
>高频事件触发，但在n秒内只执行1次，所以节流会稀释函数的执行频率。
* 思路：
>每次触发事件时都判断当前是否有等待执行的延时函数。
```JavaScript
function throttle (fn) {
  let canRun = true //通过闭包保存一个标记
  return function () {
    if (!canRun) return false //在函数开头判断标记是否为true，不为true则return
    canRun = false //立即置false
    setTimeout(() => { //将外部传入的函数的执行放到setTimeout中
      fn.apply(this, arguments) //定时器执行完毕后再把标记设置为true表示可以进行下一次循环，定时器没有执行时标记永远是false，在开头被return
      canRun = true
    }, 500)
  }
}
function sayHi (e) {
  console.log(e.target.innerWidth, e.target.innerHeight)
}
window.addEventListener('resize', throttle(sayHi))
```
## 介绍下 Set、Map、WeakSet 和 WeakMap 的区别？
Set和Map主要的应用场景在于数据重组和数据储存。<br/>
Set是叫做**集合**的数据结构，Map是叫做**字典**的数据结构。
1. 集合(Set)

ES6新增的一种新的数据结构，类似于数组，但成员唯一且无序，无重复值。
**Set本身是一种构造函数，用来生成Set数据结构**
```JavaScript
new Set([iterable])
```
```JavaScript
const s = new Set()
[1, 2, 3, 4, 3, 2, 1].forEach(x => s.add(x)) //Set的add方法，在Set对象尾部添加一个元素。返回该Set对象。
for(let i of s) {
  console.log(i) //1 2 3 4
}

//去重数组的重复对象
let arr = [1, 2, 3, 2, 1, 1]
[...new Set(arr)] //console报错..建议分步..
```
向Set加入值的时候，不会发生类型转换，所以5和'5'是两个不同的值。Set内部判断两个值是否不同，使用的算法叫'Same-value-zero equality'，类似于**精确相等**运算符(===)，主要区别是**NaN等于自身，而精确相等运算符认为NaN不等于自身。**
```JavaScript
let set = new Set()
let a = NaN
let b = NaN
set.add(a)
set.add(b)
set //Set(1) {NaN}

let set1 = new Set()
set1.add(5)
set1.add('5')
set1 //Set(2) {5, "5"}
```
* Set实例属性
  - constructor：构造函数
  - size：元素数量
  ```JavaScript
  let set = new Set([1, 2, 3, 2, 1])
  console.log(set.length) //undefined
  console.log(set.size) //3
  ```
* Set实例方法
  - 操作方法
    - add(value)：新增，相当于array的push
    - delete(value)：存在即删除集合中的value
    - has(value)：判断集合中是否存在value
    - clear()：清空集合
    ```JavaScript
    let set = new Set()
    set.add(1).add(2).add(1)
    set //Set(2) {1, 2}
    set.has(1) //true
    set.has(3) //false
    set.delete(1)
    set.has(1) //false
    ```
    ```Array.from```方法可以将Set结构转为数组
    ```JavaScript
    const items = new Set([1, 2, 3, 2])
    const array = Array.from(items)
    console.log(array) //[1, 2, 3]
    // 或
    const arr = [...items]
    console.log(arr) //[1, 2, 3]
    ```
  - 遍历方法(遍历顺序为插入顺序)
    - keys()：返回一个包含集合中所有键的迭代器
    - values()：返回一个包含集合中所有值的迭代器
    - entries()：返回一个包含Set对象中所有元素的键值对迭代器
    - forEach(callbackFn, thisArg)：用于对集合成员执行callbackFn操作，若提供thisArg参数，回调中的this会是这个参数，没有返回值
    ```JavaScript
    let set = new Set([1, 2, 3])
    console.log(set.keys()) //SetIterator {1, 2, 3}
    console.log(set.values()) //SetIterator {1, 2, 3}
    console.log(set.entries()) //SetIterator {1 => 1, 2 => 2, 3 => 3}
    for (let item of set.keys()) {
      console.log(item)
    } // 1 2 3
    for (let item of set.entries()) {
      console.log(item)
    } //(2) [1, 1]
      //(2) [2, 2]
      //(2) [3, 3]
    set.forEach((value, key) => {
      console.log(key + ': ' + value)
    })
    console.log([...set]) //[1, 2, 3]
    ```
    Set可默认遍历，默认迭代器生成函数是**values()方法**
    ```JavaScript
    Set.prototype[Symbol.iterator] === Set.prototype.values //true
    ```
    所以，Set可以使用map、filter方法
    ```JavaScript
    let set = new Set([1, 2, 3])
    set = new Set([...set].map(item => item * 2))
    console.log([...set]) //[2, 4, 6]

    set = new Set([...set].filter(item => (item >= 4)))
    console.log([...set]) //[4, 6]
    ```
    因此，Set容易实现交集(Intersect)、并集(Union)、差集(Difference)
    ```JavaScript
    let set1 = new Set([1, 2, 3])
    let set2 = new Set([4, 3, 2])

    let intersect = new Set([...set1].filter(value => set2.has(value)))
    let union = new Set([...set1, ...set2])
    let difference = new Set([...set1].filter(value => !set2.has(value)))
    console.log(intersect) //Set(2) {2, 3}
    console.log(union) //Set(4) {1, 2, 3, 4}
    console.log(difference) //Set(1) {1}
    ```
2. WeakSet

WeakSet对象允许将弱引用对象储存在一个集合中

WeakSet和Set的区别：
* WeakSet只能储存对象引用，不能存放值，而Set对象可以
* WeakSet对象中储存的对象值都是被弱引用的，即垃圾回收机制不考虑WeakSet对该对象的引用。如果没有其他的变量或属性引用这个对象值，则这个对象会被垃圾回收掉(不考虑该对象还存在于WeakSet中)，所以，WeakSet对象里有多少个成员元素，取决于垃圾回收有没有运行，运行前后成员个数可能不一致，遍历结束之后，有的成员可能取不到了(被垃圾回收了)，WeakSet对象是无法被遍历的，也没有办法拿到它包含的所有元素

属性：
* constructor：构造函数，任何一个具有Iterable接口的对象，都可以作参数
```JavaScript
const arr = [[1, 2], [3, 4]]
const weakset = new WeakSet(arr)
console.log(weakset) //WeakSet {Array(2), Array(2)}
```
方法：
* add(value)：在WeakSet对象中添加一个元素value
* has(value)：判断WeakSet对象中是否包含value
* delete(value)：删除元素value
* clear()：清空所有元素，**该方法已被废弃**
```JavaScript
var ws = new WeakSet()
var obj = {}
var foo = {}

ws.add(window)
ws.add(obj)

ws.has(window) //true
ws.has(foo) //false

ws.delete(window) //true
ws.has(window) //false
```
3. 字典(Map)
集合 和 字典 的区别：
 * 共同点：集合、字典可以储存不重复的值
 * 不同点：集合 是以[value, value]的形式储存元素，字典是以[key, value]的形式储存
 ```JavaScript
 const m = new Map()
 const o = { p: 'haha' }
 m.set(o, 'content')
 m.get(o) //content

 m.has(o) //true
 m.delete(o)  //true
 m.has(o) //false
 ```

 **任何具有Iterator接口、且每个成员都是一个双元素的数组的数据结构**都可以当作```Map```构造函数的参数，例如：
 ```JavaScript
 const set = new Set([
   ['foo', 1],
   ['bar', 2]
 ])
 const m1 = new Map(set)
 m1.get('foo') //1

 const m2 = new Map([['baz', 3]])
 const m3 = new Map(m2)
 m3.get('baz') //3
 ```
 如果读取一个未知的键，则返回```undefined```。
 ```JavaScript
 new Map().get('aaa') //undefined
 ```
 注意，只有对同一个对象的引用，Map结构才将其视为同一个键。
 ```JavaScript
 const map = new Map()

 map.set(['a'], 555)
 map.get(['a']) //undefined
 ```
undefined的原因在于['a']和['a']并不是同一个对象的引用，可修改为：
 ```JavaScript
 const map = new Map()
 var arr = ['a']
 map.set(arr, 555)
 map.get(arr) //555
 ```
 Map的键实际上是跟内存地址绑定的，内存地址不一样，视为两个键。这就解决了同名属性碰撞问题，在拓展别人的库的时候，如果使用对象作为键名，就不用担心属性与原作者属性名同名。

 如果Map的键是一个简单类型的值(数字、字符串、布尔值)，则只要两个值严格相等，Map将其视为一个键，。比如0和-0就是一个键，布尔值true和字符串true则是两个不同的键。undefined和null也是两个不同的键。NaN不严格相等于自身，但Map将其视为同一个键。
 ```JavaScript
 let map = new Map()

 map.set(-0, 123)
 map.get(+0) //123

 map.set(true, 1)
 map.set('true', 2)
 map.get(true) //1

 map.set(undefined, 3)
 map.set(null, 4)
 map.get(undefined) //3
 ```
 Map的属性及方法
 屬性：
 * constructor：構造函數
 * size：返回字典中所包含的元素个数
 ```JavaScript
 const map = new Map([
   ['name', 'An'],
   ['des', 'JS']
 ])
 map.size //2
 ```
 操作方法：
 * set(key, value)：向字典中添加新元素
 * get(key)：通过键查找特定的数值并返回
 * has(key)：判断字典中是否中存在键key
 * delete(key)：通过键key从字典中移除对应的数据
 * clear()：将字典中的所有元素删除
 遍历方法：
 * keys()：将字典中包含的所有**键名**以迭代器形式返回
 * values()：将字典中包含的所有**数值**以迭代器形式返回
 * entries()：返回所有成员的迭代器
 * forEach()：遍历字典的所有成员
 ```JavaScript
 const map = new Map([
   ['name', 'An'],
   ['des', 'JS']
 ])
 console.log(map.entries()) //MapIterator {"name" => "An", "des" => "JS"} 
 console.log(map.keys()) //MapIterator {"name", "des"}
 ```
 Map结构的默认遍历器接口(```Symbol.iterator```属性)，也就是```entries```方法。
 ```JavaScript
 Map[Symbol.iterator] === Map.entries //true
 ```
 Map结构转换为数组结构，比较快速的方法是用扩展运算符(...)

 与其他数据结构的相互转换：

 1. Map转Array
 ```JavaScript
 const map = new Map([[1, 1], [2, 2], [3, 3]])
 console.log([...map]) // [[1, 1], [2, 2], [3, 3]]
 ```
 2. Array转Map
 ```JavaScript
 const map = new Map([[1, 1], [2, 2], [3, 3]])
 console.log(map) //Map(3) {1 => 1, 2 => 2, 3 => 3}
 ```
 3. Map转Object
 因为Object的键名都为字符串，而Map的键名为对象，所以转换的时候会把非字符串键名转换为字符串键名。
 ```JavaScript
 function mapToObj (map) {
   let obj = Object.create(null)
   for (let [key, value] of map) {
     obj[key] = value
   }
   return obj
 }
 const map = new Map().set('name', 'An').set('des', 'JS')
 mapToObj(map) //{name: "An", des: "JS"}
 ```
 4. Object转Map
 ```JavaScript
 function objToMap (obj) {
   let map = new Map()
   for(let key of Object.keys(obj)) {
     map.set(key, obj[key])
   }
   return map
 }
objToMap({'name': 'An', 'des': 'JS'}) //Map(2) {"name" => "An", "des" => "JS"}
 ```
 5. Map转JSON
 ```JavaScript
 function mapToJSON (map) {
   return JSON.stringify([...map])
 }
 let map = new Map().set('name', 'An').set('des', 'JS')
 mapToJSON(map) //"[["name","An"],["des","JS"]]"
 ```
 6. JSON转Map
 ```JavaScript
 function jsonToStrMap (jsonStr) {
   return objToMap(JSON.parse(jsonStr))
 }
 jsonToStrMap('{"name": "An", "des": "JS"}')
 ```

4. WeakMap
WeakMap对象是一组键值对的集合，其中的**键是弱引用对象，而值可以是任意。**<br/>
**WeakMap弱引用的只是键名，而不是键值。键值依然可以正常引用。**<br/>
WeakMap中，每个键对自己所引用对象的引用都是弱引用，在没有其他引用和该键引用同一对象，这个对象将会被垃圾回收(相应的key变成无效的)，所以WeakMap的key是不可枚举的。<br/>
属性：
* constructor：构造函数
方法：
* has(key)：判断是否有key关联对象
* get(key)：返回key关联对象(没有则则返回undefined)
* set(key)：设置一组key关联对象
* delete(key)：移除key的关联对象
```JavaScript
let myElement = document.getElementById('logo')
let myWeakMap = new WeakMap()
myWeakMap.set(myElement, {timesClicked: 0})
myElement.addEventListener('click', function () {
  let logoData = myWeakMap.get(myElement)
  logoData.timesClicked++
}, false)
```
5. 总结
* Set
  - 成员唯一、无序且不重复
  - [value, value]，键值与键名是一致的(或者说只有键值，没有键名)
  - 可以遍历，方法有：add、delete、has
* WeakSet
  - 成员都是对象
  - 成员都是弱引用，可以被垃圾回收机制回收，可以用来保存DOM节点，不容易造成内存泄漏
  - 不能遍历，方法有add、delete、has
* Map
  - 本质上是键值对的集合，类似集合
  - 可以遍历，方法很多可以跟各种数据格式转换
* WeakMap
  - 只接受对象作为键名(null除外)，不接受其他类型的值作为键名
  - 键名是弱作用，键值可以是任意的，键名所指向的对象可以被垃圾回收，此时键名是无效的
  - 不能遍历，方法有get、set、has、delete
## 介绍下深度优先遍历和广度优先遍历，如何实现？
```HTML
<body>
  <div class="parent">
    <div class="child-1">
      <div class="child-1-1">
        <div class="child-1-1-1">
          a
        </div>
      </div>
      <div class="child-1-2">
        <div class="child-1-2-1">
          b
        </div>
      </div>
      <div class="child-1-3">
        c
      </div>
    </div>
    <div class="child-2">
      <div class="child-2-1">
        d
      </div>
      <div class="child-2-2">
        e
      </div>
    </div>
    <div class="child-3">
      <div class="child-3-1">
        f
      </div>
    </div>
  </div>
</body>
```
### 深度优先遍历
深度优先遍历DFS与树的先序遍历类似。假设初始状态是图中所有顶点均未被访问，则从某个顶点v出发，首先访问该顶点然后依次从它的各个未被访问的邻接点出发深度优先搜索遍历图，直至图中所有和v有路径相通的顶点都被访问到。若此时尚有其他顶点未被访问到，则另选一个未被访问的顶点作起始点，重复上述过程，直至图中所有顶点都被访问到为止。
```JavaScript
//深度优先遍历三种方式
let deepTravelsal1 = (node, nodeList = []) => {
  if (node !== null) {
    nodeList.push(node)
    let children = node.children
    for (let i = 0; i < children.length; i++) {
      deepTravelsal1(node[i], nodeList)
    }
  }
  return nodeList
}
let deepTravelSal2 = (node) => {
  let nodes = []
  if (node !== null) {
    nodes.push(node)
    let children = node.children
    for (let i = 0; i < children.length; i++) {
      nodes = nodes.concat(deepTravelSal2(children[i]))
    }
  }
  return nodes
}
// 非递归 ?
let deepTraversal3 = (node) => {
  let stack = []
  let nodes = []
  if (node) {
    // 推入当前处理的node
    stack.push(node)
    while (stack.length) {
      let item = stack.pop() //从数组中删除最后一个元素，并返回该元素的值
      let children = item.children
      nodes.push(item)
      // node = [] stack = [parent]
      // node = [parent] stack = [child3,child2,child1]
      // node = [parent, child1] stack = [child3,child2,child1-2,child1-1]
      // node = [parent, child1-1] stack = [child3,child2,child1-2]
      for (let i = children.length - 1; i >= 0; i--) {
        stack.push(children[i])
      }
    }
  }
  return nodes
}
```
### 广度优先遍历
广度优先遍历BFS从图中某顶点V出发，在访问了V之后依次访问V的各个未曾访问过的邻接点出发依次访问它们的邻接点，并使得先被访问的顶点的邻接点先于后被访问的顶点的邻接点被访问的顶点的邻接点被访问，直至图中所有已被访问的顶点的邻接点被访问到。如果此时图中尚有顶点未被访问，则需要另选一个未曾被访问过的顶点作为新的起始点，重复上述过程，直至图中所有顶点都被访问到为止。
```JavaScript
let widthTraversal2 = (node) => {
  let nodes = []
  let stack = []
  if (node) {
    stack.push(node)
    while (stack.length) {
      let item = stack.shift() //从数组中删除第一个元素，并返回该元素的值
      let children = item.children
      nodes.push(item)
      for (let i = 0; i < children.length; i++) {
        stack.push(children[i])
      }
    }
  }
  return nodes
}
```

## 请分别用深度优先思想和广度优先思想实现一个拷贝函数？
```JavaScript
//工具函数
let _toString = Object.prototype.toString
let map = {
  array: 'Array',
  object: 'Object',
  function: 'Function',
  string: 'String',
  null: 'Null',
  undefined: 'Undefined',
  boolean: 'Boolean',
  number: 'Number'
}
let getType = (item) => {
  return _toString.call(item).slice(8, -1)
}
let isTypeOf = (item, type) => {
  return map[type] && map[type] === getType(item)
}
```
深复制 深度优先遍历
```JavaScript
let DFSdeepClone = (obj, visitedArr = []) => { //在第一次调用函数时，visitedArr是空数组
  let _obj = {}
  if (isTypeOf(obj, 'array') || isTypeOf(obj, 'object')) {
    let index = visitedArr.indexOf(obj) //可返回数组中某个指定的元素位置
    _obj = isTypeOf(obj, 'array') ? [] : {}
    if (~index) {
      _obj = visitedArr[index]
    } else {
      visitedArr.push(obj)
      for (let item in obj) {
        _obj[item] = DFSdeepClone(obj[item], visitedArr)
      }
    }
  } else if (isTypeOf(obj, 'function')) {
    _obj = eval('(' + obj.toString() + ')')
  } else {
    _obj = obj
  }
  return _obj
}
```
广度优先遍历 //TODO
## ES5/ES6 的继承除了写法以外还有什么区别？
1. ```class```声明会提升，但不会初始化赋值。```Foo```进入暂时性死区，类似于```let```、```const```声明变量。
```JavaScript
const bar = new Bar()
function Bar () {
  this.bar = 42
}

const foo = new Foo() //Uncaught ReferenceError: Cannot access 'Foo' before initialization
class Foo {
  constructor() {
    this.foo = 42
  }
}
```
2. ```class```声明内部会启用严格模式。
```JavaScript
//引用一个未声明的变量
function Bar () {
  baz = 42
}
const bar = new Bar()

class Foo {
  constructor() {
    fol = 42
  }
}
const foo = new Foo() //Uncaught ReferenceError: fol is not defined
```
3. ```class```的所有方法(包括静态方法和实例方法)都是不可枚举的。
```JavaScript
//引用一个未声明的变量
function Bar () {
  this.bar = 42
}
Bar.answer = function () {
  return 42
}
Bar.prototype.print = function () {
  console.log(this.bar)
}
const barKeys = Object.keys(Bar) //["answer"]
const barProtoKeys = Object.keys(Bar.prototype) //["print"]

class Foo {
  constructor () {
    this.foo = 42
  }
  static answer () {
    return 42
  }
  print () {
    console.log(this.foo)
  }
}
const fooKeys = Object.keys(Foo) //[]
const fooProtoKeys = Object.keys(Foo.prototype) //[]
```
4. ```class```的所有方法(包括静态方法和实例方法)都没有原型对象prototype，所以也没用```[[construct]]```，不能使用```new```来调用。
```JavaScript
function Bar () {
  this.bar = 42
}
Bar.prototype.print = function () {
  console.log(this.bar)
}

const bar = new Bar()
const barPrint = new bar.print()

class Foo {
  constructor() {
    this.foo = 42
  }
  print () {
    console.log(this.foo)
  }
}
const foo = new Foo()
const fooPrint = new foo.print() //Uncaught TypeError: foo.print is not a constructor
```
5. 必须使用```new```调用```class```。
```JavaScript
function Bar () {
  this.bar = 42
}
const bar = Bar()

class Foo {
  constructor() {
    this.foo = 42
  }
}
const foo = Foo() //Uncaught TypeError: Class constructor Foo cannot be invoked without 'new'
```
6. ```class```内部无法重写类名。
```JavaScript
function Bar () {
  Bar = 'Baz'
  this.bar = 42
}
const bar = Bar()

class Foo {
  constructor () {
    this.foo = 42
    Foo = 'Fol' //Uncaught TypeError: Assignment to constant variable.
  }
}
const foo = new Foo()
Foo = 'Fol'
```