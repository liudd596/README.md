### 《javascript高级程序设计》笔记： 引用类型 Function
**迭代方法**  EC5定义了5个迭代方法。每个方法都接收两个参数：
- **要在每一项上运行的函数；** 
- **（可选的）运行该函数的作用域对象--影响this的值。**   

传入这些方法中的函数会接收三个参数：数组项的值(item)、该项在数组中的位置(index)、数组对象本身(array)。   
以下是这5个迭代方法的作用：   
-every():  对数组中的每一项运行给定函数，如果该函数对每一项都返回true，则返回true。   
-some():   对数组中的每一项运行给定函数，如果该函数对任意一项都返回true，则返回true。   
-filter(): 对数组中的每一项运行给定函数，返回该函数会返回true的项组成的数组。  
-map():    对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组。  
-forEach():对数组中的每一项运行给定函数，这个方法没有返回值。  
```javascript
var number = [1,2,3,4,5,4,3,2,1]

var everyResult = number.every(function(item, index, array){
    return (item < 2);
});
alert(everyResult);   //false

var someResult = number.some(function(item, index, array){
    return (item < 2);
});
alert(someResult);    //true

var filterResult = number.filter(function(item, index, array){
    return (item < 2);
});
alert(filterResult);   // [3,4,5,4,3]

var mapResult = number.map(function(item, index, array){
    return item * 2;
});
alert(mapResult);      // [2,4,6,8,10,8,6,4,2]

```
### 《javascript高级程序设计》笔记：变量对象与预解析

执行流在执行环境中的执行过程（执行环境的生命周期）：  

* 建立arguments对象。检查当前上下文中的参数，建立该对象下的属性与属性值。  
  
* 检查当前上下文的函数声明，也就是使用function关键字声明的函数。在变量对象中以函数名建立一个属性，属性值为指向该函数所在内存地址的引用。`如果函数名的属性已经存在，那么该属性将会被新的引用所覆盖`。  
  
* 检查当前上下文中的变量声明，每找到一个变量声明，就在变量对象中以变量名建立一个属性，属性值为undefined。`如果该变量名的属性已经存在，为了防止同名的函数被修改为undefined，则会直接跳过，原属性值不会被修改`。   
  
总之：`function声明会比var声明优先级更高一点 ; 在函数体内，函数的参数a的优先级高于变量a`     

下面通过具体的例子来看变量对象：
```
function test(){
   console.log(a);
   console.log(foo());
   
   var a= 1;
   function foo(){
      return 2;
   }
}
test();
```
当执行到text()时会生成执行环境textEC，具体形式如下：
```
// 创建过程
textEC = {
 VO: {},             // 变量对象（variable object）
 scopeChain: [],    // 作用域链
 this: {}          // this指向
}
```
仅针对变量对象来具体展开：
```
VO = {
   argument: {},             // 传参对象
   foo: "<foo reference>",  // 在testEC中定义的function
   a: undefined            // 在textEC中定义的var
}
```
`未进入执行阶段之前，变量对象中的属性都不能访问！但是进入执行阶段之后，变量对象转变为了活动对象，里面的属性都能被访问了，然后开始进行执行阶段的操作`
```
VO --> AO   // 执行阶段 Active Object
AO = {
   argument: {...},
   foo: function(){return 2},
   a: 1
}
```
最后我们将变量对象创建时的VO和执行阶段的AO整合到一起就可以得到整个执行环境中代码的执行顺序：
```
function text(){
   function foo(){
     return 2;
   }
   var a;
   console.log(a);      // undefined
   console.log(foo());   // 2
   a = 1;
}
text();
```
这个就是分步变量对象的创建和执行阶段来解读`预解析` 

#### 预解析
预解析分为 变量提升 和 函数提升    
`变量提升`：提升的是当前变量声明，赋值还保留在原来的位置  
`函数提升`：函数声明，可以认为是把整个函数体声明了   
函数表达式方式定义函数相当于变量声明 var fn = function(){}   

一个简单的变量提升的例子
```
!function(){
  console.log(a);
  var a = 1;
}()

// 实际执行顺序
!function(){
  var a;
  console.log(a);
  a = 1;
}()
```
技巧：
将执行过程手动拆分成两个步骤   
1.创建阶段：也称作编译阶段，主要是变量的声明和函数的定义（找var和function）  
2.执行阶段：变量赋值和函数执行   
   
这样看预解析还是比较容易的，但是涉及到命名冲突时，又是怎样的情况呢？

（1）一个函数和一个变量出现同名
情况一：变量只声明了，但没有赋值
```
!function(){
   var f;
   function f() {};
   console.log(f);  //f(){}
}()
```
情况一：变量只声明了且赋值
```
!function(){
   console.log(f);   // f(){} 
   var f = 123;
   function f(){};
   console.log(f);   // 123
}
```
 ` 结论：`   
     1.一个函数和一个变量出现同名，如果是变量只声明了，但没有赋值，变量名会被忽略   
     2.一个函数和一个变量出现同名，变量声明且赋值，赋值前值为函数，赋值后为变量的值   
     
（2）两个变量名出现同名   
```
!function(){
   console.log(f);  // undefined
   var f = 123;
   var f = 456;
   console.log()   // 456
}()
```
`结论：`
两个变量出现同名，重复的var声明无效，会被忽略，只会起到赋值的作用，前面赋值会被后面的覆盖   

（3）两个函数出现同名   
```
!function(){
   console.log(f);             //f(){return 456}
   function f(){return 123};
   function f(){return 456};
   console.log(f);             //f(){return 456}
}
```
`结论：`  
两个函数出现同名，由于javascript中函数没有重载，前面的同名函数会被后面的覆盖   

（4）变量名与参数名相同  
函数参数的本质是什么？函数参数也是变量，相当于在该函数的执行环境内最顶部声明了实参

情况一：参数为变量，与变量命名冲突  
```
(function (a) {
    console.log(a);  //100
    var a = 10;
    console.log(a);  //10
})(100);
// 相当于
(funcion (a) {
    var a = 100; 
    var a;          // 重复声明的var没有意义，忽略
    console.log(a); //100
    a = 10;
    console.log(a); //10
})(100);
```
情况二：参数为函数，与变量命名冲突
```
(function (a) {
    console.log(a);
    var a = 10;
    console.log(a)
})(function(){return 2});
// 相当于
(function (a){
   var a = function(){return2};
      var a;   // 重复声明的var没有意义，忽略
      console.log(a);  // function(){return 2}
      a = 10;
      console.log(a); // 10
})(function(){return 2});
```
情况三：参数为空，与变量命名冲突
```
(function (a) {
    console.log(a); // undefined
    var a = 10;
    console.log(a); // 10
})();
// 相当于
(function (a) {
  var a;
    console.log(a); // undefined
    a = 10;
    console.log(a); // 10
})();
```
### 《javascript高级程序设计》笔记：原型图解   
1. 图解原型链  
1.1 “铁三角关系”（重点）  
```
  function Person() {};  
  var p = new Person();  
```
![](https://sfault-image.b0.upaiyun.com/215/344/2153448555-5a00332c7c2df_articlex)     
该图描述了构造函数，实例对象和原型三者之间的关系，是原型链的基础：       
（1）实例对象由构造函数new产生；      
（2）`构造函数的原型属性`与`实例对象的原型对象`均指向原型；  
（3）原型对象中有一个属性constructor指向对应的构造函数；   
  





### 深入理解JavaScript系列（2）：揭秘命名函数表达式
函数声明：  function 函数名称 (参数：可选){ 函数体 } <br>
函数表达式：function `函数名称（可选）`(参数：可选){ 函数体 }<br>
function foo(){} // 声明，因为它是程序的一部分<br>
var bar = function foo(){}; // 表达式，因为它是赋值表达式的一部分<br>
new function bar(){}; // 表达式，因为它是new表达式<br>
(function(){  <br>
    function bar(){} // 声明，因为它是函数体的一部分<br>
  })();    <br>
  表达式和声明存在着十分微妙的差别，首先，函数声明会在任何表达式被解析和求值之前先被解析和求值，即使你的声明在代码的最后一行，它也会在同作用域内第一个表达式之前被解析/求值，参考如下例子，函数fn是在alert之后声明的，但是在alert执行的时候，fn已经有定义了
  
### 深入理解JavaScript系列（3）：全面解析Module模式
匿名闭包<br>
匿名闭包是让一切成为可能的基础，而这也是JavaScript最好的特性，我们来创建一个最简单的闭包函数，函数内部的代码一直存在于闭包内，在整个运行周期内，该闭包都保证了内部的代码处于私有状态。<br>
(function () {<br>
    // ... 所有的变量和function都在这里声明，并且作用域也只能在这个匿名闭包里<br>
    // ...但是这里的代码依然可以访问外部全局的对象<br>
}());<br>
(function () {/* 内部代码 */})();   
有括号，就是创建一个函数表达式，也就是`自执行`   




