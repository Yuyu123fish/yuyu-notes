# 一，类型
## 1 类型介绍
ts相比js更加严格，最后仍会被编译为js执行。
```ts
//显式指定
let a : string
let b : number
let c : boolean

function count(x:number, y:number) : number{
	return x + y
}
console.log(count(1,2))
//3

//类型推断
let d = 'hello'
d = 99//错误
```
同时，`string`类型和`String`类型不能混用，前一个是基本类型，后一个是包装类型。和Java一样，TypeScript也存在自动装箱和自动拆箱，比如`string.length`会自动装箱为`Stirng`然后再获取其`length`属性。
## 2 常用类型
`let a : any`：表示任意类型。
`let a : unknown`：表示未知类型，一个类型安全的`any`，表示暂时无法确定类型，以后可能会确定。
```ts
let a : unknown
a = 'hello'
let b : string
b = a//错误
//使用以下三种类型断言可以解决
if(typeof a === 'string'){
	b = a
}
b = a as string
b = <string> a
```
`let a : never`：不能有值，没有任何意义，用处很少。
`function my_function(a:string):void{//func}`：函数没有返回值。内部默认`return undefined`，即使不写也会返回。和`undefined`不同的是，`void`的函数调用者不应该关注其函数返回值，而`undefined`可以被调用者显式调用，比如`if(undefined)`。
`let a : object`：开发中用得少，范围太大了。`object`可以存所有对象类型，不能存基类型。`Object`可以存所有会调用到`Object`方法的类型，比如`a.toString()`。
## 3 声明对象类型
1. 实际开发中限制一般对象，常用以下形式
```ts
//name为必须的属性，age为可选的属性
//可以使用 换行，逗号，分号对属性进行分隔
let person1 : {
	name : string
	age? : number
}
let person2 : {name : string, age? : number}
let person3 : {name : string; age? : number}
//对象间的属性必须加','进行分隔，不管是否换行
person1 = {name:'李四', age:18}
person2 = {name:'张三'}
```
2. 索引签名
```ts
let person : {
	name : string
	age? : number
	[key:string] : any
}
person = {name:'张三', age:18, gender:'男', city:'北京'}
```
## 4 声明函数类型
```ts
let count : (a:number, b:number) => number
count = function(a, b){
	return a+b
}
```
- ts中的`=>`是函数类型的声明，描述其**参数类型**和**返回类型**，相当于一种约定。
- js中的`=>`是定义函数的语法，是函数的具体实现。
## 5 tuple，enum，type
`tuple`是一种特殊的数组类型，可以存储固定数量的元素。
`enum`枚举类，定义一组命名常量，增强代码的可读性与可维护性。
`type`是自定义类型，可以自己定义出一个类型，用来表示其他类型。
```ts
//联合类型
type Status = string | number
function printStatus(data:Status):void{
	console.log(data)
}
//交叉类型
type Area = {
	height: number
	width: number
}
type Address = {
	num: number
	cell: number
	room: number
}
type House = Area & Address

const house:House{
	height: 100,  
	width: 80,  
	num: 1,  
	cell: 2,  
	room: 301
}
```
## 6 特殊情况
使用**类型声明**限制函数返回值为`void`时，`ts`并不会**严格要求**函数返回为空，例如以下代码：
```ts
type LogFunc = () => void
//允许返回非空值
const f1 : LogFunc = () => {
	return 666
}
```
原因是为了给`array1.forEach((el) => array2.push(el))`做让步，因为这个`forEach`里面会默认`return`第二个数组`push`后的结果，也就是长度，如果说严格要求函数返回为空，则两个规则互相冲突了。