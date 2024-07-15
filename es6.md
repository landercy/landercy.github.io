# ES6

## 变量声明

使用`let`代替`var`声明变量

```js
var a = 10;
console.log(window.a) // 10

# 不能二次声明
let a = 10; // Uncaught SyntaxError: Identifier 'a' has already been declared

# 不是全局对象window的属性
let b = 10;
console.log(window.b) // undefined

# 块级作用域
{
  let d = 10; 
}
console.log(d) // ncaught ReferenceError: d is not defined
```

## 常量声明

```js
# ES5
Object.defineProperty(window, 'PI', {
  value: 3.14,
  writable: false
})
console.log(PI) // 3.14
PI = 5
console.log(PI) // 3.14

# ES6
const PI = 3.14
console.log(PI) // 3.15
PI = 5 // Uncaught TypeError: Assignment to constant variable.

# 基本数据类型存储在"栈内存"中，引用数据类型存储在"堆内存"中，然后在栈内存中保存"引用地址"。
# const保证了"栈内存"中数据不可变，即"引用地址"不变
const obj = {
  name: 'Lan',
  age: 35
}
obj.age = 36
console.log(obj) // {name: "Lan", age: 36}

Object.freeze(obj)
obj.age = 37
console.log(obj) // {name: "Lan", age: 36}
```

## 解构赋值

```js
# 数组解构
let [a,b,c] = [1,2,3]

# 对象解构
const contact = {name: 'Lan', age: 35, friend: {name: 'Vam', age: 40}} 

let {name, age, friend} = contact
console.log(name, age, friend) // Lan 35 {name: "Vam", age: 40}

let {name: selfName, age: selAge, friend: {name: friendName, age: friendAge}} = contact
console.log(selfName,selAge, friendName, friendAge) // Lan 35 Vam 40

let {name, age, friend: {name, age}} = contact // Uncaught SyntaxError: Identifier 'name' has already been declared

# let {name, ...data} = contact
console.log(name, data) // Lan, {name: "Lan", age: 35, friend: {…}}
```

## 数组遍历

```js
let arr = [1, 2, 3]

# for
for (let i = 0; i < arr.length; i++) {
 	console.log(arr[i])
}

# ES6 for...of
for (let v of arr){
  console.log(v) 
}
```