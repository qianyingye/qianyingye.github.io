---
title: 'Weibo User Profile Spider'
date: 2023-03-15
permalink: /posts/2012/08/blog-post-4/
tags:
  - Python
  - Weibo
  - Spider
---

This is a reference of Weibo user profile data spider.
# 使用python爬取微博用户简历信息
1.变量包括：性别，阳光信用，生日（星座），注册日期，简介，是否认证，认证信息，粉丝数，关注数，微博数，svip等级。
2.基于GitHub用户inspurer的代码补充修改。
3.代码仅供参考，请在法律允许范围内使用。
```python
import requests
import pandas as pd
from time import sleep
import time
import json

headers = {
    'authority': 'weibo.com',
    'x-requested-with': 'XMLHttpRequest',
    'sec-ch-ua-mobile': '?0',
    'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36',
    'content-type': 'application/x-www-form-urlencoded',
    'accept': '*/*',
    'sec-fetch-site': 'same-origin',
    'sec-fetch-mode': 'cors',
    'sec-fetch-dest': 'empty',
    'referer': 'https://weibo.com/1192329374/KnnG78Yf3?filter=hot&root_comment_id=0&type=comment',
    'accept-language': 'zh-CN,zh;q=0.9,en-CN;q=0.8,en;q=0.7,es-MX;q=0.6,es;q=0.5',
    'cookie': '请输入自己的cookie'
}

def parseUid(uid):
    response = requests.get(url=f'https://weibo.com/ajax/profile/info?custom={uid}', headers=headers)
    try:
        return response.json()['data']['user']['id']
    except:
        return None

def getUserInfo(uid,f1):
    try:
        uid = int(uid)
        if len(uid) != '10': ####也有一些数字生成的
            uid = parseUid(uid)
    except:
        # 说明是英文串
        uid = parseUid(uid)
        if not uid:
            return None
    #############################获取阳光信用等级，教育，学校，性别， 注册日期，个人简介，我关注的人中有多少人关注 ta
    response1 = requests.get(url=f'https://weibo.com/ajax/profile/detail?uid={uid}', headers=headers)
    if response1.status_code == 400:
        f1.write(str(uid)+'\n')
        
    resp_json1 = response1.json().get('data', None)
    if not resp_json1:
        return None
    #print(resp_json1)
    sunshine_credit = resp_json1.get('sunshine_credit', None)
    if sunshine_credit:
        sunshine_credit_level = sunshine_credit.get('level', None)
    else:
        sunshine_credit_level = None
        
    education = resp_json1.get('education', None)
    if education:
        school = education.get('school', None)
    else:
        school = None

    location = resp_json1.get('location', None)
    gender = resp_json1.get('gender', None)
    birthday = resp_json1.get('birthday', None)
    created_at = resp_json1.get('created_at', None)
    description = resp_json1.get('description', None)
    # 我关注的人中有多少人关注 ta
    followers = resp_json1.get('followers', None)
    if followers:
        followers_num = followers.get('total_number', None)
    else:
        followers_num = None
    
    verified_url = resp_json1.get('verified_url', None)
    #title = resp_json1.get('title', None)
    #############################获取是否个人认证，认证信息，粉丝，关注人，微博，vip等级，粉丝数的整数 
    #time.sleep(1.7)
    response2 = requests.get(url=f'https://weibo.com/ajax/profile/info?uid={uid}', headers=headers)
    if response2.status_code == 400:
            f1.write(str(uid)+'\n')
    resp_json2 = response2.json().get('data', None)
    user = resp_json2.get('user')
    #print(user)
    
    if not resp_json2:
        return None
    
    #############
    verified = user.get('verified', None)
    verified_reason = user.get('verified_reason', None)
    followers_count = user.get('followers_count',None)
    friends_count = user.get('friends_count',None)
    statuses_count = user.get('statuses_count',None)
    svip = user.get('svip',None)
    followers_count_str = user.get('followers_count_str',None)
    uid = user.get('id',None)
    screen_name = user.get('screen_name',None)
    #############写入到txt
    f1.write(str(screen_name) +'\t'+ str(url)+'\t'+str(uid)+'\t'+ str(gender)+'\t'+str(sunshine_credit_level)+'\t'+ str(school)+'\t'+str(location) +'\t'+ 
             str(birthday)+'\t'+str(created_at)+'\t'+ str(description) +'\t'+ str(followers_num) + '\t'+ str(verified_url) +'\t'+ str(verified)+'\t'+
             str(verified_reason) +'\t'+str(followers_count)+'\t'+str(friends_count)+'\t'+
             str(statuses_count)+ '\t'+ str(svip)+'\t'+str(followers_count_str)+'\n')

f = open("url.txt")
f1 = open("weibo_user_data.txt",'a',encoding='utf-8')
f1.write('screen_name' +'\t'+ 'url'+'\t'+ 'uid'+'\t'+ 'gender'+'\t'+ 'sunshine_credit_level'+'\t'+ 'school'+'\t'+'location' +'\t'+ 'birthday'+'\t'+'created_at'
        '\t'+ 'description' +'\t'+ 'followers_num' + '\t'+ 'verified_url' +'\t'+ 'verified'+'\t'+
         'verified_reason' +'\t'+'followers_count'+'\t'+'friends_count'+'\t'+
         'statuses_count'+ '\t'+ 'svip'+'\t'+'followers_count_str'+'\n')
for line in f:
    url = line.replace("\n","").replace("https://weibo.com/","").replace("u/","")
    getUserInfo(url,f1)
f1.close()

#reference：本文基于GitHub用户‘inspurer‘的代码修改：https://github.com/Python3Spiders/WeiboSuperSpider/blob/master/无%20GUI%20功能独立版/WeiboUserInfoSpider.py
```
