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
<!-- ## 介绍下 Set、Map、WeakSet 和 WeakMap 的区别？
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
``` -->