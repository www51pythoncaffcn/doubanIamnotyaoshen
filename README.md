# doubanIamnotyaoshen
#!/usr/bin/env python3
# -*- coding: utf-8-*-

import requests

from bs4 import BeautifulSoup

import numpy

import time

import pymysql

import traceback

import pandas

import chardet

import pyecharts

import matplotlib.pyplot as plt

from wordcloud import WordCloud

from pyecharts import Pie

import pymysql

from pyecharts import Bar

import jieba


# 第一步：请求豆瓣服务器，获取数据。

# 第二步：对请求回来的数据解析

# 第三步：展示解析后的数据


# 构建请求头部

headers = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36"}


# 获取豆瓣电影数据

def data(page):

    list2 = []

    list1 = []

    url = "https://movie.douban.com/subject/26752088/reviews?start={}".format(page*20)

    res = requests.get(url, headers)

    res.encoding = "utf-8"

    soul = BeautifulSoup(res.text, "html.parser")

    for i in range(0, 20):

        nickname = soul.find_all(class_="name")[i].string

        xingji=soul.find_all(class_="main-hd")[i].contents[5].attrs.get("title")

        timedate = soul.find_all(class_="main-meta")[i].string

        content = soul.find_all(class_="short-content")[i].contents[2].strip()

        support = soul.find_all(class_="action-btn up")[i].contents[1].contents[1].string.strip()

        if support=='':

            support='0'

        aginst = soul.find_all(class_="action-btn down")[i].contents[3].string.strip()

        if aginst=='':

            aginst='0'

        question = soul.find_all(class_="reply")[i].string.strip()

        image = soul.find_all(class_="avator")[i].contents[1].attrs.get("src")

        list2.append(nickname)

        list2.append(xingji)

        list2.append(timedate)

        list2.append(content)

        list2.append(support)

        list2.append(aginst)

        list2.append(question)

        list2.append(image)

        list1.append(list2)

        list2 = []

    if (res.status_code == 200):

        print("第{}页爬取成功..........................................................................".format(page + 1))


    else:

        print("第{}页爬取失败..........................................................................".format(page + 1))

    sava(list1)


# 解析数据

def sava( list11 ):


    db = pymysql.connect(

        host="localhost",
        user="root",

        password="rootroot",

        db="moc",

        charset='utf8'

    )

    cursor1 = db.cursor();

    for m in list11:

        try:

            sql = "INSERT INTO ysmovies (nickname,xingji,timedate1,content,support,aginst,question,image)VALUES('%s','%s','%s','%s',%s,%s,'%s','%s')" % (m[0], m[1], m[2], m[3], m[4], m[5], m[6], m[7])
            # 执行sql语句
            cursor1.execute(sql)

            # 提交到数据库执行
            db.commit()

        except:

            # 如果发生错误则回滚
            db.rollback()

            traceback.print_exc()

            # print(sql)

    # 关闭数据库连接

    db.close()


# 将数据存储到数据库后，下一步就开始对数据进行分析了。

# 数据分析的角度：基于目标用户场景需求，对内容数据涵盖的字段，进行分析

# 分析点评数据

def pJ():

# 执行查询语句

    sql="select xingji,count(xingji) as f from ysmovies group by xingji"

    cursor1.execute(sql)

    # 获取元组数据

    data=cursor1.fetchall()

    lijian=data[0][1]

    tuijian=data[1][1]

    haixing=data[2][1]

    jiaocha=data[3][1]

    hencha=data[4][1]

    print(lijian,tuijian,haixing,jiaocha,hencha)

    attr=["力荐","推荐","还行","较差","很差"]

    v1=[lijian,tuijian,haixing,jiaocha,hencha]

    pie=Pie("我不是药神评价比例")

    pie.add("",attr,v1,is_label_show=True)

    pie.show_config()

    pie.render()


def dj():

    sql1="select support,nickname from ysmovies ORDER BY support+0 DESC LIMIT 10"

    cursor1.execute(sql1)

    data1=cursor1.fetchall()

    f0=data1[0][0]
    f1 = data1[1][0]
    f2 = data1[2][0]
    f3 = data1[3][0]
    f4 = data1[4][0]
    f5 = data1[5][0]
    f6= data1[6][0]
    f7 = data1[7][0]
    f8 = data1[8][0]
    f9=data1[9][0]

    attr1=[data1[0][1],data1[1][1],data1[2][1],data1[3][1],data1[4][1],data1[5][1],data1[6][1],data1[7][1],data1[8][1],data1[9][1]]


    v1=[f0,f1,f2,f3,f4,f5,f6,f7,f8,f9]


    bar=Bar("点赞数最多的评论")

    bar.add(" ",attr1,v1,is_label_show=True,is_label_emphasis=True,is_area_show=True)

    bar.show_config()

    bar.render()

dj();

def wordc():


        sql2="select content from ysmovies WHERE content is not NULL and content !=')'"


        cursor1.execute(sql2)

        data2=cursor1.fetchall()

        for t in data2:

               with open("wordcloud.txt","a",encoding="utf-8") as f:

                        f.write(t[0])

        f1=open('wordcloud.txt','r',encoding="utf-8").read()

        cut=jieba.cut(f1)

        string=' '.join(cut)

        font="simhei.ttf"

        wordcloud = WordCloud (background_color="white", width=1000, height=860, font_path=font,margin=2).generate (string)

        plt.imshow (wordcloud,interpolation="bilinear")

        plt.figure()

        plt.axis ("off")


        wordcloud.to_file ('test.png')


wordc()

# 设置公共模块，以便其他模块调用

# def commonMoudle():

if __name__ == '__main__':

    for page in range(0, 697):

        data(page)

        time.sleep(1)
