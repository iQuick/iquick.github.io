---
layout:		post
title:		妹子下载器
category:	扯淡
tags:		Python
published:	true
---
# 妹子下载器
---

# 一个妹子下载神器

<!--break-->

最近在学习 python ，闲的无事，于是自己写的了个妹子下载神器，妹子图地址：http://sexy.faceks.com 利用正则找出妹子图的链接，然后将图片分类下载到本地文件夹

```
#!/usr/bin/env python
# -*- coding: utf-8 -*- 

# Program:         妹子下载器
# Authro:          iquick
# History:         2015.8.12

import re
import os

try:
	from urllib import request
except Exception:
	import urllib2 as request

# 妹子图 Url
MEIZI_URL = r'http://sexy.faceks.com/?page=%s';
MEIZI_PATH = r'./'
MEIZI_DIR = r'meizi'
MEIZI_IS_GET_CHILD = True

# 获取妹子图 Url
def getMeiziUrl(page):
	return MEIZI_URL%page

# 获取 Http Get 请求结果
def getHttpData(url):
	return request.urlopen(url).read().decode('utf-8')

# 获取图片数据
def getImageData(url):
	return request.urlopen(url).read()

# 抓取妹子图片 Url
def findMeizituUrl(data):
	re_value = '<img src="(.*?)".*?/>'
	return findallData(re_value, data)

# 抓取妹子图片标题
def findMeizituTitle(data):
	re_value = '<div class="text"><p>(.*?)</p></div>'
	return findallData(re_value, data)

# 查找对应妹子图的子分类 url
def findMeizituChildUrl(data):
	re_value = r'<a class="img" href="(.*?)">'
	return findallData(re_value, data)

# 根据 re_value 查找
def findallData(re_value, data):
	return re.findall(re_value, data, re.S)

# 查找妹子图片
def findMeizituFeng(data):
	meizi_list = []
	meizi_title_list = findMeizituTitle(data)
	meizi_url_list = findMeizituUrl(data)
	meizi_child_list = findMeizituChildUrl(data)

	for i in range(0, len(meizi_title_list)):
		title = re.sub('<br />|&nbsp|;| ', '', meizi_title_list[i])
		url = meizi_url_list[i]
		child = meizi_child_list[i]

		# 获取子分类图
		if MEIZI_IS_GET_CHILD:
			print('Is grabbing the beauty pictures child', title)
			childData = getHttpData(child)
			childUrls = findMeizituChild(childData)

		# 添加到 meizi_list 中
		meizi_list.append(MeiZiFeng(title, url, childUrls))
	return meizi_list

# 查找封面下的妹子图
def findMeizituChild(data):
	return findMeizituUrl(data)

# 下载妹子图
def downloadMeizitu(meizis):
	for meizi in meizis:
		# 保存封面图片
		# print(meizi.url)
		downloadImage(meizi.url, meizi.title, MEIZI_PATH + MEIZI_DIR)
		# 保存子分类图片
		if MEIZI_IS_GET_CHILD:
			dirIsCreate(meizi.title, '%s%s/'%(MEIZI_PATH, MEIZI_DIR))
			for i, url in enumerate(meizi.childs):
				downloadImage(url, '%s%s'%(meizi.title, i), '%s%s/%s'%(MEIZI_PATH, MEIZI_DIR, meizi.title))

# 下载图片
def downloadImage(url, name, path):
	print('download image : %s'%name)
	# 保存图片
	with open('%s/%s.jpg'%(path, name), 'wb') as f:
		data = getImageData(url)
		f.write(data)
	
# 判断路径是否存在，如果不存在，则创建
def dirIsCreate(name, path):
	if not os.path.exists('%s%s'%(path, name)):
		os.mkdir('%s%s'%(path, name))

# 妹子封面
class MeiZiFeng(object):
	"""docstring for MeiZiFeng"""
	def __init__(self, title, url, childs):
		super(MeiZiFeng, self).__init__()
		self.title = title
		self.url = url
		self.childs = childs


# 输入开始页和结束页
startPage = input('please input crawler start page: ')
endPage = input('please input crawler end page: ')
# 转换成 int 
startPage = int(startPage)
endPage = int(endPage)

totalMeizi = []

# 抓取所有的妹子图片信息
for i in range(startPage, endPage + 1):
	print('Is grabbing the %s page beauty pictures'%i)
	data = getHttpData(getMeiziUrl(i))
	meiziList = findMeizituFeng(data)
	for meizi in meiziList:
		totalMeizi.append(meizi)

# 判断目录是否存在
dirIsCreate(MEIZI_DIR, MEIZI_PATH)

# 下载妹子图
downloadMeizitu(totalMeizi)

print('Download complete the meizitu! Thank you for your use.')
```