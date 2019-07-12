关于声明提前的问题以及function()与var的优先级
1.var 在正式执行脚本之前,将所有的var声明的变量,提前到当前作用域的顶部,集中声明创建,然后开始执行,但是赋值留在本地如. 

```javascript
console.log(n);
var n=100;
comsole.log(n);
/****
等效于
var n;//声明提前
console.log(n);//undefined
n=100;//赋值留下来
console.log(n);//100
***/
```

2 function(),同上 .插入一句 function()的优先级大于var,也就是说 function()会在var 的前面

```javascript
function fun(){
 return 10;
}
function fun(){
 return 20;
}
console.log(fun());
var fun=100;
console.log(fun());
```

上面代码解析顺序是这样

```javascript
function fun(){
return 10;
}
function fun(){//覆盖上面
return 20;
}
var fun;//声明提前
console.log(fun());//20
fun=100;//赋值留在本地fun内存里被赋值为100
console.log(fun());//因为上面被赋值不再指向函数,所以会报错not a function
```


上面这几个例子基本上就明了了.注意两个文题一是作用域只针对当前,不会提前到其他作用域.二是我当时在学习过程中误以为所有所有function(),都声明优先级都大于var ,但是前提是建立在同一元素上.如 function a() ,和var a 遵循原则 而 function a() ,和var b 该是什么顺序就是什么顺序.

再写一个局部作用域与全局作用域问题
如果局部作用域里有声明的变量,那么改变的值就是局部变量,如果局部作用域里没有已声明的变量,再用的全局声明的变量.(哈哈如果都没有话当然会报错)

```javascript
var n=1;//全局
function fun(){
console.log(n);//当前作用域有声明用当前作用域变量n 结果为undefined
var n=2;//局部 var=n;提前到当前作用域(函数)顶部 n=2 留下来
console.log(n);// 2
}
fun(); // undefined   2
console.log(n); //这个当前作用域全局  结果为1
```



用几个例子基本就可以解决理解这个问题了.
--------------------- 