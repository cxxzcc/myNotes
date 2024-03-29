# conda

```shell

conda info 
conda config --show

conda env list 
conda config --get 查看已修改配置
conda config --add channels 通道地址
conda config --remove channels 通道地址



查看conda版本：conda --version
查看conda的环境配置：conda config --show
添加镜像：conda config --add channels <镜像>
更新conda：conda update conda

创建虚拟环境：conda create -n <环境名> python=<Python版本> 
            conda env create -f xx.yml
查看虚拟环境：conda info -e
激活虚拟环境：conda activate <环境名>
退出虚拟环境：conda deactivate
删除虚拟环境：conda remove --name <环境名> --all



# 查看当前环境下的所有包
conda list
# 安装numpy
conda install numpy  # 可以不添加版本，默认为最新版本
conda install numpy=1.24.3
# 更新某个包到最新版本
conda update numpy
# conda卸载包
conda uninstall numpy
# 清理缓存
conda clean -t

```
















# jupyter

[Users Manual](https://jupyter.brynmawr.edu/services/public/dblank/Jupyter%20Notebook%20Users%20Manual.ipynb)

A jupyter notebook document has the .ipynb extension and is composed of a number of cells. In cells, you can write program code, make marked and unmarked notes. These three types of cells correspond to: 
    1. code
    2. markdown
    3. raw

After filling the cell, you need to press *Shift + Enter*, this command will process the contents of the cell:   
interpret the code or lay out the marked-up text.


---

# NumPy

**NumPy** – Python library that allows to conveniently work with multidimensional arrays and matrices, containing mathematical functions. In addition, NumPy can vectorize many of the computations that take place in machine learning.

- [numpy](http://www.numpy.org/)
- [numpy tutorial](http://cs231n.github.io/python-numpy-tutorial/)
- [100 numpy exercises](http://www.labri.fr/perso/nrougier/teaching/numpy.100/)
- [numpy for matlab users](https://numpy.org/doc/stable/user/numpy-for-matlab-users.html)

# Matplotlib

- [matplotlib](http://matplotlib.org/)
- [matplotlib - 2D and 3D plotting in Python](http://nbviewer.jupyter.org/github/jrjohansson/scientific-python-lectures/blob/master/Lecture-4-Matplotlib.ipynb)
- [visualization in pandas](http://pandas.pydata.org/pandas-docs/stable/visualization.html)



# 基础
## 数据类型

* 数值
    * 整型
		* 整数都是int类型 大小无限制 长度过大，可用下划线作为分隔符 c=123_456_789
		* 二进制 0b开头 八进制 0o开头 十六进制 0x开头
		* 布尔值 True 1 False 0
    * 浮点型 小数都是float类型
    * 复数

* 字符串
	* 单引号双引号都可用
	* 三引号 多行字符串
	* \\ t 表示制表符  
	* \\ n 表示换行符
	* 格式化
	```python
	# %s 在字符串中表示任意字符  
	# %f 浮点数占位符  
	# %d 整数占位符  
	b = 'Hello %s'%'孙悟空'  
	b = 'hello %s 你好 %s'%('tom','孙悟空')  
	b = 'hello %3.5s'%'abcdefg' # %3.5s字符串的长度限制在3-5之间  
	b = 'hello %s'%123.456  
	b = 'hello %.2f'%123.456  
	b = 'hello %d'%123.95
	
	# 在字符串前加f来创建一个格式化字符串  
	# 在格式化字符串中可以直接嵌入变量  
	c = f'hello {a} {b}'
	
	name = '孙悟空'  
	# 拼串  
	print('欢迎 '+name+' 光临！')  
	# 多个参数  
	print('欢迎',name,'光临！')  
	# 占位符  
	print('欢迎 %s 光临！'%name)  
	# 格式化字符串  
	print(f'欢迎 {name} 光临！')
	
	
	# 字符串乘数字，重复指定的次数并返回  
	a = a * 20
	
	
	
	input('Please type something: ')
	print('hello', 'world', '!', sep='_', end='\n\n')  separator
	```

* 空值 None


## 类型转换

类型转换四个函数 int() float() str() bool()任何对象都可以转换为布尔值 none/0 -false 其余true

根据当前对象的值创建一个新对象


## 对象
结构
* id 唯一
	* id()函数查看
	* 由解析器生成的，在CPython中，id就是对象的内存地址
	* 创建后不能变
* type
	* 类型决定了对象有哪些功能
	* type()函数查看
	* Python是强类型语言，创建后类型不能修改
* value
	* 两类，可变对象 不可变对象


## 运算符

```python
# / 除，返回浮点类型  
# // 整除 
# ** 幂运算

比较运算符 除了下面两个其他的比较值 
#比较两个字符串的Unicode编码时，是逐位比较的, 可对字母顺序进行排序
# is 比较两个对象是否是同一个对象，比较的是对象的id  
# is not 比较两个对象是否不是同一个对象，比较的是对象的id

条件运算符（三元运算符）  
# 语法： 语句1 if 条件表达式 else 语句2
# true 语句1 false 语句2

逻辑运算符可以连着使用
```


## 流程控制语句
```
while 条件表达式 :
	代码块  
else :  
	代码块

true 循环 false 进else结束循环

for 变量 in 序列 :
	代码块
else可以用在for循环

pass 判断/循环中占位
```


## 序列
sequence
保存一组有序的数据
按照添加的顺序来分配索引

可变序列 
* 列表（list）
不可变序列
* 字符串（str）    
* 元组（tuple）



### 列表 list
```python
my_list = [] # 创建了一个空列表
my_list = [10,'hello',True,None,[1,2,3],print]# 列表中可以保存任意的对象

按序插入
len()取list长度
索引是负数，从后向前获取元素


切片  
# 切片指从现有列表中，获取一个子列表
列表[起始:结束]
包括起始位置, 不包括结束位置
列表[起始:结束:步长]
步长不能是0，可以是负数 后向前取

+可以将两个列表拼接为一个列表
* 可以将列表重复指定的次数

in 和 not in 存在/不存在
min()/max()/index()/count()

del stus[2] # 删除索引为2的元素

通过切片来修改删除元素



append()向后添加  
insert()  
extend()把序列添加到序列
clear()  
pop()  
remove()
reverse()反转
sort() 默认升序 reverse=True降序

range()生成自然数序列
range(0,10,2) start end 步长
for i in range(30):  
	print(i)
```



### 元组 tuple
不可变序列
操作基本和列表一致

```python
my_tuple = () # 创建了一个空元组
my_tuple = (1,2,3,4,5) # 创建了一个5个元素的元组
my_tuple = 10,20,30,40# 当元组不是空元组时，括号可以省略

# 元组的解包（解构）  
# 解包指就是将元组当中每一个元素都赋值给一个变量  
a,b,c,d = my_tuple
a , b = b , a #交换ab的值

# 在对一个元组进行解包时，变量的数量必须和元组中的元素的数量一致  
# 也可以在变量前边添加一个*，这样变量将会获取元组中所有剩余的元素  
a , b , *c = 'hello world'
```

### 字典 dict
```python
d = {} # 创建了一个空字典
{key:value,key:value,key:value}
#字典的键可以是任意的不可变对象 值可以是任意对象

# 使用 dict()函数来创建字典 key都是字符串
d = dict(name='孙悟空',age=18,gender='男')
# 也可以将一个包含有双值子序列的序列转换为字典  
# 双值序列，序列中只有两个值，[1,2] ('a',3) 'ab'  
# 子序列，如果序列中的元素也是序列，那么我们就称这个元素为子序列  
# [(1,2),(3,5)]  
d = dict([('name','孙悟饭'),('age',18)])

len() 获取字典中键值对的个数  
in 检查字典中是否包含指定的键
通过[]来获取值时，如果键不存在，会抛出异常 KeyError
get(key[, default]) 该方法用来根据键来获取字典中的值
setdefault(key[, default]) 可以用来向字典中添加key-value  
# 如果key已经存在于字典中，则返回key的值，不会对字典做任何操作  
# 如果key不存在，则向字典中添加这个key，并设置value

update([other])  
# 将其他的字典中的key-value添加到当前字典中
popitem()  
# 随机删除字典中的一个键值对，一般都会删除最后一个键值对 返回k v 元组
pop(key[, default])  
# 根据key删除字典中的key-value
clear()用来清空字典
copy()  
# 该方法用于对字典进行浅复制  
# 复制以后的对象，和原对象是独立，修改一个不会影响另一个  
# 注意，浅复制会简单复制对象内部的值，如果值也是一个可变对象，这个可变对象不会被复制

keys() 该方法会返回字典的所有的key
values()  
# 该方法会返回一个序列，序列中保存有字典的左右的值
items()  
# 该方法会返回字典中所有的项  
# 它会返回一个序列，序列中包含有双值子序列  
# 双值分别是，字典中的key和value
```

### 集合 set
集合中只能存储不可变对象
无序
不重复

```python
# 使用 {} 来创建集合  
s = {10,3,5,1,2,1,2,3,1,1,1,1}  
# 使用 set() 函数来创建集合  
s = set() # 空集合  
# 可以通过set()来将序列和字典转换为集合  
s = set([1,2,3,4,5,1,1,2,3,4,5])  
s = set('hello')  
s = set({'a':1,'b':2,'c':3}) # 使用set()将字典转换为集合时，只会包含字典中的键

in和not in来检查集合中的元素
len()来获取集合中元素的数量
add()
update() 将一个集合中的元素添加到当前集合中
pop() 随机删除并返回一个集合中的元素
remove()
clear()
copy()对集合进行浅复制

# & 交集运算  
result = s & s2 
# | 并集运算  
result = s | s2 
# - 差集  
result = s - s2 
# ^ 异或集 获取只在一个集合中出现的元素  
result = s ^ s2
# <= 检查一个集合是否是另一个集合的子集
result = a <= b
# < 检查一个集合是否是另一个集合的真子集
result = {1,2,3} < {1,2,3} # False  
result = {1,2,3} < {1,2,3,4,5} # True

# >= 检查一个集合是否是另一个的超集  
# > 检查一个集合是否是另一个的真超集
```



























