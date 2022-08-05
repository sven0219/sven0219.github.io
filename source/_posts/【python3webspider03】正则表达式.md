---
title: 【Python3WebSpider03】正则表达式
tags:
  - Python
  - 正则
  - 爬虫
id: '12133'
categories:
  - - skills
date: 2020-08-10 15:58:46
---

# 正则表达式

正则表达式是处理字符串的强大工具，它有自己特定的语法结构，可以实现字符串的检索、替换、匹配认证，是爬虫学习中必不可少的一个工具

### 常用匹配规则

模式

描述

`w`

匹配字母、数字及下划线

`W`

匹配不是字母、数字及下划线的字符

`s`

匹配任意空白字符，等价于`[tnrf]`

`S`

匹配任意非空字符

`d`

匹配任意数字，等价于`[0-9]`

`D`

匹配任意非数字的字符

`A`

匹配字符串开头

`Z`

匹配字符串结尾，如果存在换行，只匹配换行前的结束字符串

`z`

匹配字符串结尾，如果存在换行，同时还会匹配换行符

`G`

匹配最后匹配完成的位置

`n`

匹配一个换行符

`t`

匹配一个制表符

`^`

匹配一行字符串的开头

`$`

匹配一行字符串的结尾

`.`

匹配任意字符，除了换行符，当`re.DOTALL`标记被指定时，则可以匹配包括换行符的任意字符

`[...]`

用来表示一组字符，单独列出，比如`[amk]`匹配a、m或k

`[^...]`

不在`[]`中的字符，比如`[^abc]`匹配除了`a`、`b`、`c`之外的字符

`*`

匹配0个或多个表达式

`+`

匹配1个或多个表达式

`?`

匹配0个或1个前面的正则表达式定义的片段，非贪婪方式

`{n}`

精确匹配n个前面的表达式

`{n,m}`

匹配n到m次由前面正则表达式定义的片段，贪婪方式

`ab`

匹配a或b

`()`

匹配括号内的表达式，也表示一个组

### 方法介绍

#### `match()`

`match()`是常见的匹配方法，向 `match()`传入要匹配的字符串以及正则表达式，就可以检测这个正则表达式是否匹配字符串 `match()`方法会尝试从字符串的起始位置匹配政策表达式，如果匹配，就返回匹配成功的结果；如果不匹配，就返回`None`。 示例：

```python
import re

content = 'Hello 123 4567 World_This is a Regex Demo'
print(len(content))
result = re.match('^Hellosdddsd{4}sw{10}', content)
print(result)
print(result.group())
print(result.span())
```

运行结果： [![](https://i.loli.net/2020/08/10/vN6Hd2jcyTPsfSo.jpg)](https://i.loli.net/2020/08/10/vN6Hd2jcyTPsfSo.jpg)

##### 匹配目标

上面例子是匹配得到字符串内容，如果想从字符串中提取一部分内容可以使用如下方法 可以使用()括号将想提取的子字符串括起来。()实际上标记了一个子表达式的开始和结束位置，被标记的每个子表达式会依次对应每一个分组，调用group()方法传入分组的索引即可获取提取的结果

```python
import re

content = 'Hello 1234567 World_This is a Regex Demo'
result = re.match('^Hellos(d+)sWorld', content)
print(result)
print(result.group())
print(result.group(1))
print(result.span())
```

运行结果 [![](https://i.loli.net/2020/08/13/U2DACisTvJwIgyP.jpg)](https://i.loli.net/2020/08/13/U2DACisTvJwIgyP.jpg) 假如正则表达式后面还有()包括的内容，那么可以依次用group(2)、group(3)等来获取。

##### 通用匹配

上面匹配工作量非常大，可以使用`.*`匹配任意字符。

```python
import re

content = 'Hello 123 4567 World_This is a Regex Demo'
result = re.match('^Hello.*Demo$', content)
print(result)
print(result.group())
print(result.span())
```

运行结果 [![](https://i.loli.net/2020/08/13/23KQNOtCkAJSon5.jpg)](https://i.loli.net/2020/08/13/23KQNOtCkAJSon5.jpg)

##### 贪婪和非贪婪

使用通用匹配有时候得等到的结果可能不是我们想要的结果：

```python
import re

content = 'Hello 1234567 World_This is a Regex Demo'
result = re.match('^He.*(d+).*Demo$', content)
print(result)
print(result.group(1))
```

结果 [![](https://i.loli.net/2020/08/13/jsMemxtwiG697uf.jpg)](https://i.loli.net/2020/08/13/jsMemxtwiG697uf.jpg) 这里得到数字只有 7，而不是`1234567`,这里涉及到贪婪匹配和非贪婪匹配的问题，`.*`会匹配尽可能多的字符，正则表达式中`.*`后面是`d+`，也就是至少一个数字，并没有指定具体多少个数字，因此，`.*`就尽可能匹配多的字符，这里就把`123456`匹配了，给`d+`留下一个可满足条件的数字`7`，最后得到的内容就只有数字`7`了。这里就需要使用非贪婪模式`.*?`匹配

```python
import re

content = 'Hello 1234567 World_This is a Regex Demo'
result = re.match('^He.*?(d+).*Demo$', content)
print(result)
print(result.group(1))
```

[![](https://i.loli.net/2020/08/13/XNxERJFauW36vif.jpg)](https://i.loli.net/2020/08/13/XNxERJFauW36vif.jpg) 贪婪匹配是尽可能匹配多的字符，非贪婪匹配就是尽可能匹配少的字符。当`.*?`匹配到`Hello`后面的空白字符时，再往后的字符就是数字了，而`d+`恰好可以匹配，那么这里`.*?`就不再进行匹配，交给`d+`去匹配后面的数字。所以这样`.*?`匹配了尽可能少的字符，`d+`的结果就是`1234567`了。

##### 修饰符

正则表达式可以包含一些可选标志修饰符来控制匹配的模式。修饰符被指定为一个可选的标志。 在字符串中加入换行符之后

```python
import re

content = '''Hello 1234567 World_This
is a Regex Demo
'''
result = re.match('^He.*?(\d+).*?Demo$', content)
print(result.group(1))
```

[![](https://i.loli.net/2020/08/13/QrtGAR1vzPcae3N.jpg)](https://i.loli.net/2020/08/13/QrtGAR1vzPcae3N.jpg) 运行直接报错，因为`\.`匹配的是除换行符之外的任意字符，当遇到换行符时，`.*?`就不能用，这里需要加个修饰符`re.S`修正这个错误

```python
result = re.match('^He.*?(\d+).*?Demo$', content, re.S)
```

这个修饰符的作用是使.匹配包括换行符在内的所有字符。 在网页匹配中经常要用到这个修饰符，因为 html 节点经常会有换行。除了这个修饰符，还有其它修饰符

修饰符

描述

`re.I`

使匹配对大小写不敏感

`re.L`

做本地化识别（`locale-aware`）匹配

`re.M`

多行匹配，影响`^`和`$`

`re.S`

使.匹配包括换行在内的所有字符

`re.U`

根据Unicode字符集解析字符。这个标志影响\\w、`\W`、 `\b`和`\B`

`re.X`

该标志通过给予你更灵活的格式以便你将正则表达式写得更易于理解

##### 转义字符

我们知道正则表达式定义了许多匹配模式，如.匹配除换行符以外的任意字符，但是如果目标字符串里面就包含. 这里就需要用到转义匹配了`\`

#### `search()`

`match()`方法是从字符串开头开始匹配的，一旦开头不匹配，那么整个匹配就失败了。

```python
import re

content = 'Extra stings Hello 1234567 World_This is a Regex Demo Extra stings'
result = re.match('Hello.*?(\d+).*?Demo', content)
print(result.group(1))
```

运行结果 [![](https://i.loli.net/2020/08/13/DkdmTNpwq8jWOE9.jpg)](https://i.loli.net/2020/08/13/DkdmTNpwq8jWOE9.jpg) 这里返回的是 none，因为`match()`方法使用时需要考虑开头的内容，这在做匹配时并不方便。它更适合用来检测某个字符串是否符合某个正则表达式的规则。 `search()`，它在匹配时会扫描整个字符串，然后返回第一个成功匹配的结果。也就是说，正则表达式可以是字符串的一部分，在匹配时，`search()`方法会依次扫描字符串，直到找到第一个符合规则的字符串，然后返回匹配内容，如果搜索完了还没有找到，就返回`None`。 我们把上面代码中的`match()`方法修改成`search()`，再看下运行结果： [![](https://i.loli.net/2020/08/13/dSA8gX6RLTpkiGU.jpg)](https://i.loli.net/2020/08/13/dSA8gX6RLTpkiGU.jpg) 因此，为了匹配方便，匹配时可以尽量使用`search()`方法 使用一个实例看看`search()`的用法，准备一个待匹配的`HTML`文档：

```markup
html = '''<div id="songs-list">
    <h2 class="title">经典老歌</h2>
    <p class="introduction">
        经典老歌列表
    </p>
    <ul id="list" class="list-group">
        <li data-view="2">一路上有你</li>
        <li data-view="7">
            <a href="/2.mp3" singer="任贤齐">沧海一声笑</a>
        </li>
        <li data-view="4" class="active">
            <a href="/3.mp3" singer="齐秦">往事随风</a>
        </li>
        <li data-view="6"><a href="/4.mp3" singer="beyond">光辉岁月</a></li>
        <li data-view="5"><a href="/5.mp3" singer="陈慧琳">记事本</a></li>
        <li data-view="5">
            <a href="/6.mp3" singer="邓丽君"><i class="fa fa-user"></i>但愿人长久</a>
        </li>
    </ul>
</div>'''
```

我们尝试提取class为active的li节点内部的超链接包含的歌手名和歌名，此时需要提取第三个li节点下a节点的singer属性和文本。

```python
import re

html = '''<div id="songs-list">
    <h2 class="title">经典老歌</h2>
    <p class="introduction">
        经典老歌列表
    </p>
    <ul id="list" class="list-group">
        <li data-view="2">一路上有你</li>
        <li data-view="7">
            <a href="/2.mp3" singer="任贤齐">沧海一声笑</a>
        </li>
        <li data-view="4" class="active">
            <a href="/3.mp3" singer="齐秦">往事随风</a>
        </li>
        <li data-view="6"><a href="/4.mp3" singer="beyond">光辉岁月</a></li>
        <li data-view="5"><a href="/5.mp3" singer="陈慧琳">记事本</a></li>
        <li data-view="5">
            <a href="/6.mp3" singer="邓丽君"><i class="fa fa-user"></i>但愿人长久</a>
        </li>
    </ul>
</div>'''
result = re.search('<li.*?active.*?singer="(.*?)">(.*?)</a>', html, re.S)
if result:
    print(result.group(1), result.group(2))
```

结果 [![](https://i.loli.net/2020/08/13/JEr863zZGIfFOuw.jpg)](https://i.loli.net/2020/08/13/JEr863zZGIfFOuw.jpg)

##### `findall()`

`search()`方法的用法，可以匹配正则表达式的第一个内容，但是要获取匹配的所有内容得借助`findall()`方法。 还是上面的 HTML文档，如果想要获取所有 a 节点的超链接、歌手和歌名

```python
import re

html = '''<div id="songs-list">
    <h2 class="title">经典老歌</h2>
    <p class="introduction">
        经典老歌列表
    </p>
    <ul id="list" class="list-group">
        <li data-view="2">一路上有你</li>
        <li data-view="7">
            <a href="/2.mp3" singer="任贤齐">沧海一声笑</a>
        </li>
        <li data-view="4" class="active">
            <a href="/3.mp3" singer="齐秦">往事随风</a>
        </li>
        <li data-view="6"><a href="/4.mp3" singer="beyond">光辉岁月</a></li>
        <li data-view="5"><a href="/5.mp3" singer="陈慧琳">记事本</a></li>
        <li data-view="5">
            <a href="/6.mp3" singer="邓丽君"><i class="fa fa-user"></i>但愿人长久</a>
        </li>
    </ul>
</div>'''
results = re.findall('<li.*?href="(.*?)".*?singer="(.*?)">(.*?)</a>', html, re.S)
print(type(results))
for result in results:
    print(result)
    print(result[0], result[1], result[2])
```

运行结果 [![](https://i.loli.net/2020/08/13/o4xKuqzslGcpfF8.jpg)](https://i.loli.net/2020/08/13/o4xKuqzslGcpfF8.jpg) 可以看到，返回的列表中的每个元素都是元组类型，我们用对应的索引依次取出即可。 如果只是获取第一个内容，可以用search()方法。当需要提取多个内容时，可以用findall()方法

##### `sub()`

除了使用正则表达式提取信息外，有时候还需要借助它来修改文本。比如，想要把一串文本中的所有数字都去掉，如果只用字符串的replace()方法，那就太烦琐了，这时可以借助sub()方法。示例如下：

```python
import re

content = '54aK54yr5oiR54ix5L2g'
content = re.sub('\d+', '', content)
print(content)
```

运行结果

```
aKyroiRixLg
```

##### `compile()`

前面所讲的方法都是用来处理字符串的方法，最后再介绍一下compile()方法，这个方法可以将正则字符串编译成正则表达式对象，以便在后面的匹配中复用。

```python
import re

content1 = '2016-12-15 12:00'
content2 = '2016-12-17 12:55'
content3 = '2016-12-22 13:21'
pattern = re.compile('\d{2}:\d{2}')
result1 = re.sub(pattern, '', content1)
result2 = re.sub(pattern, '', content2)
result3 = re.sub(pattern, '', content3)
print(result1, result2, result3)
```

例如，这里有3个日期，我们想分别将3个日期中的时间去掉，这时可以借助sub()方法。该方法的第一个参数是正则表达式，但是这里没有必要重复写3个同样的正则表达式，此时可以借助compile()方法将正则表达式编译成一个正则表达式对象，以便复用。 运行结果如下：

```
2016-12-15  2016-12-17  2016-12-22
```

另外，compile()还可以传入修饰符，例如re.S等修饰符，这样在search()、findall()等方法中就不需要额外传了。所以，compile()方法可以说是给正则表达式做了一层封装，以便我们更好地复用。