# Python学习

## Python基础

### 数据类型

```python
print(r'\t\t\t') #在字符串前加r表示字符串中的'\'不表示转义
# \t\t\t
# 在Python中空值由None表示
```

### 变量

Python是一种动态语言,在赋值前不需要指定类型

1. 命名规则:数字,字母,下划线.不能以数字开头

2. 包名和变量名小写,变量名分割符可以用_,模块(函数)名用\_分割,常量名用大写


### 字符串

1. Python3中的字符串是以Unicode编码的

   ```python
   ord('A') # 65 输出字符的Unicode码
   chr('65') # A 将编码转化为对应的字符
   
   # 在Python中对byte字节的数据要在前面加一个'b'
   # b'123'占用的大小为3个字节
   # 通过str.encode方法可以将字符指定为bytes
   '123'.encode('ascii') #b'123'
   '中文'.encode('utf-8') #b'\xe4\xb8\xad\xe6\x96\x87'
   # 通过str.decode方法可以将bytes转为对应的字符
   
   print("他的名字是:%s..%s\n" % ("邱瑞翔","邱瑞翔2号")) # 在Python中,用%代替,分割格式化与输入内容
   ```

### 列表list

```python
x_list = ['aaa', 'bbb', 'ccc']
```

1. 列表支持下标访问

   ```python
   list[0] # 列表的第一个元素
   list[-1] # 列表的最后一个元素
   ```

2. 列表中的元素可以不是一个数据类型

   ```python
   x_list = ['aaa', 111, True, ['aaa', 'bbb']]
   ```

3. 列表的常用方法

   ```Python
   # 追加
   list.append('aa')
   # 插入
   list.insert(1,'bbb') # 将bbb插入到列表下标为1的位置
   # 删除
   list.pop() # 删除列表末尾的元素
   # 替换元素可以直接使用下标进行替换
   ```

### 元组tuple

```python
x_tuple = ('aa', 'bb', 'cc')
```

1. 元组和列表类似,但是在定义的时候必须初始化,且某个元组一经初始化就不能再被更改

   ```python
   x_tuple = () # 定义一个空的元组
   x_tuple = (1, 2)
   x_tuple = (1,) # 注意当定义只有一个元素的元组时,要和普通变量的赋值区分开来
   ```

2. 元组不可变指的是元组中的元素的指向不可以发生改变,但元组中某个元素的值可以发生改变

   ```python
   x_tuple = (1, 2, ['aa', 'bb'])
   x_tuple[2][0] = 'bbb' # 是符合语法的	
   ```

### 字典(dict)

```python
x_dict = {'Jack':100, 'Tom':90, 'Jery':98}
```

1. 常用方法

   ```python
   my_dict = {'Tom': 1, "Mary": 2}
   my_dict1 = {'Tom' :4}
   # 添加/修改
   my_dict[key] = value  # dict中有key就修改对应的value值,否则添加一个新的键值对
   # 删除
   del my_dict[key] # 删除dict中键为key的键值对
   # 更新
   my_dict1.update(my_dict) # key相同的为修改,key不同的为添加
   # setdefault()
   my_dict.setdefault(key,value) #有key则无效,无key则将默认值value添加
   # 得到值
   my_dict.get(key)
   # 得到键/值的列表
   key_list,value_list = my_dict.keys(),my_dict.values()
   # 删除
   val = my_dict.pop(key) # 删除key对应的键值对,返回key对应的value
   my_dict.popitem() # 删除最后一个键值对
   ```

### 集合(set)

```python
x_set = set([1, 2, 3]) # 集合中不能有重复的元素
```

1. 常用方法

   ```python
   x_set = set([1, 2, 3])
   # 添加
   x_set.add('a') # 在向set中添加元素时一定保证加入的元素是可以哈希的
   x_set.update([1, 2, 3]) # 会将列表中的元素拆分成单个的字符添加到set中
   # 删除
   x_set.remove(value) # 将value删除,如果value不存在会报错
   x_set.discard(value) # 将value删除,如果value不存在不会报错
   # 查找
   value in/not in x_set
   # set也可以作交集,并集,补集的运算
   x_set & x_set1
   x_set | x_set1
   
   ```

### 条件判断

```python
if 条件1:
    执行语句1
elif 条件2:
    执行语句2
...
else:
    执行语句n
```

1. 输入函数

   ```python
   s = input("输入年份:")
   year = int(s)
   if year > 2022:
       print("未来")
   elif year == 2022:
       print("现在")
   else:
       print("过去")
   ```

### 循环

#### for循环

1. ```python
   for i in range(11):
       print(i)
   ```

2. ```python
   x_list = [1, 2, 3, 4, 5]
   for i in x_list:
       print(i)
   ```

#### while循环

```python
n = 100
while n > 0:
    print(n)
    n = n - 1
```

### 函数

```python
def 函数名:
	函数内容
	return...	
# python中的函数的返回值可以有多个结果


# 在python中,有pass关键字可以作为占位符,表示目前还没有决定此处的逻辑
def func():
    # 占位符
    pass

# python中支持默认参数,n默认为2
def func1(a, n = 2):
    ans = 1
    while(n > 0):
        ans = ans * a
        n = n - 1
    return ans


if __name__ = '__main__':
    print(func(2))  # 4
    print(func(2, 3)) # 8
    
# 在python中函数的返回值如果是多个的话,返回的是一个元祖(tuple)

# 在使用默认参数是要注意默认的值在定义的时候被赋值并分配内存空间,之后每次调用会改变默认参数的值
# 因此最好将默认参数设置为不可变对象None,str之类的
def func2(l = []):
    l.append('end')
    return l
def func3(l = None):
    if l is None:
        l = []
     l.append('end')
     return l
if __name__ = '__main__':
    print(func2())  # ['end']
    print(func2())  # ['end', 'end']
    print(func3())  # ['end']
    print(func3())  # ['end']
    
 # python中还允许有可变参数:*variable
def func4(*var):
    sum = 0
    for i in var:
        sum = sum + i
    return sum
if __name__ = '__main__':
    print(func4(1,2,3)) # 6
    l = [1, 2, 3]
    print(func4(*l)) # 6
```



### 高级特性

#### 切片

#### 列表生成式

```python
l = [x * x for x in range(1,11)] # 快速生成1,11的平方组成的列表,甚至还可以在for循环之后添加if判断语句
print([m+n for m in 'abc' for n in 'abc']) # 一行生成全排列,这个太秀了!
```

#### 生成器

```python
g = x * x for x in range(1,11) # 得到一个生成器generator
# 生成器可以通过for循环来得到生成器中的元素
for i in g:
    print(i)
```

