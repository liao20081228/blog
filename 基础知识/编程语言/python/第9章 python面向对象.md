---
title: 第9章 python面向对象
tags: python
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.5*

------

# 1 类和对象
## 1.1 python中面向对象
* 类——具有相似属性和方法的对象的抽象。
* 对象——由其类定义的数据结构的唯一实例。对象由数据成员和方法组成。
	* 类对象：类定义完成后，会在当前作用域中定义一个以类名为名字，指向类对象的名字。支持操作：
		* 实例化：使用`instance_name = class_name()`的方式实例化，实例化操作创建该类的实例。
		* 属性引用：使用`class_name.class_attr_name`的方式引用类属性。
	* 实例对象：实例对象是类对象实例化的产物，实例对象仅支持一个操作：
		* 属性引用；与类对象属性引用的方式相同，使用`instance_name.attr_name`的方式。
* 实例——某个类的单个对象。 例如，对象obj属于Circle类，它是Circle类的实例。
* 实例化——创建类的实例。
* 数据成员（属性）——保存与类及其对象相关联的数据的类变量或实例变量。
	* 类变量（类属性）——由类的所有实例共享的变量。类变量在类中定义，但在类的任何方法之外。**可以通过类对象和实例对象调用**。
	* 实例变量（实例属性）——在构造方法中定义并仅属于类的实例的变量。**只能通过实例对象调用**。
* 方法——在类定义中定义的方法。
	* 普通方法 
		* 和普通函数一样
		* 可以访问类属性，不能访问实例属性
		* 通过类对象调用
		* 通过实例调用时会转化为实例方法，要求第一个参数作为实例本身。
	* 静态方法：
		* 以`@staticmethod`修饰，其他和普通函数类似
		* 通过类对象和实例对象调用
		* 可以访问类属性，不能访问实例属性
	* 类方法：
		* 以`@classmethod`修饰，且方法的第一个参数是`cls`，`cls`表示类对象
		* 第一个参数参数为`cls`
		* 通过类对象和实例对象调用
		* 可以访问类属性，不能访问实例属性
	* 实例方法：
		* 和普通函数类似
		* 方法的第一个参数是`self`，`self`表示实例对象。
		* 通过实例对象调用
		* 通过类调用时需要传入一个实例对象
* 继承——将类的特征传递给从其派生的其他类。
* 函数重载——将多个行为分配给特定函数。 执行的操作因涉及的对象或参数的类型而异。
* 运算符重载——将多个函数分配给特定的运算符。
* 方法重写——如果从父类继承的方法不能满足子类的需求，可以对其进行改写，这个过程叫方法的覆盖（override），也称为方法的重写。
* 局部变量——定义在方法中的变量，只作用于当前实例的类。

## 1.2 类的创建
```python
#语法格式：
class ClassName:
   'Optional class documentation string'
   class_attr=value #类属性，可以通过类对象或实例对象访问
   ... ...
   def fun1(arg1,arg2，...):  #普通方法，只能通过类对象调用，通过实例调用时转为实例方法
        ... ...
   @staticmethod 
   def fun1(arg1,arg2，...):  #静态方法，可以通过类对象或实例对象调用
        ... ...
   @classmethod 
   def fun1(cls，arg1,arg2，...):  #类方法，第一个参数cls，可以通过类对象或实例对象调用
        ... ...
   def function(self,arg1,arg2，...): #实例方法，第一个参数self，只能通过实例访问，通过类对象访问时要传入一个实例对象
        self.instance_attr=value #实例属性，只能通过实例访问
        ... ... 
   ... ...
```
* class语句定义一个新的类。 类的名称紧跟在class关键字之后，在类的名称之后紧跟冒号
* 类体的第一行可以有一个类的说明字符串，可以通过`ClassName.__doc__`访问。
* **属性**（数据成员）
	* 类属性：为类对象和所有实例对象共享。
		* 创建：
			* 在类中，但在类的任何函数和方法外，使用`class_attr=value`静态创建
			* 在类外或类的方法中，只能使用`classname.class_attr=value`动态创建，不能使用	`InstanceName.class_attr=value`
		* 读取： 
			* 在类的方法中，`ClassName.class_attr`，`self.class_attr`，`cls.class_attr`
			* 在类外，`ClassName.class_attr`和`InstanceName.class_attr`
		* 修改：
			* 在类的方法中，使用`ClassName.class_attr=value`，`cls.class_attr=value`修改
			* 在类外，只能使用`classname.class_attr=value`，不能使用	`InstanceName.class_attr=value`创建
		* 删除：
			* 类中和类外，`del ClassName.class_attr`，不能使用`del InstanceName.class_attr` 
	 * 实例属性——为某个具体的实例对象所有，无论类中还是类外：
		 * 通过`self.instance_attr=value`创建
		 * 只能通过`instance_name.class_attr`进行增删读改。
* **方法**
	* 普通方法 
		* 和普通函数一样
		* 通过类对象调用
		* 通过实例调用时会转化为实例方法，要求第一个参数作为实例本身。
		* 可以访问类属性，不能访问实例属性
	* 静态方法：
		* 以`@staticmethod`修饰，其他和普通函数类似
		* 通过类对象和实例对象调用
		* 可以访问类属性，不能访问实例属性
	* 类方法：
		* 以`@classmethod`修饰，且方法的第一个参数是`cls`，`cls`表示类对象
		* 第一个参数为类对象，一般是`cls`
		* 通过类对象和实例对象调用
		* 可以访问类属性，不能访问实例属性
	* 实例方法：
		* 和普通函数类似
		* 方法的第一个参数是实例对象，一般是`self`
		* 通过实例对象调用时
		* 通过类调用时需要传入一个实例对象
			* 初始化函数——`__init__()`
				* 对象创建时调用
				* 可以重载
				* 可以接受参数
				* 只能自动调用，不可主动调用
				* 当没有显式定义构造函数时，Python会自动创建一个不执行任何操作的无参默认构造函数。
			* 析构函数——`__del__()`
				* 对象销毁时调用
				* 可以重载
				* 可以接受参数
				* 主要用于释放堆空间
				* 只能自动调用，不可主动调用
				* 当没有显式定义析构函数时，Python会自动创建一个不执行任何操作的无参默认析构函数。
* **使用实例访问方法和属性时，同名的实例方法和实例属性优先于同名的类属性和类方法**
* 访问权限
	* 不以双下划线开头的属性和方法是公有的，类外可以访问
	* 以双下划线开头，但不以双下划线结尾的属性和方法是私有的，类外不可访问
	* 以双下划线开头和结尾的属性和方法是特殊专用的，类外可访问

## 1.3 实例对象创建与销毁
* 创建实例对象：使用类名调用该类的构造函数来创建对象，并传递其 `__init__`函数接受的参数。语法格式：`instance_name=classname([构造参数])`
* 销毁实例对象
	* 销毁对象时会调用析构函数`__del__()`
	* Python自动定期回收不需要的对象(内置类型或对象)以释放内存空间，叫做**垃圾收集**。 
	* Python的垃圾收集器在程序执行期间运行，**当对象的引用计数达到零时触发**。
	* 对象的引用计数随着指向它的别名数量而变化。
		* 当对象的引用被分配一个新名称或放置在容器(列表，元组或字典)中时，引用计数会增加。   
		* 当用del删除对象的引用时，引用被重新分配，或者其引用超出范围时，引用计数会减少 。



## 1.4 访问对象的属性与方法

* 可以使用带有对象的点(.)运算符来访问对象的属性和方法。语法如下：
```python
object_name.attr
object_name.method([参数])
```
还可以使用内建函数来访问对象的属性和方法
* getattr(obj，name [，default]) ——访问对象的属性。这里参数name是字符串
* hasattr(obj，name) ——检查属性是否存在。
* setattr(obj，name，value) ——设置一个属性。如果属性不存在，那么它将被创建。
* delattr(obj，name) —— 删除一个属性。



# 2 类的内建属性和方法（未完待续）

|内建属性|描述|备注
|--|--|--|
|`__class__`|对象类别。类对象返回type，实例对象返回`模块名.类名`，如果在主模块则模块名为`__main__`
|`__name__`|类名
|`__module__`| 定义类的模块名称。如果在主模块则模块名为`__main__`
|`__dict__`|包含该类的命名空间的字典。
| `__doc__ `| 类文档字符串或无，如果未定义。
|`__base__`|基类
|`__bases__`|所有基类的元组


|内置方法|描述 
|--|--|
|`__getattribute__(self，name)`|访问对象的属性。default是 默认返回值，如果不提供该参数，在没有对应属性时，将触发 AttributeError。`object.attr`调用
|`__setattr__(self，name，value)`| 设置一个属性。如果属性不存在，那么它将被创建。`object.attr=value`调用
|`__delattr__(self，name)`|删除一个属性。`del object.attr`调用|
|`__format__(self, format_spec)`|由`formate()`和`str.format()`调用
|`__hash__()`|
|`__reduce__`|
|`__reduce_ex__`|
|`__repr__(self)`|由`repr()`调用
|`__sizeof__`|调用此函数创建类cls的新实例
|`__str__(self)`|由`print()`调用
|`__dir__(self)`|由`dir()`调用。返回该对象的合法属性列表。
|`__subclasshook__`|
|`__weakref__`|
|`__init__(self[,...])`|在实例创建之后(`__new__()`)调用，但在返回给调用者之前调用。
|`__new__(cls[,...])`|调用此函数创建类cls的新实例
|`__del__(self)`|在实例即将被销毁时调用
|`__le__(self.other)`|`x<=y`调用`x.__le__(y)`|
|`__ge__(self.other)`|`x>=y`调用`x.__ge__(y)`|
|`__eq__(self.other)`|`x==y`调用`x.__eq__(y)`|
|`__ne__(self.other)`|`x!=y`调用`x.__ne__(y)`|
|`__lt__(self.other)`|`x<y`调用`x.__lt__(y)`|
|`__gt__(self.other)`|`x>y`调用`x.__gt__(y)`|





 
# 3 继承
继承的语法格式：
```
class calss_name(base1,base2....):
    ....
```

* 子类可以继承父类的属性和方法
* 私有方法和属性不能继承
* 支持多继承和单继承
* 支持方法重写。子类可以改写父类的方法。
* 如果派生类的构造方法没有重写，则基类的构造方法（`__init__()`方法）会被自动调用；否则，它需要在其派生类的构造方法中专门调用基类的构造方法，`super(派生类,self).__init__(参数)或 基类.__init__(self,参数)`。
* 在调用基类的方法时，需要加上基类的类名的前缀。区别在于类中调用普通函数时并不需要带上self参数。新式类中使用`super(基类,self).method(参数)或 基类.method(self,参数)`。
* Python总是首先查找对应类型的方法，如果它不能在派生类中找到对应的方法，它才开始到基类中逐个查找。
* 可以使用issubclass()或isinstance()函数来检查两个类和实例之间的关系。
	* issubclass(sub，sup)，布尔函数如果给定的子类sub确实是超类sup的子类返回True。
	* isinstance(obj，Class)，布尔函数如果obj是类Class的一个实例，或者是类的一个子类的实例则返回True。

# 4 重载

**重载**：函数名相同，函数的参数列表不同(包括参数个数和参数类型)，至于返回类型可同可不同。
* 类外：
	*  在类外可以根据需要对函数重载。
* 类中：
	* 在类中可以根据需要对方法重载（实际就是重写方法，因为python中所有的类都继承自object类）。
	* 在类中可以根据需要对运算符重载。

|运算符|表达式|内置方法|
|--|--|--|
|加法|p1 + p2|`__add__`|
|减法|p1 - p2|`__sub__`|


乘法
p1 * p2
p1.mul(p2)


次幂
p1 ** p2
p1.__pow__(p2)


除法
p1 / p2
p1.__truediv__(p2)


地板除法
p1 // p2
p1.__floordiv__(p2)


模 (modulo)
p1 % p2
p1.__mod__(p2)


按位左移
p1 << p2
p1.__lshift__(p2)


按位右移
p1 >> p2
p1.__rshift__(p2)


按位AND
p1 & p
p1.__and__(p2)


按位OR
p1 or p2
p1.__or__(p2)


按位XOR
p1 ^ p2
p1.__xor__(p2)


按位NOT
`~p1
p1.invert() `原文出自【易百教程】，商业转载请联系作者获得授权，非商业请保留原文链接：https://www.yiibai.com/python/operator-overloading.html




# 5 经典类和新式类的区别

* 定义方式不同
	* 在Python 2.x 版本中，默认类都是旧式类，除非显式继承object。
	* 在Python 3.x 版本中，默认类就是新式类，无需显示继承object。
* 保持class与type的统一。
	* 对旧式类的实例执行`a.__class__`返回`__main__.cc`,执行`type(a)`返回`<type 'instance'>`
	* 对新式类的实例执行`a.__class__`和`type(a)都返回`<class '__main__.aa'>
* 对于多重继承的属性搜索顺序不一样
	 * 新式类是采用广度优先搜索:先水平搜索，然后再向上移动
	 * 旧式类采用深度优先搜索: 先深入继承树左侧，再返回，开始找右侧

# 6 示例

```python
class person:
"这是一个基类" #文档字符串
	count=0 #类属性，所有类对象和实例对象共享，可被类对象和实例对象访问，但会被同名实例对象覆盖
	__count=0 #私有类属性，类外不可访问，不可继承
	def __init__(self,name,age):
		self.name=name  #实例属性，只属于某个具体实例对象，只有实例对象可以访问
		self.__age=age  #私有实例属性
		person.count+=1
		person.__count+=1
	
	#普通方法，可被类对象和实例对象调用，但使用实例对象调用时会转为实例方法，此时至少要有一个参数
	def nomal_method(): 
		print("nomal method,person.count：",person.count)
	
	#静态方法，可被类对象和实例对象调用，不可访问实例属性和方法
	@staticmethod  
	def static_method(): 
		print("static method,person.count：",person.count)
	
	#类方法，可被类对象和实例对象调用，不可访问实例属性和方法
	@classmethod
	def class_method(cls):
		print("class method,cls.count：",cls.count,"person.count：",person.count)
	
	#实例方法，只可被实例对象调用，类对象调用需要传入实例对象。可以访问类和实例的属性和方法
	def instance_method(self):
		print("instance method,self.count：",self.count,"person.count：",
		person.count,"self.name:",self.name,"self.__age:",self.__age)
	
	#私有方法，不可访问，不可继承
	def __private_method(self): 
		print("private method,self.count：",self.count,"person.count：",person.count,
		"self.name:",self.name,"self.__age:",self.__age)	


class adult(person):
	def __init__(self,age)
	



class child(person):





```




------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.5*

------
