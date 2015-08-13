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

<pre class="prettyprint linenums">
#!/usr/bin/env python
# -*- coding: utf-8 -*- 

import re
from urllib import request

# 获取 Http Get 请求结果
def getHttpData(url):
	with request.urlopen(url) as f:
		data = f.read().decode('gbk')
	return data

# 查找省 Url
def findAreaUrl(data, area):
	re_value = r'<a href="[\S]+" rel="[\S]+">%s</a>'%area
	url_value = re.search(re_value, data).group()
	url = re.search('http://[\w\./]+\.htm', url_value).group()
	return url

# 查找市 Url
def findCityUrl(data, city):
	re_value = r'<a href="[\S]+?" title="[\S]+?">%s</a>'%city
	url_value = re.search(re_value, data).group()
	url = re.search('/[\S]+?\.htm', url_value).group()
	return 'http://tianqi.2345.com' + url

# 查找所有省份
def finAllArea(data):
	re_value = r'<a href="[\S]+" rel="[\S]+">([\S]+)</a>'
	area_list = re.findall(re_value, data)
	return area_list

# 查找省下所有城市
def findAllCity(data):
	re_value = r'<dt><a href="[\S]+?" title="[\S]+?">([\S]+)天气</a></dt>'
	city_list = re.findall(re_value, data)
	return city_list

# 获取天气信息
def getWeatherInfo(data):
	re_value = r'<li class="week-detail-now" >[\s\S]+?</li>'
	weather_list = re.findall(re_value, data)
	return weather_list


# 格式化天气信息
def makeWeatherInfo(data):
	dA_re = r'[\d]{2}月[\d]{2}日'
	dA = re.search(dA_re, data).group()

	dT_re = r'<b><font class="gray">白天：</font>.{1,6}</b>'
	dT = re.search(dT_re, data).group()
	dT = re.sub(r'<b>.+</font>', '', dT)
	dT = re.sub(r'</b>', '', dT)

	nT_re = r'<b><font class="gray">夜间：</font>.{1,6}</b>'
	nT = re.search(nT_re, data).group()
	nT = re.sub(r'<b>.+</font>', '', nT)
	nT = re.sub(r'</b>', '', nT)

	t_re = r'<font class="blue">.{0,4}</font>～<font class="red">.{0,4}</font>'
	t = re.search(t_re, data).group()
	t = re.findall(r'[\d]+', t)
	return WeatherModel(dA, dT, nT, t[0], t[1])

# 天气
class WeatherModel(object):
	"""docstring for WeatherModel"""
	def __init__(self, date, daytime, nighttime, temL, temH):
		super(WeatherModel, self).__init__()
		self.date = date
		self.daytime = daytime
		self.nighttime = nighttime
		self.temL = temL
		self.temH = temH

	def print(self):
		print('\n%s：白天：%s，夜间：%s，\n最低温度：%sC，最高温度：%sC\n'%(self.date,self.daytime,self.nighttime,self.temL,self.temH))
		

homePageUrl = r'http://tianqi.2345.com/';
homePageData = getHttpData(homePageUrl)
# print(homePageData)

# 打印省份信息
areaList = finAllArea(homePageData)
for area in areaList:
	print(area)

# 输入要查询的省份
area = input('please input area: ')
areaUrl = findAreaUrl(homePageData, area)
print(areaUrl)

areaData = getHttpData(areaUrl)
# print(areaData)

# 打印城市信息
cityList = findAllCity(areaData)
for city in cityList:
	print(city)

# 输入要查询的城市
city = input('please input city: ')
cityUrl = findCityUrl(areaData, city)
# print(cityUrl)

weatherData = getHttpData(cityUrl)
# print(weatherData)

weatherList = getWeatherInfo(weatherData)
for weather in weatherList:
	# print(weather)
	makeWeatherInfo(weather).print()
</pre>