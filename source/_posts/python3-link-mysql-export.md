---
title: 利用Python3 连接mysql 导出数据为excel格式
date: 2018-07-09
category: "python3"
tags: [python,mysql]
---
## 背景
最近在工作中，遇到每隔一段时间都要跑一次数据校验。大概有15个sql语句，需要将这15个sql语句查询的记录导出并且存为excel格式。
才开始觉得sql每次都是固定不变，总量也不多，就通过 MySQL workbench 一条一条查询导出为 *.csv文件，但是发现 *.csv文件直接打开会出现中文乱码，就需要转码，我就通过Notepad++ 打开csv文件，设置文件格式 utf-8 bom 编码。这一系列步骤下来。人都不好了。
后面自己受不了了，就想通过脚本来解决这个重复性工作。我是一个Java开发者，就准备用Java来实现这个工作，但是后面发现Java来解决这个问题，有点太麻烦了，需要连接MySQL，查询数据，通过poi遍历做成poi对应excel的hssfworkbook对象（或者xssfworkbook），然后通过io流输出成excel文件。想到这些，就直接放弃了用Java来实现这个工作了。
后面想到了Python，我对Python不熟。就在网上查找了一些资料，发现还是满简单的了。就这样决定用Python来实现了。
## 具体实现
下面是Python的具体实现代码：

```python
# coding:utf8
import sys
import xlwt
#import MySQLdb
import pymysql as MySQLdb
import datetime

def get_conn_mysql():
    conn = MySQLdb.connect(host='localhost', port=3306, user='root', passwd='root', db='test', charset='utf8')
    return conn
def query_data(cur, sql, args):
    cur.execute(sql, args)
    return cur.fetchall()

# out_path 保存的路径， sql： 需要执行的语句
def read_mysql_to_excel(out_path, sql):
    conn = get_conn_mysql()
    cursor = conn.cursor()
    count = cursor.execute(sql)
    if count != 0:
        print(count)
        cursor.scroll(0, mode='absolute')
        results = cursor.fetchall()
        fields = cursor.description
        workbook = xlwt.Workbook()
        sheet = workbook.add_sheet('building', cell_overwrite_ok=True)

        for field in range(0, len(fields)):
            sheet.write(0, field, fields[field][0])

        row = 1
        col = 0
        for row in range(1, len(results)+1):
            for col in range(0, len(fields)):
                if results[row-1][col] != None:
                    sheet.write(row, col, u'%s'%results[row-1][col])
                else:
                    val = ''
                    sheet.write(row, col, u'%s'%val)
        workbook.save(out_path)
if __name__ == '__main__':
	# 执行的sql语句
	sql = 'select * from sys_user'
	read_mysql_to_excel('1.xls', sql)
```

* 就这样一个简单的 导出数据为excel格式的demo就完成了。
* 如果有多个sql语句，只需要 hard code，在分别调用那个 `read_mysql_to_excel` 方法就好了。

参考资料：[https://blog.csdn.net/tingzuhuitou/article/details/78749185](https://blog.csdn.net/tingzuhuitou/article/details/78749185 "https://blog.csdn.net/tingzuhuitou/article/details/78749185")
