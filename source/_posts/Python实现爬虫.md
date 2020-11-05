title: Python实现爬虫
author: Wtli
tags: []
categories:
  - Python
date: 2020-11-03 08:21:00
---
爬虫学的好，牢饭吃得饱～～

使用Python实现爬虫，首先根据网上项目入门简单爬虫，然后爬取山东省各地市政府招标网站的信息。
<!--more-->

### 简单爬虫

运行主函数：
```
def main():
    for i in range(1, 14):
        url = 'https://qiushibaike.com/text/page/{}'.format(i)
        html = download_page(url)
        get_content(html, i)


if __name__ == '__main__':
    main()

```

抓取糗事百科的文字，设置url，将网页下载下来，然后处理下载下来的网页。
<font color="gray">这里使用到requests和BeautifulSoup包，如果没有的话请使用pip install安装</font>

```
import requests
from bs4 import BeautifulSoup

def download_page(url):
    headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:61.0) Gecko/20100101 Firefox/61.0"}
    r = requests.get(url, headers=headers)
    return r.text
```
这里的到的r是指的Response网页请求的返回信息,r.text指的就是html代码：
```
<!DOCTYPE html
PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<meta http-equiv="X-UA-Compatible" content="chrome=1,IE=edge">
<meta name="renderer" content="webkit" />
<meta name="applicable-device" content="pc">

······

<a href="/article/123739738" target="_blank" class="contentHerf" onclick="_hmt.push(['_trackEvent','web-list-content','chick'])">
<div class="content">
<span>


病房里有个大叔在我们科室住过几次院，科室里的护士他基本上都能认出来！<br/>接班的时候他发现是我值班，就跟隔壁床的说他血管不好找，每次都要找我扎 针，说我扎 针技术就是好，最后差点吹嘘到我就是闭着眼睛就能扎上！<br/>隔壁床听着有点不信他，他就有点不高兴，立马叫上我，非让我把手上的针给拔了重新扎一下。<br/>我劝他别这样，手上的留置针其实还能用，他非不听，后来我没办法，在他期待的眼神里给他扎了两下才扎上！<br/>他望着隔壁床尴尬的说，可能晚上光线不好才会扎两下的！

</span>
```

接下来是处理网页html数据，将有效的信息摘选出来：

1. 使用BeautifulSoup，将html转换成soup；
2. 在网页中查看源代码， 看到想要的数据在content标签下，调用soup.find方法，找到\<div id="content"\>标签下的数据；
3. 找到文章列表con.find_all('div', class_="article")，在列表中遍历。
4. 遍历每条数据，调用save_txt方法，保存到txt中。

```
def get_content(html, page):
    output = "第{}页 作者：{} 性别：{} 年龄：{} 点赞：{} 评论：{}\n{}\n------------\n"
    soup = BeautifulSoup(html, 'html.parser')
    con = soup.find(id='content')
    con_list = con.find_all('div', class_="article")
    for i in con_list:
        author = i.find('h2').string  # 获取作者名字
        content = i.find('div', class_='content').find('span').get_text()  # 获取内容
        stats = i.find('div', class_='stats')
        vote = stats.find('span', class_='stats-vote').find('i', class_='number').string
        comment = stats.find('span', class_='stats-comments').find('i', class_='number').string
        author_info = i.find('div', class_='articleGender')  # 获取作者 年龄，性别
        if author_info is not None:  # 非匿名用户
            class_list = author_info['class']
            if "womenIcon" in class_list:
                gender = '女'
            elif "manIcon" in class_list:
                gender = '男'
            else:
                gender = ''
            age = author_info.string  # 获取年龄
        else:  # 匿名用户
            gender = ''
            age = ''

        save_txt(output.format(page, author, gender, age, vote, comment, content))
        
        
def save_txt(*args):
    for i in args:
        with open('qiubai.txt', 'a', encoding='utf-8') as f:
            f.write(i)        
```

输出：
```
第1页 作者：
夏有凉风秋望月
 性别：女 年龄：16 点赞：846 评论：26



小时候，对三国是很迷恋的，对于杀又鸟敬猴这招阳谋很是推崇，平时没少对小伙伴们灌输这个思想。由于我们这伙小伙子不太爱学习，老师被班主任罚站，于是我们决定杀又鸟敬候，于是我们伙在一起蹭着月黑风高把校长办公室的玻璃给砸了，还自报了家门，连校长我们都敢招惹看你班主任敢拿我们怎么样。果然杀又鸟儆猴凑效了，校长和班主任让我们这些小又鸟，让我们在全校的猴面前做检讨和保证。。。。


------------
第1页 作者：
无书斋主
 性别：男 年龄：42 点赞：1066 评论：38



本人糗百上用的陈道明图像，觉得他不但温文尔雅还有真性情……昨天外地出差，一糗友特意赶过来一起聚聚，他还带了两个朋友！……打招呼寒暄三杯酒下肚后，这糗友的朋友:他不是无书吧，无书不是长这样吗？（他指着我糗百图像）……我……我是，你说的是明哥！……


------------
```
### 爬虫获取拉钩网数据

先发代码：
```
import requests
import time
import json


def main():
    url_start = "https://www.lagou.com/jobs/list_运维?city=%E6%88%90%E9%83%BD&cl=false&fromSearch=true&labelWords=&suginput="
    url_parse = "https://www.lagou.com/jobs/positionAjax.json?city=成都&needAddtionalResult=false"
    headers = {
        'Accept': 'application/json, text/javascript, */*; q=0.01',
        'Referer': 'https://www.lagou.com/jobs/list_%E8%BF%90%E7%BB%B4?city=%E6%88%90%E9%83%BD&cl=false&fromSearch=true&labelWords=&suginput=',
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36'
    }
    for x in range(1, 5):
        data = {
            'first': 'true',
            'pn': str(x),
            'kd': '运维'
                }
        s = requests.Session() # 创建一个session对象
        s.get(url_start, headers=headers, timeout=3)  # 用session对象发出get请求，请求首页获取cookies
        cookie = s.cookies  # 为此次获取的cookies
        response = s.post(url_parse, data=data, headers=headers, cookies=cookie, timeout=3)  # 获取此次文本
        time.sleep(5)
        response.encoding = response.apparent_encoding
        text = json.loads(response.text)
        info = text["content"]["positionResult"]["result"]
        for i in info:
            print(i["companyFullName"])
            companyFullName = i["companyFullName"]
            print(i["positionName"])
            positionName = i["positionName"]
            print(i["salary"])
            salary = i["salary"]
            print(i["companySize"])
            companySize = i["companySize"]
            print(i["skillLables"])
            skillLables = i["skillLables"]
            print(i["createTime"])
            createTime = i["createTime"]
            print(i["district"])
            district = i["district"]
            print(i["stationname"])
            stationname = i["stationname"]

if __name__ == '__main__':
    main()

```
有很多写法：
发现返回结果为访问太频繁，可以认为拉勾网做了一定的反扒技术，下面在慢慢分析一下拉勾网的请求策略：

参考网上说的可能性
1. 拉勾网对cookie动了手脚，cookie具有实时性，只能使用一次。
2. 拉勾网对获取数据请求之前还有获取cookie的前置请求。


#### 获取ajax请求

```
    json = requests.post(url, data, headers=headers).json()
```



#### 写入到excel中

```
from openpyxl import Workbook

wb = Workbook()
wb.save('{}职位信息.xlsx'.format(lang_name))
```





#### 写入到数据库中

```
import pymysql.cursors

def get_conn():
    '''建立数据库连接'''
    conn = pymysql.connect(host='localhost',
                           user='root',
                           password='root',
                           db='python',
                           charset='utf8mb4',
                           cursorclass=pymysql.cursors.DictCursor)
    return conn


def insert(conn, info):
    '''数据写入数据库'''
    with conn.cursor() as cursor:
        sql = "INSERT INTO `python` (`shortname`, `fullname`, `industryfield`, `companySize`, `salary`, `city`, `education`) VALUES (%s, %s, %s, %s, %s, %s, %s)"
        cursor.execute(sql, info)
    conn.commit()
```

#### 输出

```
北京中富金石咨询有限公司四川分公司
linux运维工程师
10k-15k
150-500人
['Linux', 'CI/CD', 'Python', 'Ansible']
2020-11-04 09:27:12
高新区
倪家桥
成都华栖云科技有限公司
高级运维工程师
10k-15k
500-2000人
['Linux']
2020-11-04 09:23:18
高新区
天府五街
成都心田花开科技有限公司
高级运维工程师
15k-20k
500-2000人
['CI/CD', 'Python', '运维开发', '系统运维']
2020-11-04 09:04:40
武侯区
桐梓林
新蛋科技（成都）有限公司
运维开发工程师
15k-25k
500-2000人
['python', 'IT 自动化方向']
2020-11-04 08:56:22
高新区
天府五街
成都维堰信息技术服务有限公司陕西分公司
数据库运维工程师
10k-15k
150-500人
['数据库运维', 'DBA', 'Hadoop', 'Oracle']
2020-11-04 08:42:38
郫县
None
科大讯飞股份有限公司
运维工程师-运营商BU-成都
15k-20k
2000人以上
['Shell', 'Linux', 'Python', 'MySQL']
2020-11-04 09:28:02
双流县
兴隆湖
成都心田花开科技有限公司
运维总监
20k-30k
500-2000人
['运维管理', '高级技术管理', '领导力', '技术管理']
2020-11-04 09:04:39
武侯区
桐梓林
深圳前海中电慧安科技有限公司
运维工程师（四川地区）
6k-12k
50-150人
['MongoDB', '大数据运维', 'MySql', 'Tomcat']
2020-11-04 08:57:23
高新区
天府五街
成都华栖云科技有限公司
中级售后运维工程师
8k-12k
500-2000人
['Linux', 'MySQL', 'SQLServer', 'Oracle']
2020-11-04 09:23:18
高新区
天府五街
北京字节跳动科技有限公司
分布式存储运维开发工程师
20k-40k
2000人以上
[]
2020-11-04 03:27:45
武侯区
天府五街
北京字节跳动科技有限公司
运维研发工程师
30k-60k
2000人以上
[]
2020-11-04 03:22:22
高新区
天府五街
博睿天下（成都）科技有限责任公司
运维工程师
10k-15k
50-150人
['DevOps/AIOps']
2020-11-04 08:51:53
高新区
天府三街
成都云智天下科技股份有限公司
系统运维工程师
9k-11k
50-150人
['Linux', 'DevOps/AIOps', 'CI/CD']
2020-11-04 09:27:48
高新区
红牌楼
四川久远银海软件股份有限公司
软件运维开发工程师
4k-8k
2000人以上
['Java', '运维开发', '系统运维']
2020-11-04 09:22:48
锦江区
None
成都维堰信息技术服务有限公司陕西分公司
linux运维工程师
8k-15k
150-500人
['Python', '现场运维', 'K8s/Docker', 'open stack']
2020-11-04 08:42:34
郫县
None
```


### 获取济南市政府招标内容


代码：
```
import requests
from bs4 import BeautifulSoup


def download_page(url):
    headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:61.0) Gecko/20100101 Firefox/61.0"}
    r = requests.get(url, headers=headers)
    return r.text


def get_content(html):
    output = "标题：{}   日期：{}\n------------\n"
    soup = BeautifulSoup(html, 'html.parser')
    con = soup.find(id='content')
    con_list = con.find_all('li')
    for i in con_list:
        a = i.find("a").text
        d = i.find("span", class_="span2").text

        save_txt(output.format(a, d))


def save_txt(*args):
    for i in args:
        with open('jn.txt', 'a', encoding='utf-8') as f:
            f.write(i)


def main():
    url = 'http://jnggzy.jinan.gov.cn/jnggzyztb/front/noticelist.do?type=1&xuanxiang=1&area='
    html = download_page(url)
    get_content(html)


if __name__ == '__main__':
    main()
```


解析获取内容时可以使用
- i.find("a").text
- i.find("a").get_text()
- i.find("a").string  

方式来进行获取解析内容，这里只用到了标题名称和日期，将解析的内容写到txt文件中。

**问题：通过上面代码获取到的内容，并不是最新的内容**

```
标题：济南市生态环境局济南市部分城镇集中式饮用水水...   日期：2020-06-11
------------
标题：商河县残疾人家庭无障碍改造项目招标公告   日期：2020-05-18
------------
标题：济南职业学院济南职业学院现代学徒制管理...   日期：2020-04-01
------------
标题：济南信息工程学校济南信息工程学校普通图...   日期：2020-01-09
------------
标题：山东省济南市人民检察院山东省济南市人民...   日期：2019-12-20
------------
标题：济南护理职业学院济南护理职业学院消防工...   日期：2019-06-13
------------
标题：济南市济阳区回河街道办事处回跺路改造工程   日期：2019-05-27
------------
标题：济南市排水管理服务中心济南市排水管理服...   日期：2019-01-10
------------
标题：山东省济南中学山东省济南中学软件开发服务竞争...   日期：2018-08-31
------------
标题：济南市食品药品监督管理局济南市食品药品监督管...   日期：2018-08-31
------------
标题：山东省济南第七中学足球训练营采购竞争性磋商公...   日期：2018-08-31
------------
标题：济南市技师学院济南市技师学院空调、电梯维修和...   日期：2018-08-31
------------
标题：济南市农业科学研究院济南市农业科学研究院试验...   日期：2018-08-31
------------
标题：济南信息工程学校济南信息工程学校物业管理服务...   日期：2018-08-31
------------
标题：济南市城乡交通运输委员会济南市主城区道路及公...   日期：2018-08-31
------------
```

**初步估计：**
网页是通过http:\//jnggzy.jinan.gov.cn/jnggzyztb/front/search.do这个请求获取最新的内容，并不是打开网页就是最新的数据。

下一步就是想办法获取search.do这个请求的数据

**步骤一：**
根据chrome的控制台，将From Data转换成source格式，放在url后方：
```
Form Data：area=&type=1&xuanxiang=%E6%8B%9B%E6%A0%87%E5%85%AC%E5%91%8A&subheading=&pagenum=1
```


**步骤二：**
配置连接header
```
def download_page(url):
    headers = {
        'Accept': '*/*',
        'Referer': 'http://jnggzy.jinan.gov.cn/jnggzyztb/front/noticelist.do?type=1&xuanxiang=1&area=',
        'Origin': 'http://jnggzy.jinan.gov.cn',
        'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.183 Safari/537.36'
    }
    r = requests.post(url, headers=headers, timeout=3)
    return json.loads(r.text)
```

**步骤三：**
解析连接数据
```
def get_content(html):
    output = "标题：{}   日期：{}\n------------\n"
    params = html.get("params")
    result = params.get("str")
    con_list = BeautifulSoup(result).find_all("li")
    for i in con_list:
        a = i.find("a").text
        d = i.find("span", class_="span2").text
        save_txt(output.format(a, d))
```
**步骤四：**
数据结果
```
标题：【电子全流程】济南市机械化清扫大队车辆保险竞争性磋商公告   日期：2020-11-04
------------
标题：【电子全流程】济南市南部山区管理委员会综合管理执法局济南市...   日期：2020-11-04
------------
标题：【电子全流程】中国共产党济南市委员会宣传部印刷服务竞争性磋...   日期：2020-11-04
------------
标题：【电子全流程】中国共产党济南市委员会宣传部印刷服务竞争性磋...   日期：2020-11-04
------------
标题：【电子全流程】山东省实验中学东校扫描仪竞争性谈判公告   日期：2020-11-04
------------
标题：【电子全流程】莱芜职业技术学院技术测量实训室建设公开招标公...   日期：2020-11-04
------------
标题：【电子全流程】济南市城市管理局济南市固体废弃物分类专项规划...   日期：2020-11-04
------------
标题：【电子全流程】济南市历城区发展和改革局济南市历城区发展和改...   日期：2020-11-04
------------
标题：【电子全流程】莱芜职业技术学院商管系学生创新创业实训室建设...   日期：2020-11-04
------------
标题：【电子全流程】莱芜职业技术学院智能工厂虚拟仿真与云服务实训...   日期：2020-11-04
------------
```
#### 获取跳转链接

获得的数据，只存在一个点击事件，只要将对应的id数据存到数据库中，就能分析点击事件组成跳转链接。
```
'showview(\\'E6BB8EE65366081B72346771888505B6\\',1)'
```
使用正则表达式：
```
rex = "\('(.+?)',"
```
直接获取到E6BB8EE65366081B72346771888505B6，将id存到数据库中，在前端显示时，组合成跳转链接。

#### 写入数据库

连接数据库,使用pymysql包建立连接：
```
import pymysql.cursors


def get_conn():
    '''建立数据库连接'''
    conn = pymysql.connect(host='localhost',
                           user='root',
                           password='******',
                           db='spider',
                           charset='utf8mb4',
                           cursorclass=pymysql.cursors.DictCursor)
    return conn
```

将数据传入insert方法，我这里传入的是一个元组类型：
```
def insert(conn, *info):
    '''数据写入数据库'''
    with conn.cursor() as cursor:
        sql = "INSERT INTO `jinanshizhengfu` (`title`, `link`, `date`, `city`) VALUES (%s, %s, %s, %s)"
        n = cursor.execute(sql, info)
    conn.commit()
```























