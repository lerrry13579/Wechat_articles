# Wechat_articles

#系统windows64  Pycharm里运行 Python3.6.1

import re
import urllib.request
import time
import urllib.error

#模拟成浏览器
headers = ("User-Agent", 'Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:56.0) Gecko/20100101 Firefox/56.0')
opener = urllib.request.build_opener()
opener.addheaders = [headers]

#将opener 安装为全局
urllib.request.install_opener(opener)

#设置一个存储文章网址的列表listurl
listurl = []

#使用代理服务器函数
def use_proxy(proxy_addr,url):
    #建立异常处理机制
    try:
        import urllib.request
        proxy = urllib.request.ProxyHandler({'http':proxy_addr})
        opener = urllib.request.build_opener(proxy,urllib.request.HTTPHandler)
        urllib.request.install_opener(opener)
        data = urllib.request.urlopen(url).read().decode('utf-8')
        return data
    except urllib.error.URLError as e:
        if hasattr(e,'code'):
            print(e.code)
        if hasattr(e,'reason'):
            print(e.reason)
        time.sleep(10)
    except Exception as e:
        print('Exception:'+str(e))
        time.sleep(1)

#获取文章链接函数
def getlistrul(key,pagestart,pageend,proxy):
    try:
        page = pagestart
        #编码关键字
        keycode = urllib.request.quote(key)
        #编码&page
        pagecode = urllib.request.quote('&page')
        for page in range(pagestart,pageend+1):
            #循环构建各页的url链接
            url = 'http://weixin.sogou.com/weixin?type=2&query='+keycode+pagecode+str(page)
            #用代理服务器爬取
            data1 = use_proxy(proxy,url)
            #为了调试获取data1是否成功
            print(data1)
            #文章的正则表达式
            listurlpat = '<div class="txt-box">.*?(http://.*?)"'
            lists = re.compile(listurlpat, re.S).findall(data1)
            #为了看获取文章链接是否成功
            for list in lists:
                print(list)
            listurl.append(lists)
        print('一共获取到'+str(len(listurl))+'页')
        return listurl
    except urllib.error.URLError as e:
        if hasattr(e,'code'):
            print(e.code)
        if hasattr(e,'reason'):
            print(e.reason)
        time.sleep(10)
    except Exception as e:
        print('Exception:'+str(e))
        time.sleep(1)

#通过文章链接获取文章内容
def getcontent(listurl,proxy):
        i = 0
        fh = open('wechat.html','ab')
        #listurl此时是二维列表,第一维是页，第二维是处于该页的第几个
        for i in range(0,len(listurl)):
            for j in range(0,len(listurl[i])):
                try:
                    url = listurl[i][j]
                    #采集到的网址里“&”后面比真实网址多了一个"amp;"
                    url = url.replace("amp;","")
                    data = use_proxy(proxy,url)

                    #文章标题和内容的正则表达式
                    titlepat = '<title>(.*?)</title>'
                    contentpat = '<div id="js_content" class="rich_media_content">(.*?)</div>'
                    title = re.compile(titlepat).findall(data)
                    content = re.compile(contentpat,re.S).findall(data)

                    #初始化文章标题和内容
                    thistitle = '此次没有获取到'
                    thiscontent = '此次没有获取到'

                    #如果标题列表不为空，说明找到了标题，获取列表第0个元素，即此次标题赋给标量thistitle
                    if (title !=[]):
                        thistitle = title[0]
                    if (content !=[]):
                        thiscontent = content[0]

                    #将标题和内容汇总赋值给dataall
                    dataall = '<p>标题为："+thistitle+"</p><p>内容为："+thiscontent+"</p><br>'

                    #将该篇文章的标题与内容总信息写入对应文件
                    fh.write(dataall.encode('utf-8'))
                    print("第"+str(i)+"几个网页"+str(j)+"个文章")
                except urllib.error.URLError as e:
                    if hasattr(e,'code'):
                        print(e.code)
                    if hasattr(e,'reason'):
                        print(e.reason)
                    time.sleep(10)
                except Exception as e:
                    print("Exception:"+str(e))
                    time.sleep(1)
        fh.close()

#main()函数
key = '物联网'
#在该网址上复制的最新代理ip   http://31f.cn/area/%E4%B8%AD%E5%9B%BD/
proxy = '223.112.84.30:3128'
pagestart = 1
pageend = 2
# end = 10
#循环获取proxy并且调用前面我们定义的函数
# for k in range(end)
#     proxy =

listurl = getlistrul(key,pagestart,pageend,proxy)

getcontent(listurl,proxy)






















