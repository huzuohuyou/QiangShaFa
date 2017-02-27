# QiangShaFa

> 使用fiddler工具进行抓包，使用python进行osc乱弹抢沙发

1. 查看乱弹列表
![](http://i.imgur.com/AcjZ7Qz.jpg)
2. 使用fiddler抓包
![](http://i.imgur.com/f6iru0s.jpg)
3.使用python beautifulsoup对返回的数据进行分析并分析如何判断最新乱弹的出现

```

flag=False
def hasnews(preid):
    try:
        global flag
        ssl._create_default_https_context = ssl._create_unverified_context
        import http.client,datetime
        conn = http.client.HTTPSConnection('my.oschina.net')
        conn.request("GET", "/xxiaobian/blog")
        sourp=BeautifulSoup(conn.getresponse().read(),'lxml')
        gilslist=sourp.find_all(class_='time')
        import time
        id=str(sourp.find_all(class_='blog-title',limit=1)[0]['href']).split('/')[-1]
        if preid==None:
            return id
        if id!=str(preid):
            return id
        else:
            print(time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())+str(gilslist[0].get_text().strip().replace('发布','').strip()))
            if flag:
                print('sleep 2 second ...')
                time.sleep(2)
            else :
                flag=True
                date_str=datetime.datetime.now().strftime("%Y-%m-%d 23:40:50")
                endtime=datetime.datetime.strptime(date_str,"%Y-%m-%d %H:%M:%S")
                now=datetime.datetime.now()
                print('endtime:{end} now:{now}'.format(end=endtime,now=now))
                interval=(endtime-now)
                sec = interval.days*24*3600 + interval.seconds
                print('need sleep {second} second ...'.format(second=abs(sec)))
                time.sleep(sec)
            return None
    except ValueError as e:
        sendEmail('发生异常了！'+str(e))
        fr=open('log.txt')
        fr.write(str(e))
        fr.close()

```

4.进行评论在fiddler中进行抓包
![](http://i.imgur.com/vQRcDlR.jpg)
5.使用python进行请求发送

```

def qiangshafa(url):
    ssl._create_default_https_context = ssl._create_unverified_context
    #url = "https://my.oschina.net/action/blog/add_comment?blog=797134"
    postdata =urllib.parse.urlencode({ "content":"今天是周一哈哈哈！" }).encode('utf-8')
    header = {
    "Accept": "application/json, text/javascript, */*; q=0.01",
    "Origin": "https://my.oschina.net",
    "X-Requested-With": "XMLHttpRequest",
    "User-Agent": "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36",
    "Content-Type": "application/x-www-form-urlencoded; charset=UTF-8",
    "Referer":"https://my.oschina.net/xxiaobian/blog/844061?p=3&temp=1487827633938",
    "Accept-Encoding": "gzip, deflate, br",
    "Accept-Language": "zh-CN,zh;q=0.8,en;q=0.6",
    "Cookie": "你的Cookie"}
    req = urllib.request.Request(url,postdata,header)
    print(urllib.request.urlopen(req).read().decode('utf-8'))
    cj = http.cookiejar.CookieJar()
    opener = urllib.request.build_opener(urllib.request.HTTPCookieProcessor(cj))
    r = opener.open(req)
    print(r.read().decode('utf-8')) 

```
 
6.发送Email模块方便通知

`def sendEmail(message):
    from email.mime.text import MIMEText
    msg = MIMEText(message, 'plain', 'utf-8')
    from_addr = '13126506430@163.com'
    password = '*******'
    输入收件人地址:
    to_addr = '13126506430@163.com'
    输入SMTP服务器地址:
    smtp_server = 'smtp.163.com'
    import smtplib
    server = smtplib.SMTP(smtp_server, 25) # SMTP协议默认端口是25
    server.set_debuglevel(1)
    server.login(from_addr, password)
    server.sendmail(from_addr, [to_addr], msg.as_string())
    server.quit()
`
 
7.运行

`
class log():
    def __init__(self,id):
        self.id=id
id=None
if __name__ == '__main__':
    global id
    import http.client,datetime, pickle,os
    if os.path.exists('dump.txt'):
        f = open('dump.txt', 'rb')
        logid = pickle.load(f)
        preid=logid.id
        f.close()
        print('preid is '+str(preid))
    else:
        preid=None
    while id==None:
        id = hasnews(preid)
    f = open('dump.txt', 'wb')
    pickle.dump(log(id), f)
    f.close()
    url="https://my.oschina.net/action/blog/add_comment?blog={id}".format(id=id)    
    qiangshafa(url)
    sendEmail('over \t'+url+'\t'+str(datetime.datetime.now()))
` 