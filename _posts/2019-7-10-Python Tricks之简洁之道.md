---
layout: post
title: Python Tricks笔记
published: false
---

这是<<Python Tricks>>一书的读书笔记.什么是Python Tricks?就是一小段Python代码,它既传授了Python语言的一个特性,也是一个让你深入发掘这一特性的驱动力.

本文是该书第一章"写出更简洁的Python"的读书笔记.这里的简洁不止意味的代码量的少,还是指不需要加很多注释就可以让别人看懂,即可读性高.

## 善用assert

有的时候一个很有用的语言特性会得不到特别多的关注,Python的`assert`就是一个例子.它是一个dubug的帮手,可以测试一个条件表达式.如果表达式的结果为真,什么事都不会发生;如果表达式为假,程序会抛出一个`AssertionError`的异常.

### 一个assert的例子

来看一个计算商品打折后的价格的例子:

```python
def apply_discount(product, discount):
    price = int(product['price'] * (1.0 - discount))
    assert 0 < price < product['price']
    return price
```

这段代码中的`assert`保证了打折后的价格不会低于0,也不会高于商品的原来的价格.使用代码测试一下.

```python
shoes = {'name': 'Fancy Shoes', 'price': 14900}
apply_discount(shoe,0.25)
```

最后的输出是`11175`,符合预期.再故意使用一个错误的折扣来测试.

```python
apply_discount(shoe,2.0)
```

程序会报错:

```
Traceback (most recent call last):
  File "D:/WorkSpace/PythonTricks/ch2/assert.py", line 10, in <module>
    apply_discount(shoes, 2.0)
  File "D:/WorkSpace/PythonTricks/ch2/assert.py", line 3, in apply_discount
    assert 0 < price < product['price']
AssertionError
```

使用`assert`的好处是可以更快速的debug,而且从长远来看它会让你的代码更容易维护.

### 为什么不使用一般的异常

为什么不使用`if-else`并抛出一个一般的异常呢?因为`assert`告知开发人员出现了不可恢复的错误,而不是那些可以预估到的错误,如文件不存在这样的错误.

`assert`用于程序内部的自省,它断言一些条件是不可能的,如果这些条件有一个不满足,那就说明你的代码是有bug的,你就可以很快的通过报错信息来定位出错的代码.这条准则同样适用于其他语言.但记住,**断言错误永远不应该出现,除非你的程序有bug**.

### assert的语法

根据Python的文档,`assert`语法是这样的:

> assert_stmt ::= "assert" expression1 ["," expression2]

其中`expression`是一个条件表达式,`expression2`是可选的自定义错误信息.

Python解释器会把`assert`表达式转换成下面的代码:

```python
if __debug__:
	if not expression1:
		raise AssertionError(expression2)
```

可以看出,全局变量`__debug__`决定了`assert`是否起作用,一般情况下这个变量都是`True`的.

### assert的陷阱

1.不要把`assert`用于数据验证.因为`assert`可以通过命令行参数`-O`和`-OO`,或是环境变量`PYTHONOPTIMIZE`来禁用.如果你的代码使用`assert`来检查方法的参数是否包含错误的或不符合预期的参数,那么就会产生bug或安全漏洞.

举个例子,你写了一个删除商品的方法:

```python
def delete_product(prod_id, user):
	assert user.is_admin(), 'Must be admin'
	assert store.has_product(prod_id), 'Unknown product'
	store.get_product(prod_id).delete()
```

当`assert`禁用时,会出现两个问题:

- 任何用户都可以删除商品
- 黑客可以通过DDOS来攻击,导致服务器宕机

怎么解决这个问题,那就是不用`assert`来验证,而是用一般的`if`表达式并在必要的时候抛出验证错误.就像这样:

```python
def delete_product(product_id, user):
	if not user.is_admin():
		raise AuthError('Must be admin to delete')
	if not store.has_product(product_id):
		raise ValueError('Unknown product id')
	store.get_product(product_id).delete()
```

这样写的还有一个好处是,相比断言错误,这里抛出的自定义的异常在语义上更加正确.

2.永远不会失败的断言.如果你想要抛出`AssertError`的时候显示一些自定义信息,那么你可以在布尔表达式后面加一个字符串,但这种写法有一个陷阱,如果你不小心用括号把这两个参数包了起来,那么这个断言就永远是`True`的.例子:

```python
assert(1 == 2, 'This should fail')
```

原因是实际上这里的条件表达式变为了一个tuple,而tuple作为一个条件表达式永远是`True`的.这种写法在Python3中会导致一个语法告警,所以不再那么容易出现了.

## 充足的逗号

当你在dict,list和set常量中添加删除元素的时候,建议你在每一个行的末尾都加上逗号.

你可能一下子不明白我在说啥?看一个例子,假设你的代码中有下面这样的一个list:

```python
names = ['Alice', 'Bob', 'Dilbert']
```

如果你修改了其中一个元素,那么在版本控制系统中很难看出你改的内容,特别是当这个list的元素比较多的时候,因为版本控制系统如Git都是基于行的.

一种更好的代码风格是这样的:

```python
names = [
	'Alice',
	'Bob',
	'Dilbert'
]
```

每个元素一行,这样在版本控制系统中看代码的diff的时候很容易就知道加了新增,删除或修改了哪些元素.

但这样写还有一个问题,就是当你新增或删除元素的时候,需要手动的修改逗号的位置,比如在这个list后面再加一个元素,你需要在`'Dilbert'`后加一个逗号.这样很容易因为忘记修改逗号导致bug.比如:

```python
names = [
	'Alice',
	'Bob',
	'Dilbert' # 漏加了一个逗号
    'Jane'
]
```

这个list的打印结果是:

> ['Alice', 'Bob', 'DilbertJane']

Python把最后两行的字符串合并成了一个字符串.这被称为`字符串字面量拼接`.这是文档中明确说明的一种行为.它在有些情况下很管用,但也容易引起问题.

回到上面的代码,要解决这个bug只要加上缺失的逗号就行了,但这样每加一个元素都需要改两行代码显然是件麻烦的事情.解决的办法就是在每一个元素后都加逗号,包括最后一个元素,这种写法在Python中是合法的.

所以`names`这个list最好的写法是这样的：

```python
names = [
	'Alice',
	'Bob',
	'Dilbert',
]
```

## 上下文管理器和`with`表达式

对一些开发者来说,`with`被认为是一个很复杂的特性.但如果你一窥它的实质,你会发现其中没有什么魔法,而只是一个语法糖.
看一个比较常见的例子:

```python
with open('hello.txt', 'w') as f: 
	f.write('hello, world!')
```

这段代码在运行的时候会被转换成下面这样的代码:

```python
f = open('hello.txt', 'w') 
try:
	f.write('hello, world') 
finally:
	f.close()
```

其中的`try`和`finally`是必须的,因为这样写才能保证文件的descriptor最终会被释放,但这样写的不好的地方是比较啰嗦.而如果使用`with`,代码就简洁了不少.

在看一个使用`with`的例子:

```python
some_lock = threading.Lock()
with some_lock:
	# Do something...
```

在这两个例子中,`with`都让你把大多数对资源的处理的逻辑抽象出来.相比于冗长的`try...finally`,仅仅使用`with`就可以为你处理这些.

`with`不止可以提高处理系统资源相关的代码的可读性,还可以让忘记清理或释放不再需要的资源这种事情变得几乎不可能,这也避免了bug和泄露.

### 让自定义类型支持`with`

只要你的类实现了context manager,它就可以使用`with`声明.什么是context manager?它其实是一个简单的协议,只要类中有`__enter__`和`__exit__`这两个方法,它就算是context manager了.Python会在资源管理周期的适当时候调用它们.

现在试着自己实现类似open()的功能:

```python
class ManagedFile:
    def __init__(self, name):
        self.name = name

    def __enter__(self):
        self.file = open(self.name, 'w')
        return self.file

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()
```

使用它:

```python
with ManagedFile('hello.txt') as f:
	f.write('hello, world!')
	f.write('bye now')
```

Python会在进入`with`声明的上下文的时候调用`__enter__`方法来获取资源,在离开这个上下文的时候调用`__exit__`方法来是否资源.

写一个基于类的context manager并不是Python中唯一的支持`with`声明的方法.还有另一种方法,在标准库中的`contextlib`模块提供了一些建立于context manager协议之上的抽象.
如果你的使用场景适合,它可以让你更容易的支持`with`.

比如,你可以使用`contextlib.contextmanager`注解来标注一个基于生成器的资源工厂方法,然后这个方法就会自动支持`with`声明了.我们试着用这种方法重新实现`ManagedFile`这个例子.

```python
from contextlib import contextmanager
@contextmanager
def managed_file(name):
    try:
        file = open(name, 'w')
        yield file
    finally:
        file.close()
```

`managed_file`是一个生成器,它首先获取资源,然后它使用yield阻塞方法的继续执行并把资源返回给调用者.当调用者离开`with`,生成器会继续执行以便进行对资源的清理工作,资源就可以被释放.

这两种方法本质上是等价的,你可以选择适合你情况的,主要取决于你的团队的习惯以及哪种方法的可读性更好.

### 使用context manager写出更漂亮的API

实际上context manager不止能用于资源管理,它的灵活性很高,你可以用它写便利的API.
比如,你要写一个报表生成程序,需要处理文本缩进.想要达到这样的效果:

```python
with Indent() as indent:
    indent.print('hi!')
    with indent:
        indent.print('hello')
        with indent:
            indent.print('bonjour')
    indent.print('hey')
```

预期的输出:

> hi!
> &emsp;&emsp;hello
> &emsp;&emsp;&emsp;&emsp;bonjour 
> hey

这样的API读着很像DSL,基本做到了代码即结果,你看代码就很容易知道它运行的结果,可读性很高.下面是`Indent`的实现:

```python
class Indent:
    def __init__(self):
        self.indentSize = 0
        self.entered = False
        return

    def __enter__(self):
        if self.entered:
            self.indentSize += 1
        self.entered = True
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.indentSize:
            self.indentSize -= 1

    def print(self, s):
        indent_str = '\t' * self.indentSize
        print(indent_str, s)
```

## 下划线,dunders和其他

单下划线或双下划线在Python的变量名或方法名中很常见.它们实际是有一些含义的,其中一些只是约定俗成的习惯(如本地变量名使用单下划线分隔不同的单词,即snake case),还有一些则是Python解释器强制规定的.

你肯定会好奇单下划线和双下划线分别代表什么含义,接下来就会介绍五种下划线的模式和命名习惯以及它们是如何影响Python程序的行为的.

### 单下划线开头:"_var"

以这样的方法命名变量或方法,仅仅是一种Python社区的习惯.它对于程序员是一个暗示:这个变量或方法只在类的内部使用.这个惯例在PEP 8(最常用的Python编码规范)中有明确的规定.

但这种写法并不是Python解释器强制规定的,Python并没有像Java那样,`private`和`public`变量有着很大的区别.它就像是一个警告标志,告诉程序员不要把这个变量或方法当成这个类的公共接口的一部分.
例子:

```python
class Test:
    def __init__(self):
        self.foo = 11
        self._bar = 23
```

如果你实例化了这个类并访问`foo`和`_bar`属性会怎么样?

```python
test = Test()
print("foo:%d" % test.foo)
print("_bar:%d" % test._bar)
```

运行结果:

> foo:11
> _bar:23

所以,开头的下划线没有阻止`_bar`被外部代码访问,这只是一种编程规范.

但它确实会影响模块的导入.如果你使用wildcard import(import *)导入一个含有名称以单下划线开头的函数或变量的模块,那么这些单下划线开头的函数或变量不会被导入.
例子:

```python
def external_func():
    return 23


def _internal_func():
    return 42
```

在另一个Python文件中导入这个模块:

```python
from my_module import *
external_func()
_internal_func()
```

输出结果:

> 23
> NameError: "name '_internal_func' is not defined"

但如果你遵守PEP 8,那么wildcard import并不是推荐的做法,使用一般的import更好.一般的import也不会发生这种以下划线开头的字段没有被导入的情况.

```python
import my_module
my_module.external_func()
my_module._internal_func()
```

输出结果:

> 23
> 42

### 单下划线结尾:"var_"

有的时候,你觉得最适合这个方法或变量的单词已经被Python用作关键字了.所以像`class`,`def`不能作为变量的名称.这种情况下,你可以在这些词后面加一个下划线避免命名冲突.

```python
def make_object(name, class_):
    pass
```

总之,以单下划线结尾这种命名方式是用于防止命名冲突的,这个惯例也是在PEP 8中规定的.

### 双下划线开头:"__var"

之前的命名模式都只是一种编程惯例,但类中以双下划线开头的方法或变量就不一样了.这种写法会使解释器对它的名称重写以防止子类中的命名冲突.这种做法被称为名称重整(name mangling).

例子:

```python
class Test:
    def __init__(self):
        self.foo = 11
        self._bar = 23
        self.__baz = 23
```

用`dir()`来看看这个类的实例中的成员:

```python
t = Test()
dir(t)
```

> ['\_Test\_\_baz', '\_\_class\_\_', '\_\_delattr\_\_', '\_\_dict\_\_', '\_\_dir\_\_', '\_\_doc\_\_', '\_\_eq\_\_', '\_\_format\_\_', '\_\_ge\_\_', '\_\_getattribute\_\_', '\_\_gt\_\_', '\_\_hash\_\_', '\_\_init\_\_', '\_\_init\_subclass\_\_', '\_\_le\_\_', '\_\_lt\_\_', '\_\_module\_\_', '\_\_ne\_\_', '\_\_new\_\_', '\_\_reduce\_\_', '\_\_reduce\_ex\_\_', '\_\_repr\_\_', '\_\_setattr\_\_', '\_\_sizeof\_\_', '\_\_str\_\_', '\_\_subclasshook\_\_', '\_\_weakref\_\_', '\_bar', 'foo']

可以看出,`__baz`这个变量的名称被改为了`_Test__baz`.这是解释器的一种叫做名称重整的行为.这样做是为了防止这个变量被子类覆盖.

可以尝试写一个类来继承`Test`:

```python
class ExtendedTest(Test):
    def __init__(self):
        super().__init__()
        self.foo = 'overridden'
        self._bar = 'overridden'
        self.__baz = 'overridden'

t2 = ExtendedTest()
t2.foo
t2._bar
t2.__baz
```

输出:

> overridden
> overridden
>
> AttributeError:"'ExtendedTest' object has no attribute '__baz'"

为什么会报错?因为又发生了名称重整:

```python
dir(t2)
```

输出:

> ['\_ExtendedTest\_\_baz', '\_Test\_\_baz', '\_\_class\_\_', '\_\_delattr\_\_', '\_\_dict\_\_', '\_\_dir\_\_', '\_\_doc\_\_', '\_\_eq\_\_', '\_\_format\_\_', '\_\_ge\_\_', '\_\_getattribute\_\_', '\_\_gt\_\_', '\_\_hash\_\_', '\_\_init\_\_', '\_\_init\_subclass\_\_', '\_\_le\_\_', '\_\_lt\_\_', '\_\_module\_\_', '\_\_ne\_\_', '\_\_new\_\_', '\_\_reduce\_\_', '\_\_reduce\_ex\_\_', '\_\_repr\_\_', '\_\_setattr\_\_', '\_\_sizeof\_\_', '\_\_str\_\_', '\_\_subclasshook\_\_', '\_\_weakref\_\_', '\_bar', 'foo']

可以看到, `ExtendedTest`的 `__baz`变成了`_ExtendedTest__baz`.而且父类的`_Test__baz`也仍然存在.

```python
t2._Test__baz
t2._ExtendedTest__baz
```

名称重整对程序员来说是完全透明的.看一个例子:

```python
class ManglingTest:
    def __init__(self) -> None:
        self.__mangled = 'hello'

    def get_mangled(self):
        return self.__mangled

mt = ManglingTest()
mt.get_mangled()
```

输出:

> hello

可以看出,在类内部访问以双引号开头的变量`__mangled`,是不需要知道它经过重整后的名字的.

名称重整也会作用于方法.例子:

```python
class MangledMethod:
    def __method(self):
        return 42

    def call_it(self):
        return self.__method()

mm = MangledMethod()
mm.call_it()
mm.__method()
```

输出:

> 42
>
> AttributeError: 'MangledMethod' object has no attribute '__method'

可以看出,因为名称重整,`MangledMethod`不存在名为`__method`的成员了.

下面看一个会让人惊讶的例子:

```python
_MangledGlobal__mangled = 11


class MangledGlobal:
    def test(self):
        return __mangled

MangledGlobal().test()
```

输出:

> 11

`_MangledGlobal__mangled`是一个全局变量,但在`MangledGlobal`的内部,居然通过`__mangled`就可以访问它.显然,名称重整在其中又起了作用.这个例子显示了名称重整针对的是类中出现的所有以双下划线开头的变量,而不只是内的成员变量.

### 啥是dunders?

如果你曾听过一些Python老鸟讨论Python或一些Python相关的演讲,你可能听到过这个单词:`dunder`.你肯定会疑惑这个单词时啥意思.

其实答案很简单:这是Python社区对`double underscore`的称呼.原因是双下滑线在Python中很常见,以至于如果总是使用英文`double underscore`称呼它会让你下巴脱臼,所以使用了这么一个缩写来指代它.例如,`__baz`读作"dunder  baz",`__init__`读作"dunder init". 

### 前后双下划线:"\_\_var\_\_"

说起来可能会让你惊讶,名称重整不会作用于名称以双下划线开头和结尾的变量.例子:

```python
class PrefixPostfixTest:
    def __init__(self) -> None:
        self.__bam__ = 42

PrefixPostfixTest().__bam__
```

输出:

> 42

但是,在Python中以双下划线开头和结尾的名称是被保留做特殊用途的.比如`__init__`用于对象构造器,`__call__`是对象可以像方法一样调用.这些方法被称为"dunder methods"或"magic methods",但后者并不合适,它会让人觉得使用这些方法并不被鼓励(就像magic number),而实际情况并非如此.它们是Python语言的核心要素,在必要的时候就应该使用它们.它们没有什么神奇的或难懂的东西.

我们还是要**避免使用这种命名模式**,因为这可能让你的代码和未来的Python版本发生冲突.

### 单下划线:"_"

单个下划线作为变量名,说明这个变量是临时的或无足轻重的,给它一个名称没什么意义,所以就是简单的用`_`来为它命名.

例子:

```python
for _ in range(3):
    print('Hello,World')
```

还可以在解包的时候用`_`表示一个你不关心的变量.这只是一个命名习惯,不会触发任何解释器的行为.`_`仅仅是一个有效的变量名.

```python
car = ('red', 'auto', 12, 3812.4)
color, _, _, mileage = car
color
mileage
```

输出:

> red
>
> 3812.4

在大多数Python REPL中,`_`还表示最近一个表达式的结果.

## 关于字符串格式化的惊人事实

在著名的<<Python之禅>>中,有一句是说"应该有一个明确的方式去完成某件事".但事实上,Python中却有四种方法可以对字符串进行格式化!

先定义两个变量:

```python
errno = 50159747054
name = 'Bob'
```

我们想要基于这两个变量来生成一个字符串:

> Hey Bob, there is a 0xbadc0ffee error!

接下来会分别使用四种方式来完成这个任务,同时对它们的优点和缺点逐一介绍,并阐述选取最合适的字符串格式化方式的经验准则.

### "旧式"的字符串格式化

Python的字符串有一个独特的内置操作,通过操作符`%`调用.它是一个进行基于位置的格式化的快捷操作.类似于C中的`printf`.例子:

```python
'Hello, %s' % name
```

输出:

> Hello,Bob

其中的`%s`是格式声明符,它会告诉Python在字符串的什么位置替换为`name`的值,以字符串的形式.还有很多其他的格式声明符可以用来控制输出的格式,比如有的可以把数字转换成16进制表示或填充空格来生成格式漂亮的表和报告.

`%x`就可以把int类型的值转成string并以16进制表示:

```python
n = '%x' % errno
```

输出:

> badc0ffee

如果你想要对单个字符串进行多次替换,你需要把这多个参数放到一个元组中,因为`%`只接收一个参数.

```python
'Hey %s, there is a 0x%x error!' % (name, errno)
```

输出:

> Hey Bob, there is a 0xbadc0ffee error!

你还可以在格式化字符中通过名称来指向替换变量,这种情况传递给`%`的参数是一个名称和遍历的映射关系.例子:

```python
'Hey %(name)s, there is a 0x%(errno)x error!' % {"name": name, "errno": errno}
```

输出:

> Hey Bob, there is a 0xbadc0ffee error!

这种方式会让你的代码更容易维护和修改,因为你不需要关系传入的值的顺序是否和格式化字符串中的一致.当然,它的缺点是会增加代码量.

### "新式"的字符串格式化

Python3引入了一种新的字符串格式化的方式,它后来还被移植到了2.7中.新的方式摆脱了奇怪的`%`符号,使得字符串的格式化更加标准.格式化现在只要对字符串对象调用`format`方法就可以了.例子:

```python
s = 'Hello,{}'.format(name)
```

输出:

> Hello,Bob

这种方法同样可以在格式化字符中通过名称来指向替换变量.例子:

```python
'Hey {name}, there is a 0x{errno:x} error!'.format(name=name, errno=errno)
```

输出:

> Hey Bob, there is a 0xbadc0ffee error!

这也展示了如何格式化一个int值为十六进制的字符串,只需要在字符串中的变量名后加一个`:x`后缀.

在Python3中,这种方式比旧式的更受人喜欢,但从Python3.6开始,有了一种更好的格式化字符串的方法.

### 字符串字面量插入(Python3.6+)

Python3.6加入加入了一种叫做格式化字符串字面量的方法.这种格式化字符串的方法让我们可以再字符串常量中加入内嵌的表达式.一个简单的例子:

```python
f'Hello,{name}'
```

这种方法非常强大,因为你可以嵌入任何的表达式,包括算术运算.例子:

```python
f'Five plus ten is {a + b} and not {2 * (a + b)}.'
```

输出:

> Five plus ten is 15 and not 30.

在底层,格式化字符串字面量是Python解释器的一项特性,它会把这些f开头的字符串转换成字符串常量和表达式,然后把他们拼接起来来组成最终的字符串.

一个例子:

```python
def greet(name, question):
	    return f"Hello, {name}! How's it {question}?"
```

把这个函数反汇编,你可以看到这个函数被转换成了类似下面的形式:

```python
def greet(name, question):
	    return ("Hello, " + name + "! How's it " + question + "?")
```

当然,实际的实现方式更搞笑,它使用了`BUILD_STRING`这个opcode(Python源代码编译之后的格式,类似Java字节码)作为优化.但它们最终的效果是一样的.

它也支持和`format`方法类似的格式化语法.例子:

```python
f"Hey {name}, there is a {errno:#x} error!"
```

输出:

> Hey Bob, there is a 0xbadc0ffee error!

### 模板字符串

还有一个字符串格式化的方式是模板字符串,这种方式更简单,但没那么强大.有些情况使用它还是非常合适的.

例子:

```python
from string import Template
t = Template('Hello,$name')
t.substitute(name=name)
```

输出:

> Hello,Bob

可以看到使用模板字符串需要导入`Template`这个类.它不是Python的一个核心语言特性,但它存在于标准库中.

另一个区别是,它不支持格式化声明符.所以需要我们自己把int转换成十六进制的字符串.

```python
templ_string = 'Hey $name, there is a $error error!'
Template(templ_string).substitute(name=name, error=hex(errno))
```

什么时候使用这个方法呢?一般来说,如果格式化字符串是用户自己输入的,那么可以使用这个方法,因为它更加安全.看一个例子:

```python
SECRET = 'this-is-a-secret'

class Error:
    def __init__(self):
        pass

err = Error()
user_input = '{error.__init__.__globals__[SECRET]}'
user_input.format(error=err)
```

输出:

> this-is-a-secret

可以看到,攻击者可以拿到`SECRET`这个字符串的,这是一个安全漏洞.而模板字符串可以避免这个问题.

例子:

```python
user_input = '${error.__init__.__globals__[SECRET]}'
Template(user_input).substitute(error=err)
```

输出:

> ValueError:"Invalid placeholder in string: line 1, col 1"

### 怎么选择格式化字符串的方式

有一个简单的规则:如果需要格式化的字符串是用户提供的,那么使用模板字符串来避免安全问题.否则,使用格式化字符串字面量如果是在Python3.6及以上版本,使用字符串的`format`的方法如果是在较旧的版本.

## Python之禅

只要是Python的书,基本绕不开Tim Peter的<<Python之禅>>.这首诗让人受益良多,历久弥新.它可以让我们成为一个更好的程序员,因为它里面的这些道理适用于几乎所有编程语而不局限于Python.

Python中有一个关于它的彩蛋,在Python REPL中输入下面的指令:

> import this

<<Python之禅>>就会被打印出来:

> The Zen of Python, by Tim Peters
>
> Beautiful is better than ugly.
> Explicit is better than implicit.
> Simple is better than complex.
> Complex is better than complicated.
> Flat is better than nested.
> Sparse is better than dense.
> Readability counts.
> Special cases aren't special enough to break the rules.
> Although practicality beats purity.
> Errors should never pass silently.
> Unless explicitly silenced.
> In the face of ambiguity, refuse the temptation to guess.
> There should be one-- and preferably only one --obvious way to do it.
> Although that way may not be obvious at first unless you're Dutch.
> Now is better than never.
> Although never is often better than *right* now.
> If the implementation is hard to explain, it's a bad idea.
> If the implementation is easy to explain, it may be a good idea.
> Namespaces are one honking great idea -- let's do more of those!
