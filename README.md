# fudan

---
title: 复旦大学计算机拟录取名单分析
date: 2020-07-18 19:16:36
cover: https://mryanhehe-1300112970.cos.ap-chengdu.myqcloud.com/img/pexels-photo-1850137.jpeg
categories: 数据分析
tags:
- 复旦大学
- 数据分析

---

# 复旦大学计算机拟录取名单分析

**复旦大学是考研的目标院校，最近名单下来了，决定对名单进行一下数据分析**。

## 第一步：信息获取

复旦大学的信息获取，我是自己写了一个脚本，源代码再次。该脚本可以发现复旦大学新出的消息，并向邮件发送，保证自己掌握第一手消息。

话不多说，直接上数据。[复旦大学拟录取名单](http://www.gsao.fudan.edu.cn/b4/08/c15014a242696/page.htm)。

下载官网数据。

## 第二步：数据文件处理

下载的文件是pdf，所以我们先将pdf中的文字提取出来，利用pandas库分析。

我才用的库是pdfplumber。该库是一个专门用于pdf数据分析的库，识别率还比较满意。

先将每页的数据转成DataFrame。（我们只需要计算机的，因此只提取了20到40页）。

```python
import pdfplumber
import pandas as pd
pages = []
df_list = []
with pdfplumber.open("复旦.pdf") as pdf:
    for i in range(20,40):
            pages.append(pdf.pages[i])
    for page in pages:
        for table in page.extract_tables(): 
            #得到的table是嵌套list类型，转化成DataFrame更加方便查看和分析 
            df_list.append(pd.DataFrame(table[1:], columns=table[0]))

```

pdf文件的索引一般在第一页，我们提取的是20到40页，因此每页的所以都是第一行，在之后的合并中会产生bug，因此我们先把每个DataFrame的索引改一下。

```python
for df in df_list:
    df.columns=['考生编号','姓名','院系','初始成绩',"复试成绩",'总成绩','备注'] 
```

现在就可以将每页的DataFrame合并了。

```python
df = pd.concat(df_list,axis=0,sort = False)
df
df_list[0]
```

## 第三步：数据过滤及处理

筛选出院系是计算机科学技术学院的名单。

```python
computer = df.loc[df['院系'] == '计算机科学技术学院']
df.count()
```

![](https://mryanhehe-1300112970.cos.ap-chengdu.myqcloud.com/img/微信截图_20200718192656.png)

对成绩等信息进行类型转换

```python
computer[['初始成绩',"复试成绩",'总成绩']] = computer[['初始成绩',"复试成绩",'总成绩']].apply(pd.to_numeric)
computer.dtypes
```

![](https://mryanhehe-1300112970.cos.ap-chengdu.myqcloud.com/img/微信截图_20200718194728.png)

对成绩等信息进行分析

```python
computer[['初始成绩',"复试成绩",'总成绩']].describe()
```

![](https://mryanhehe-1300112970.cos.ap-chengdu.myqcloud.com/img/微信截图_20200718194839.png)

果然如对面所说，去年复试线310，刚好有一个人考了310，于是好奇去看了一下他的信息。

```python
computer.loc[computer['初始成绩'] == 310]
```

![](https://mryanhehe-1300112970.cos.ap-chengdu.myqcloud.com/img/微信截图_20200718195126.png)

emmm,这个人的复试成绩这么高。。。难怪可以310进去。。。。。

对初试成绩，复试成绩，总成绩进行分数段统计

```python
computer[['初始成绩',"复试成绩",'总成绩']] = computer[['初始成绩',"复试成绩",'总成绩']].apply(pd.to_numeric)
res = pd.cut(computer['初始成绩'] , [i for i in range(310,420,10)],right = False,labels= [i for i in range(310,410,10)])
computer['初始分数段'] = res


res = pd.cut(computer['复试成绩'] ,[i for i in range(60,100,5)],right = False,labels= [i for i in range(60,95,5)])
list(res)
computer['复试分数段'] = res

res = pd.cut(computer['总成绩'],[i for i in range(60,90,5)],right = False ,labels= [i for i in range(60,85,5)])
list(res)
computer['总成绩分数段'] = res
```



![](https://mryanhehe-1300112970.cos.ap-chengdu.myqcloud.com/img/微信截图_20200718211843.png)



## 数据可视化

**初始分数段柱状图**

![](https://mryanhehe-1300112970.cos.ap-chengdu.myqcloud.com/img/下载 (2).png)

**初始分数段饼图**

![](https://mryanhehe-1300112970.cos.ap-chengdu.myqcloud.com/img/下载 (11).png)

**复试分数段柱状图**

![](https://mryanhehe-1300112970.cos.ap-chengdu.myqcloud.com/img/下载.png)

**复试分数段饼图**

![](https://mryanhehe-1300112970.cos.ap-chengdu.myqcloud.com/img/下载 (8).png)

**总成绩分数段柱状图**

![](https://mryanhehe-1300112970.cos.ap-chengdu.myqcloud.com/img/下载 (6).png)

**总成绩分数段饼图**

![](https://mryanhehe-1300112970.cos.ap-chengdu.myqcloud.com/img/下载 (9).png)

最终成绩，大部分都是在70-80之间，根据复试和总成绩来看，不难看到复试的相对给分还是比较高的。得益于今年的扩招，复试的分数线有50%的人都在360以下。这个上岸的几率还是蛮大的。

**总成绩前20和倒数20的对比**

```python
min_stu = computer.sort_values(['总成绩']).iloc[:20]
max_stu = computer.sort_values(['总成绩'],ascending=[False]).iloc[:20]

plt.scatter(max_stu['初始成绩'],max_stu['复试成绩'],color='green', label='总成绩前20名')
plt.scatter(min_stu['初始成绩'],max_stu['复试成绩'],color='blue', label='总成绩后20名')



plt.legend()
plt.show()
```

<img src="https://mryanhehe-1300112970.cos.ap-chengdu.myqcloud.com/img/下载 (17).png" style="zoom:150%;" />



<img src="https://mryanhehe-1300112970.cos.ap-chengdu.myqcloud.com/img/下载 (18).png" style="zoom:150%;" />

hhh，让我想起了向量机，有一条很明显的线将两者隔开，不过我们还是初见端倪，复旦的总成绩还是主要看重初试成绩，并且，前50的人一般分数线都在360以上哦。





看完了总成绩的前20，来看看初试成绩的前20吧。

**初试成绩前20和倒数20对比**

```python
min_stu = computer.sort_values(['初始成绩']).iloc[:20]
max_stu = computer.sort_values(['初始成绩'],ascending=[False]).iloc[:20]

plt.scatter(max_stu['初始成绩'],max_stu['复试成绩'],color='green', label='总成绩前20名')
plt.scatter(min_stu['初始成绩'],max_stu['复试成绩'],color='blue', label='总成绩后20名')
plt.xlabel("初始成绩",{"size":15})
plt.ylabel("复试成绩",{"size":15})

plt.legend()
plt.show()
```

<img src="https://mryanhehe-1300112970.cos.ap-chengdu.myqcloud.com/img/下载 (14).png" style="zoom:150%;" />

emmmmm，从图中并看不出复试和初试成绩的关系。。。成绩高的也是复试很低，成绩低的也有复试很高的，而且每个复试分数段的人数都差不多？



## 总结

这次数据分析，数据并不多，相关量也还好，主要是了解一下复旦考研的基本情况。说实话，我有点酸额，感觉这个分数也太香了吧。50%的人复试都在360以下。。。感觉比北京那边竞争小了好多好多。希望今年复旦能对我好一点。

另外，附上源码和数据