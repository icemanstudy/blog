---
title: python3操作excel文件
date: 2018-02-02 10:01:32
categories:
- python
tags:
index_img:
- /img/post/20.jpg
---
前段时间遇到一个需求,要从一个有7000多行的excel文件中读取数据.第一反应是python应该能很方便的解析出需要的数据.

### 1.python读excel.

有个叫xlrd的模块.使用pip install xlrd安装即可

打开一个excel文件:

data = xlrd.open_workbook(‘xxx.xls’)
打开其中一张工作表
table = data.sheets()[0] //第一张表

table = data.sheet_by_index(0) //也是第一张表

table = data.sheet_by_name(‘Sheet1’) //根据名字获取表
得到表的行数和列表
table.nrows

table.ncols
得到某行数据或者某列数据,会以数组返回
table.row_values(i)
table.col_values(i)
于是可以这样打印整张表内容
for i in range(table.nrows):

print(table.row_values(i))
7000多行瞬间完成.碉堡了.

获取某个单元格内容,第一个参数是行,第二个为列

table.cell(0, 1)
打印得知里面包含单元格的格式,比如’’number’’,”text”,还有单元格的数据

直接得到单元格数据可以用

table.cell(0, 1).value
比如某个单元格里面写的”省”

我调用如下代码

print(table.cell(0, 1))

print(table.cell(0, 1).value)
显示
text:’省’

省
根据行数,列数返回excel中的单元格名字:
print(xlrd.cellname(0, 0)) //返回 A1
xlrd模块基本上就到这里了.

虽然网上有这样的方法:

table.put_cell(13, 5, 0, ‘哈哈’, 0) //写入数据

print(table.cell(13, 5)) //打印数据,这里确实打印的”哈哈”出来

data.save(‘xxx.xls’) //运行不会报错,但实际上是没有效果的
查资料得知:xlrd是不能修改文件的,前面调用open_workbook返回的是只读的.

### 2.python创建excel文件,并写入数据
对应的模块是xlwt,可惜这个模块是对应python2.x的.还好提供了python3.x的版本.xlwt3.

使用pip install xlwt3安装

注意:如果代码运行时出现以下问题:

ValueError: ‘init‘ in slots conflicts with class variable

需要修改一下已安装的xlwt3的代码:

打开Python33Libsite-packagesxlwt3formula.py文件，将其中的

slots = [“init“, “s”, “parser”, “sheet_refs”, “xcall_refs”]

修改为

slots = [ “s”, “parser”, “sheet_refs”, “xcall_refs”]
创建一个excel文件
wb = xlwt3.Workbook();
建立一张表
ws = wb.add_sheet(‘A Test Sheet’, cell_overwrite_ok=True);
这个cell_overwrite_ok参数比较重要,设定为True的时候,才允许后面的代码中对同一个单元格进行反复写入.

对单元格进行数据写入

ws.write(13, 5, 555555);
写完保存,记得名字如果与现有的文件一样,就直接覆盖了.(悲剧极易发生)
wb.save(‘example.xls’);
自定义格式,这里我只试验了定义字体样式.
style = xlwt3.XFStyle()

font = xlwt3.Font()

font.name = ‘Times New Roman’ //使用Times New Roman字体

font.bold = True //设置为黑体

style.font = font
然后写入的时候,加上style参数就行了.
ws.write(13, 5, 555555, style);

### 3.python修改现有excel文件

使用openpyxl进行xlsx文件的读写编辑操作.python3.x可用.
