# JavaScript

### JS基础语法

1. 定义:JavaScript是一个脚本语言,通过解释器运行.主要在客户端(浏览器)上运行,也可以基于node.js的服务器上运行.

2. JavaScript的运行过程:首先代码被存储到本地硬盘上;双击html文件就会读取该文件,将文件内容读取到内存中;浏览器会解析文件中的代码,并转换为二进制形式的机器指令;机器指令会被CPU识别并执行.

3. JavaScript的组成

   + ECMAScript:JavaScript语法
   + DOM:页面文档对象模型,对页面中的元素进行操作
   + BOM:浏览器对象模型:对浏览器窗口进行操作.

4. JavaScript的格式

   ```javascript
   	<script>
       	//script标签内写JavaScript的内容.
           //输入框
           promt("xxxx:");
   		//警告框		
   		alert("xxxx");
   		//输出语句,在控制台打印一个日志
   		console.log("xxx");
   		
       </script>
   ```

5. JavaScript的语法

   1. JavaScript中的变量类型是动态类型

      ```html
      <script>
      	var a = 10;
          a = "hello";
      </script>
      ```

   2. JavaScript中有一个特殊的类型undefined,表示当前的变量没有被初始化.

      ```html
      <script>
      	var a;
      </script>
      ```

   3. JavaScript中的数组

      ```html
      <script>
      	var arr1 = new Array();
          var arr2 = [];
          //JS中的数组中的元素类型是不限制的.
          var arr3 = [1,2,4,5,"hello"];
          //向数组中添加元素
          arr3.push(val);
          //删除数组中的某个元素
          arr3.splice(index,length);
          
      </script>
      ```

   4. JavaScript中的函数

      ```html
      <script>
          function 函数名(形参列表){
              
          }
          //在JS中函数是头等公民,可以将一个函数赋给一个变量
          var a = function(){
              
          };
      </script>
      ```

   5. JavaScript中的对象

      ```html
      <script>
      	var a = {}; //创建了一个空的对象
          //JS中的对象中的属性是 以键值对的形式描述的,以逗号隔开.
          var student = {
              name:"邱瑞翔",
              age:21,
              birthday:"2002-03-13",
              eat:function(){
                  console.log("~~");
              }
          };
          //使用构造函数来构造对象 !JS中没有java中继承,封装,多台那一套
          //JS和java其实毫无关联
          function Stu(name, age, birthday){
              this.name = name;
              this.age = age;
              this.birthday = birthday;
          }
          let stu1 = new Stu("邱瑞翔",21,"2002-03-13");
          console.log(stu1.name);
      </script>
      ```

### JS-WebAPI

JS的WebAPI包括了DOM API 和 BOM APT两部分.

#### DOM API

1. DOM全称为Document Object Model

2. 一个页面的结构其实是一种树形结构,被称为DOM 树.在DOM树中,一个页面就是一个文档,使用document表示;页面中的所有标签都被称为元素,使用element表示;页面中所有内容可以被称为节点,用node表示.

3. 常见API

   + 获取元素

     ```html
     <script>
     	//querySelector:获取一个元素.
         //selectors是一个或多个要匹配的选择器的字符串形式,这里的选择器和CSS中的选择器的格式一样.
         //如class 就是 ".className";id 就是 "#idName"; 标签 就是 "标签名"
         var element = document.querySelector(selectors);
         //querySelectorAll:获取html中的所有元素并按照顺序返回一个元素列表
         let elements = document.querySelectorAll("selectors");
         
     </script>
     ```

   + 事件

     + 定义:用户在页面上进行的一系列操作(点击,选择,修改,拖动)会产生一个一个的事件,这些事件会被JS获取到,从而针对这些事件产生响应.
     + 事件三要素:
     + 1. 事件源:有哪个元素触发的(选择器)
       2. 事件类型:是点击,选中,修改还是拖动等产生的事件?
       3. 事件处理程序:根据事件产生的响应.

   + 操作元素

     1. 获取/修改元素内容

        ```html
        <button>
            点击
        </button>
        <div class = "parent">
            <div class = "child">
                修改元素之前
            </div>
        </div>
        
        <script>
            //element.innerText:获取元素内的文本内容,无法区分元素内的标签结构.
        	var button = document.querySelector("button");
            console.log(button.innerText);
            button.innerText = "点击之后";
            console.log(button.innerText);
            //element.innerHTML:获取元素内的内容,可以区分元素内的标签结构
         	var parent = documetn.querySelector(".parent");
            console.log(parent.innerHTML);
            //修改成这个情况也会导致页面内的内容发生改变.
            parent.innerHTML = "<div class = 'child'>修改元素之前</div>";
        </script>
        ```

     2. 获取/修改表单元素内容

        表单元素(主要指input标签)下的属性都可以被获取或修改

        + value:值
        + disabled:禁用
        + checkeded:复选框的值
        + selected:下拉框的值
        + type:input的类型

        1. 修改value的值

           ```html
             <input type="button" value = "播放">
           
               <script>
                 let input = document.querySelector("input");
                 input.onclick = function(){
                   if(input.value === "播放"){
                   input.value = "暂停";
                   }else{
                   input.value = "播放";
                   }
                 }
               </script>
           ```

        2. 点击计数

           ```html
            <input type="text" value = '0' id = 'text'>
              <input type="button" value = '加1' id = 'button'>
           
              <script>
                  let text = document.querySelector("#text");
                  let button = document.querySelector("#button");
                  button.onclick = function(){
                      //获取到的num本来为字符串,前面加一个"+"转换层数值
                      let num = +text.value;
                      num = num + 1;
                      console.log(num);
                      text.value = num;
                  }
           
              </script>
           ```

     3. 获取/修改样式元素内容

        1. 修改行内样式元素:控制字体大小.

           ```html
           <div style="font-size:20px;">
                 邱瑞翔
             </div>
             <button class = 'increase'>+</button>
             <button class = 'decrease'>-</button>
           
             <script>
                 let div = document.querySelector("div");
                 let increase = document.querySelector(".increase");
                 let decrease = document.querySelector(".decrease");
                 increase.onclick = function(){
                     let size = parseInt(div.style.fontSize);
                     size += 5;
                     console.log(size);
                     div.style.fontSize = size + "px";
                 }
                 decrease.onclick = function(){
                     let size = parseInt(div.style.fontSize);
                     size-=5;
                     console.log(size);
                     div.style.fontSize = size + "px";
                 }
             </script>
           ```

        2. 修改类名

           ```html
           切换背景颜色
           
             <style>
                   *{
                       margin: 0;
                       padding:0;
                       box-sizing: border-box;
                   }
                   html,
                   body{
                       height:100%;
                       width: 100%;
                   }
                   .container{
                       height: 100%;
                       width: 100%;
                   }
                   .light{
                       background-color: white;
                       color:black;
                   }
                   .dark{
                       background-color:black;
                       color:white;
                   }
                   
               </style>
               <div class="container light">
                   <div class="row">一</div>
                   <div class="row">二</div>
                   <div class="row">三</div>
                   <div class="row">四</div>
               </div>
           
               <script>
                   let div = document.querySelector("div");
                   div.onclick = function(){
                   let name = div.className;
                   if(name.indexOf("light") != -1){
                       div.className = "container dark";
                   }else{
                       div.className = "container light";
                   }
               }
               </script>
           ```

     4. 操作节点

        1. 创建节点

           ```html
           <script>
                 let div = document.createElement("div");
                 div.id = "first";
                 div.className = "q";
                 div.innerHTML = "呵呵";
             </script>
           ```

        2. 添加节点

           ```html
           <div class="container">
           
             </div>
             <script>
                 let div = document.createElement("div");
                 div.id = "first";
                 div.className = "q";
                 div.innerHTML = "呵呵";
                 let container = document.querySelector(".container");
                 container.appendChild(div);
             </script>
           ```

        3. 插入节点

           ```html
           <div class="container">
                   <div>11</div>
                   <div>22</div>
                   <div>33</div>
                   <div>44</div>
             </div>
             <script>
                 let newDiv = document.createElement("div");
                 newDiv.id = "first";
                 newDiv.className = "q";
                 newDiv.innerHTML = "呵呵";
                 let container = document.querySelector(".container");
                 //将newDiv节点插入到原先的第二个节点之前(也就是33)
                 container.insertBefore(newDiv,container.children[2]);
             </script>
           ```

        4. 删除节点

           ```html
           let oldChild = container.removeChild(container.children[0]);
           ```




