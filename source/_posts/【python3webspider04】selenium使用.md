---
title: 【Python3WebSpider04】selenium使用
tags:
  - Python
  - 爬虫
id: '12197'
categories:
  - - skills
date: 2020-09-18 14:02:49
---

# 介绍

Selenium是一个自动化测试工具，利用它可以驱动浏览器执行特定的动作，如点击、下拉等操作，同时还可以获取浏览器当前呈现的页面的源代码，做到可见即可爬。
<!--more-->
# 准备

本文将以Chrome为例来讲解Selenium的使用，在使用之前需要先安装Chrome浏览器并配置好了ChromeDriver。并且需要正确安装好Python的Selenium库。

# 示例

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.wait import WebDriverWait

browser = webdriver.Chrome()
try:
    browser.get('https://www.baidu.com')
    input = browser.find_element_by_id('kw')
    input.send_keys('Python')
    input.send_keys(Keys.ENTER)
    wait = WebDriverWait(browser, 10)
    wait.until(EC.presence_of_element_located((By.ID, 'content_left')))
    print(browser.current_url)
    print(browser.get_cookies())
    print(browser.page_source)
finally:
    browser.close()
```

运行该代码之后，会自动弹出一个Chrome浏览器，浏览器首先会跳转到百度，然后在搜索框中输入Python，接着跳转到搜索结果页。 [![](https://i.loli.net/2020/09/18/OlXhWfrm3ZaSL8q.jpg)](https://i.loli.net/2020/09/18/OlXhWfrm3ZaSL8q.jpg) 控制台会打印出当前的URL、当前的Cookies和网页源代码

# 详细讲解

## 声明浏览器对象

Selenium支持非常多的浏览器，如Chrome、Firefox、Edge等，还有Android、BlackBerry等手机端的浏览器。另外，也支持无界面浏览器PhantomJS 我们可以用如下方式初始化：

```python
from selenium import webdriver

browser = webdriver.Chrome()
browser = webdriver.Firefox()
browser = webdriver.Edge()
browser = webdriver.PhantomJS()
browser = webdriver.Safari()
```

这样就完成了浏览器对象的初始化并将其赋值为browser对象。接下来，我们要做的就是调用browser对象，让其执行各个动作以模拟浏览器操作。

## 访问页面

可以使用`get()`方法来请求页面，参数传入要访问的链接即可。比如用`get()`方法来访问`Google`,然后打印出页面源代码

```python
from selenium import webdriver
browser = webdriver.Chrome()
browser.get("https://www.google.com")
print(browser.page_source)
browser.close()
```

运行之后，会打开一个Chrome页面并且自动访问Google，然后控制台输出Google首页源代码。

## 查找节点

Selenium可以驱动浏览器完成各种操作，比如填充表单、模拟点击等。比如，我们想要完成向某个输入框输入文字的操作，总需要知道这个输入框在哪里吧？而Selenium提供了一系列查找节点的方法，我们可以用这些方法来获取想要的节点，以便下一步执行一些动作或者提取信息

#### 单个节点

比如想要从Google首页提取搜索框这个节点，首先要观察它的源代码。 [![](https://i.loli.net/2020/09/18/NysA6f8Jt15EMiC.jpg)](https://i.loli.net/2020/09/18/NysA6f8Jt15EMiC.jpg) 可以发现，它的name是`q`。此外，还有许多其他属性，此时我们就可以用多种方式获取它了。比如，`find_element_by_name()`是根据name值获取，`find_element_by_id()`是根据id获取。另外，还有根据XPath、CSS选择器等获取的方式。 用代码实现一下

```python
from selenium import webdriver
browser = webdriver.Chrome()
browser.get("https://www.google.com")
input = browser.find_element_by_name('q')
print(input)
browser.close()
```

打印出结果为：

```
<selenium.webdriver.remote.webelement.WebElement (session="5f9629a45a89e0879c203f1bb568be19", element="470af759-e2bc-4346-bbc4-e54b1655d106")>
```

下面列出所有获取单个节点 方法：

```python
find_element_by_id
find_element_by_name
find_element_by_xpath
find_element_by_link_text
find_element_by_partial_link_text
find_element_by_tag_name
find_element_by_class_name
find_element_by_css_selector
```

另外，selenium还提供了一个通用方法：`find_element()`,`find_element_by_id(id)`就等价于`find_element(By.ID, id)`，二者得到的结果完全一致。

#### 多个节点

如果查找的目标在网页中只有一个，那么完全可以用`find_element()`方法。但如果有多个节点，再用`find_element()`方法查找，就只能得到第一个节点了。如果要查找所有满足条件的节点，需要用`find_elements()`这样的方法。注意，在这个方法的名称中，element多了一个s，注意区分。 比如，查找京东左侧导航栏所有条目： [![](https://i.loli.net/2020/09/18/EvJyupHZXthdFUl.jpg)](https://i.loli.net/2020/09/18/EvJyupHZXthdFUl.jpg)

```python
from selenium import webdriver
browser = webdriver.Chrome()
browser.get("https://www.jd.com")
lis = browser.find_elements_by_class_name('cate_menu_lk')
print(lis)
browser.close()
```

运结果如下： [![](https://i.loli.net/2020/09/18/jX7Z86IvSiQrpos.jpg)](https://i.loli.net/2020/09/18/jX7Z86IvSiQrpos.jpg) 其他获取多个节点的方法：

```python
find_elements_by_id
find_elements_by_name
find_elements_by_xpath
find_elements_by_link_text
find_elements_by_partial_link_text
find_elements_by_tag_name
find_elements_by_class_name
find_elements_by_css_selector
```

#### 节点交互

Selenium可以驱动浏览器来执行一些操作，也就是说可以让浏览器模拟执行一些动作。比较常见的用法有：输入文字时用send\_keys()方法，清空文字时用clear()方法，点击按钮时用click()方法。示例如下：

```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import time

browser = webdriver.Chrome()
browser.get("https://www.google.com")
input = browser.find_element_by_name('q')
input.send_keys("Python 爬虫")
time.sleep(2)
input.clear()
input.send_keys("Selenium 自动化")
input.send_keys(Keys.ENTER)
browser.close()
```

运行如下 ![2020-09-18 15.11.36.gif](https://i.loli.net/2020/09/18/VXq974pitzAYmRP.gif) 这里首先驱动浏览器打开google，然后用find\_element\_by\_name()方法获取输入框，然后用send\_keys()方法输入"Python 爬虫"文字，等待一秒后用clear()方法清空输入框，再次调用send\_keys()方法输入"Selenium 自动化"文字，之后回车进行搜索

#### 动作链

上面几个例子，一些交互动作都是基于某个节点执行的，比如对于输入框，我们就调用它的输入文字和清空文字方法；对于按钮，就调用它的点击方法。其实，还有另外一些操作，它们没有特定的执行对象，比如鼠标拖曳、键盘按键等，这些动作用另一种方式来执行，那就是动作链。 比如，现在实现一个节点的拖曳操作，将某个节点从一处拖曳到另外一处，可以这样实现：

```python
from selenium import webdriver
from selenium.webdriver import ActionChains
import time

browser = webdriver.Chrome()
url = 'http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable' 
browser.get(url)
browser.switch_to.frame('iframeResult')
source = browser.find_element_by_css_selector('#draggable')
target = browser.find_element_by_css_selector('#droppable')
actions = ActionChains(browser)
actions.drag_and_drop(source, target)
actions.perform()
browser.close()
```

![2020-09-18 15.25.22.gif](https://i.loli.net/2020/09/18/SeVs3HTJG4NLW52.gif) 首先，打开网页中的一个拖曳实例，然后依次选中要拖曳的节点和拖曳到的目标节点，接着声明ActionChains对象并将其赋值为actions变量，然后通过调用actions变量的drag\_and\_drop()方法，再调用perform()方法执行动作，此时就完成了拖曳操作

#### 获取节点信息

前面说过，通过page\_source属性可以获取网页的源代码，接着就可以使用解析库（如正则表达式、Beautiful Soup、pyquery等）来提取信息了。 不过，既然Selenium已经提供了选择节点的方法，返回的是WebElement类型，那么它也有相关的方法和属性来直接提取节点信息，如属性、文本等。这样的话，我们就可以不用通过解析源代码来提取信息了，非常方便 **获取属性** 我们可以使用get\_attribute()方法来获取节点的属性，但是其前提是先选中这个节点，示例如下：

```python
from selenium import webdriver
from selenium.webdriver import ActionChains

browser = webdriver.Chrome()
url = 'https://www.zhihu.com/explore'
browser.get(url)
logo = browser.find_element_by_id('zh-top-link-logo')
print(logo)
print(logo.get_attribute('class'))
```

![2020-09-18 15.39.45.gif](https://i.loli.net/2020/09/18/zrcwMSXdCWTEiYk.gif) **获取文本值** 每个WebElement节点都有text属性，直接调用这个属性就可以得到节点内部的文本信息，这相当于Beautiful Soup的get\_text()方法、pyquery的text()方法，示例如下：

```python
from selenium import webdriver

browser = webdriver.Chrome()
url = 'https://www.zhihu.com/explore'
browser.get(url)
input = browser.find_element_by_class_name('ExploreHomePage-specialsLoginDes')
print(input.text)
```

#### 前进和后退

平常使用浏览器时都有前进和后退功能，Selenium也可以完成这个操作，它使用back()方法后退，使用forward()方法前进。示例如下：

```python
import time
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.baidu.com/')
browser.get('https://www.taobao.com/')
browser.get('https://www.python.org/')
browser.back()
time.sleep(1)
browser.forward()
browser.close()
```

#### cookies

使用Selenium，还可以方便地对Cookies进行操作，例如获取、添加、删除Cookies等。示例如下：

```python
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.zhihu.com/explore')
print(browser.get_cookies())
browser.add_cookie({'name': 'name', 'domain': 'www.zhihu.com', 'value': 'germey'})
print(browser.get_cookies())
browser.delete_all_cookies()
print(browser.get_cookies())
```

#### 选项卡管理

在访问网页的时候，会开启一个个选项卡。在Selenium中，我们也可以对选项卡进行操作。示例如下：

```python
import time
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.baidu.com')
browser.execute_script('window.open()')
print(browser.window_handles)
browser.switch_to_window(browser.window_handles[1])
browser.get('https://www.taobao.com')
time.sleep(1)
browser.switch_to_window(browser.window_handles[0])
browser.get('https://python.org')
```