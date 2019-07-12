#  ES6数组新增的几个方法 forEach() 、map()、filter()、reduce()、some()、every()

2019年05月10日 10:52:08 [为什么名字都被占用](https://me.csdn.net/qq_37588752) 阅读数 19



关于数组中forEach() 、map()、filter()、reduce()、some()、every()的总结
1、forEach()

```javascript
var arr = [1,2,3,4];



arr.forEach((item,index,arr) => {



    console.log(item) //结果为1,2,3,4



})
```

//forEach遍历数组，无返回值，不改变原数组，仅仅只是遍历、常用于注册组件、指令等等。


2、map()

```javascript
var arr = [1,2,3,4];



arr.map((item,index,arr) => {



    return item*10 //新数组为10,20,30,40



})
```

//map遍历数组，返回一个新数组，不改变原数组的值。


3、filter()

```javascript
var arr = [1,2,3,4];



arr.filter((item,index,arr) => {



    return item > 2 //新数组为[3,4]



})
```

//filter过滤掉数组中不满足条件的值，返回一个新数组，不改变原数组的值。

4、reduce()

```javascript
var arr = [1,2,3,4];



arr.reduce((result,item,index,arr) => {



    console.log(result) // 1  3  6  result为上次一计算的结果



    console.log(item)  // 2  3  4



    console.log(index) // 1  2  3



    return result+item //最终结果为10



})
```

//reduce 让数组的前后两项进行某种计算。然后返回其值，并继续计算。不改变原数组，返回计算的最终结果，从数组的第二项开始遍历。
  
5、some()

```javascript
var arr = [1,2,3,4];



arr.some((item,index,arr) => {



    return item > 3 //结果为true



})
```

//遍历数组每一项，有一项返回true,则停止遍历，结果返回true。不改变原数组


6、every()

```javascript
var arr = [1,2,3,4];



arr.every((item,index,arr) => {



    return item > 1 //结果为false



})
```

//遍历数组每一项，每一项返回true,则最终结果为true。当任何一项返回false时，停止遍历，返回false。不改变原数组

 


以上6个方法均为ES6语法，IE9及以上才支持。不过可以通过babel转意支持IE低版本。 
以上均不改变原数组。 
some、every返回true、false。 
map、filter返回一个新数组。 
reduce让数组的前后两项进行某种计算，返回最终操作的结果。 
forEach 无返回值。