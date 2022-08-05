---
title: Python 自动化运维（04）
tags:
  - Python
  - 自动化
  - 运维
id: '12045'
categories:
  - - skills
date: 2020-04-01 20:02:34
---

> 操作系统 macOS Catalina Python版本：3.7 所有代码可在 [GitHub](https://github.com/Swq1227/devops "GitHub") 获取

# 文件与目录差异对比方法

## 1\. 文件内容差异对比方法

python中可以通过 difflib 模块实现文件文件内容差异对比。difflib 作为 python 的标准库模块，无需安装，作用是对比文本之间的差异，且支持输出可读性比较强的 HTML 文档，与 Linux 下的 diff 命令相似。

#### 两个字符串的差异对比

```python
import difflib

text1 = """text1:
This module provides classes and functions for comparing sequences.
including HTML and context and unified diffs.
difflib document v7.4
add string
"""
text1_lines = text1.splitlines()  # 以行进行分隔，以便进行对比
text2 = """text2"
This module provides classes and functions for comparing sequences.
including HTML and context and unified diffs.
difflib document v7.5
"""
text2_lines = text2.splitlines()
d = difflib.Differ()  # 创建 Differ（) 对象
diff = d.compare(text1_lines, text2_lines)  # 采用 compare 方法对字符串进行比较
print('\n'.join(list(diff)))

```

执行结果 ![image.png](https://i.loli.net/2020/04/01/ad3cz5oVe78WXnS.png) **结果中有一些符号，下面进行说明：**

符号

含义

'-'

包含在第一个序列中，但不包含在第二个序列中

'+'

包含在第二个序列中，但不包含在第一个序列中

' '

两个序列行一致

'?'

标志两个序列行存在增量差异

'^'

标志处两个序列行存在差异字符

#### 生成 HTML 格式文档

采用 HtmlDiff() 类的 make\_file（）方法就可以生成 HTML 文档，对上述代码进行修改，将输出写入文件

```
import difflib

text1 = """text1:
This module provides classes and functions for comparing sequences.
including HTML and context and unified diffs.
difflib document v7.4
add string
"""
text1_lines = text1.splitlines()  # 以行进行分隔，以便进行对比
text2 = """text2:
This module provides classes and functions for comparing sequences.
including HTML and context and unified diffs.
difflib document v7.5
"""
text2_lines = text2.splitlines()
d = difflib.HtmlDiff()  # 创建对象
html = open("./diff.html", mode='w+')  # 写入文件
html.write(d.make_file(text1_lines, text2_lines))
html.close()

```

执行后在同级目录会生成一个 diff.html 文件，打开文件显示如下： ![image.png](https://i.loli.net/2020/04/01/fqySxAUlQ23nDi4.png)

## 2\. 文件目录差异对比方法

当我们进行代码审计或校验备份结果的时候，往往需要检查原始与目标目录的文件一致性，Python 的标准库 filecmp 可以满足此需求。filecmp 可以实现文件、目录、便利子目录的差异对比功能。

#### 模块常用方法说明

filecmp 提供了三个操作方法：

*   单文件对比（cmp），采用 filecmp.cmp(f1,f2\[,shallow\])比较两个文件，相同返回 True，不相同返回 False，shallow 默认为 True，意思只根据 os.stat()返回的文件基本信息进行对比，不对内容进行比较
*   多文件对比，采用 filecmp.cmpfiles(dir1,dir2,common\[,shallow\])方法，对比 dir1 与 dir2 目录给定的文件清单。该方法返回文件名的三个列表，分别为匹配、不匹配、错误。匹配为包含匹配的文件的列表，不匹配反之，错误列表包括了目录不存在文件，不具备读权限或其他原因导致不能比较的文件清单
*   目录对比，通过 dircmp（a,b\[,ignore\[,hide\]\]）类创建一个目录比较对象，其中 a 和 b 是参加比较的目录名。ignore 代表文件名忽略的列表，并默认为\['RCS','CVS','tags'\];hide 代表隐藏的列表。

下面写个例子实现目录差异对比功能，同时输出目录对比对象所有属性信息。

```python
import filecmp

a = "/Users/shenwenqiang/Desktop/dir1"  # 定义左目录，这里的目录路径需要写自己真实的路径
b = "/Users/shenwenqiang/Desktop/dir2"  # 定义右目录

dirobj = filecmp.dircmp(a, b, ['test.py'])  # 目录比较，忽略 test.py 文件
# 输出对比结果数据报表
dirobj.report()
dirobj.report_partial_closure()
dirobj.report_full_closure()

print("left_list:"+str(dirobj.left_list))
print("right_list:"+str(dirobj.right_list))
print("common:"+str(dirobj.common))
print("left_only:"+str(dirobj.left_only))
print("right_only:"+str(dirobj.right_only))
print("common_dirs"+str(dirobj.common_dirs))
print("common_files"+str(dirobj.common_files))
print("common_funny"+str(dirobj.common_funny))
print("same_file:"+str(dirobj.same_files))
print("diff_files:"+str(dirobj.diff_files))
print("funny_files:"+str(dirobj.funny_files))
```

两个文件夹目录结构为： ![image.png](https://i.loli.net/2020/04/01/ZsFSLMG7v6TKXql.png) 运行结果：

```
diff /Users/shenwenqiang/Desktop/dir1 /Users/shenwenqiang/Desktop/dir2
Only in /Users/shenwenqiang/Desktop/dir1 : ['1.txt', '111']
Only in /Users/shenwenqiang/Desktop/dir2 : ['1111', '123', '2.txt']
Differing files : ['.DS_Store']
diff /Users/shenwenqiang/Desktop/dir1 /Users/shenwenqiang/Desktop/dir2
Only in /Users/shenwenqiang/Desktop/dir1 : ['1.txt', '111']
Only in /Users/shenwenqiang/Desktop/dir2 : ['1111', '123', '2.txt']
Differing files : ['.DS_Store']
diff /Users/shenwenqiang/Desktop/dir1 /Users/shenwenqiang/Desktop/dir2
Only in /Users/shenwenqiang/Desktop/dir1 : ['1.txt', '111']
Only in /Users/shenwenqiang/Desktop/dir2 : ['1111', '123', '2.txt']
Differing files : ['.DS_Store']
left_list:['.DS_Store', '1.txt', '111']
right_list:['.DS_Store', '1111', '123', '2.txt']
common:['.DS_Store']
left_only:['1.txt', '111']
right_only:['1111', '123', '2.txt']
common_dirs[]
common_files['.DS_Store']
common_funny[]
same_file:[]
diff_files:['.DS_Store']
funny_files:[]


```

#### 校验源与备份目录差异

实现备份目录与源目录文件是否保持一致，包括源目录中的新文件或目录、更新文件或目录有无同步成功，定期进行校验，没有成功则希望有针对性地进行补备份

```python
import os, sys
import filecmp
import re
import shutil

holderlist = []


def compareme(dir1, dir2):  # 递归获取更新项函数
    dircomp = filecmp.dircmp(dir1, dir2)
    only_in_one = dircomp.left_only  # 源目录新文件或新目录
    diff_in_one = dircomp.diff_files  # 不匹配文件，源目录文件已发生变化
    dirpath = os.path.abspath(dir1)  # 定义源目录绝对路径
    # 将更新的文件名或目录追加到 holderlist
    [holderlist.append(os.path.abspath(os.path.join(dir1, x))) for x in only_in_one]
    [holderlist.append(os.path.abspath(os.path.join(dir1, x))) for x in diff_in_one]
    if len(dircomp.common_dirs) > 0:  # 判断是否存在相同子目录，以便递归
        for item in dircomp.common_dirs:  # 递归子目录
            compareme(os.path.abspath(os.path.join(dir1, item)), os.path.abspath(os.path.join(dir2, item)))
    return holderlist


def main():
    if len(sys.argv) > 2:  # 要求输入源目录和备份目录
        dir1 = sys.argv[1]
        dir2 = sys.argv[2]
    else:
        print("Usage:", sys.argv[0], "datadir backupdir")
        sys.exit()
    source_files = compareme(dir1, dir2)  # 对比源目录与备份目录
    dir1 = os.path.abspath(dir1)
    if not dir2.endwith('/'): dir2 = dir2 + '/'  # 备份目录路径加“/”
    dir2 = os.path.abspath(dir2)
    destination_files = []
    createdir_bool = False
    for item in source_files:  # 遍历返回的差异文件或目录清单
        destination_dir = re.sub(dir1, dir2, item)  # 将源目录差异路径清单对应替换成备份目录
        destination_files.append(destination_dir)
        if os.path.isdir(item):  # 如果差异路径为目录且不存在，则在备份目录中创建
            if not os.path.exists(destination_dir):
                os.makedirs(destination_dir)
                createdir_bool = True  # 再次调用 compareme 函数标记

    if createdir_bool:  # 重新调用 compareme 函数，重新遍历新创建目录的内容
        destination_files = []
        source_files = []
        source_files = compareme(dir1, dir2)  # 调用
        for item in source_files:  # 获取源目录差异路径清单，对应替换成备份目录
            destination_dir = re.sub(dir1, dir2, item)
            destination_files.append(destination_dir)
    print("update item:")
    print(source_files)  # 输出更新项列表清单
    copy_pair = zip(source_files, destination_files)  # 将源目录与备份目录文件清单拆分为元组
    for item in copy_pair:
        if os.path.isfile(item[0]):  # 判断是否为文件，是则进行复制操作
            shutil.copyfile(item[0], item[1])


if __name__ == '__main__':
    main()

```

下面进行测试，运行前源目录和备份目录结构如下 ![image.png](https://i.loli.net/2020/04/01/BYVex7yRn1O2KbP.png) 在命令行执行

```bash
 python3 filecmp02.py /Users/shenwenqiang/Desktop/datadir/  /Users/shenwenqiang/Desktop/backupdir/

```

第一次执行结果如下 ![image.png](https://i.loli.net/2020/04/01/zbiBvxywTmth9XG.png) 此时两个目录结构如下 ![image.png](https://i.loli.net/2020/04/01/dyXGQARqe9L2Drs.png) 再次执行则无差异文件： ![image.png](https://i.loli.net/2020/04/01/fshq67KmTjHxpcV.png) 说明同步成功，在运维工作中，可以将此脚本设置为定时任务，则可以定时同步文件