# CSS

1. CSS有两种格式

   1. 嵌入在style标签中
   2. 写入到css文件里,然后在html文件的头部加上css文件的link

2. 选择器:选中页面的标签元素,然后指定元素内内容的样式.

   1. 基础选择器

      + 标签选择器:标签名{},作用范围为所有的该标签.
      + 类选择器: . 类名{} 需要在body正文中的标签名后加上class 辅助使用.可以使标签差异化.最常用的一种选择器.
      + id选择器: 和类选择器类似, #id名{},body中的变迁名后加上id
      + 通配符选择器:对body中所有内容加上的格式  *{}

   2. 复合选择器

      + 后代选择器 : 元素1 元素2{}  元素1是父级,元素2是子级,涵盖的范围是只包括元素2,不包括元素1.元素1和元素2 之间一定要有空格.

      + 子选择器 :不常用

      + 并类选择器:用逗号隔开.   元素1,元素2{}   推荐竖着写,最后一个元素后面没有逗号.

      + 伪类选择器

        > 1. 链接伪类选择器
        >    + a:link :选择未被访问过的链接 a:link{}
        >    + a:visited :选择已经被访问过的链接 a:visited{}
        >    + a:hover :选择鼠标指针悬停上的链接 a:hover{}
        >    + a:active :选择活动链接(鼠标按下了但是未弹起) a:active{}
        > 2. force伪类选择器
        >    + 选取获取焦点的input表单元素 元素1 input:focus{}

3. 字体样式

   ```html
   1.设置字体 
   font-family:"微软雅黑" //设置字体的格式
   font-size:20px //设置字体的大小
   font-weight:bold //设置字体的粗细,bold normal.. 还可以用数字表示粗细(100-900,只能是整数,大于700为加粗)
   font-style:italic //设置字体样式:倾斜(italic) 无样式(normal)
   2.文本属性
   color:red //给文本设置颜色,也可以用rgb 或者 rgba
   text-align:center //控制水平文本对齐:left,center,right
   text-decoration:underline //文本装饰:underline(下划线) none(无样式) overline(上划线) line-through(删除线)
   text-indent:2em //文本缩进:可以是em,也可是px. 1em自动对应浏览器中的1个字体.
   line-height:20px //控制行高.当行高与元素高度相等可是使元素垂直居中
   ```

4. 背景属性

```html
background-color:red //设置背景颜色,transparent指的是背景透明
background-image:url(图片路径) //设置背景图片.注意:背景图片和背景颜色不冲突.
//以下属性基本是都是基于背景图片的属性.
background-repate:no-repate //背景是否平铺,默认是平铺.平铺指的是当背景图片的尺寸比背景的原大小小时会沿x,y方向进行复制. repate-x,repate-y,no-repate
background-position:center center //设置背景图片的位置,第一个是x,第二个是y  center,left,right,top,bottom
background-size:200px 300px //设置背景图片的尺寸,第一个是宽度,第二个是高度.也可以填百分比(指占父元素的百分比)


```

5. 改变元素格式

   ```
   display:block //设置元素的格式 block:块级元素  inline:行内元素  通常是把行内元素设置成块级元素
   
   ```

6. 边框属性

   ```html
   weight:200px  //设置边框的宽度
   height:300px //设置边框的高度
   border-radius:20px //使边框变成圆角矩形,圆形就是让radius的尺寸变得更大.也可以展开写,border-left-bottom-radius,为左下角的边框
   border-style:solid //为边框线设置格式,solid为实线.
   border-width:10px //为边框线设置宽度
   border-color:red // 为边框设置颜色. 对于边框的属性而言,可以单独设置某一个边或者某一个角的属性
   //边框的长宽在设置后在浏览器中展示时的边框宽度和高度会变大.因为会向外撑大盒子.改变这种做法的方式:
   box-sizing:border-box;
   ```

7. 内边距:padding

   ```html
   padding:20px //设置内边距.也可以单独为一个方向设置内边距.内边距也会撑大盒子,用box-sizing设置格式可以解决
   ```

8. 外边距

   ```html
   margin:20px //同内边距类似
   margin:0 auto    <===> margin:auto 
   把水平方向设置成auto可以让块级元素水平居中，而text-align则是把行内元素和块级元素都水平居中.
   
   //在设置页面样式时,为了使页面在不同浏览器中展现的样式相同,我们需要提前去除掉浏览器的默认样式
   *{
   	margin:0;
   	padding:0;
   }
   
   ```

9. 弹性布局

   ```html
   默认情况下子元素是在父元素的左上角堆积的.
   通过弹性布局我们可以调整元素的展现形式.
   弹性布局是对父元素设置,其子元素会按照某种规则排列
   display:flex
   flex-flow:column //让子元素沿纵向排列,默认是水平排列.
   justify-content:center; //也可以是别的:space-around:水平等间距隔开.flex-start:在容器开头,flex-end:在容器结尾. space-between:在行与行之间留有间隔.(行的两端没有间隔)
   
   justify-content是让元素按水平方向排列. align-items是让元素按竖直方向排列.
   align-items和justify的属性相似,多了一个stretch,指的是如果子元素没有指定高度,则会充满父元素的高度.
   
   align-items:center //垂直居中,仅对单行元素有效
   item-contents:center //垂直居中,对多行元素有效.
   ```
