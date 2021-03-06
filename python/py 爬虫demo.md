```python
# file: html_downloader.py
# coding:utf8
import urllib2
class HtmlDownloader(object):
    
    
    def download(self,url):
        if url is None:
            return None
        
        response = urllib2.urlopen(url)
        
        if response.getcode() != 200:
            return None
        
        return response.read()

```

```python
# file: html_parser.py
# coding:utf8
from bs4 import BeautifulSoup
import re
import urlparse
class HtmlParser(object):
    
    
    def _get_new_urls(self, page_url, soup):
        new_urls = set()
        #/view/123.htm
        links = soup.find_all('a',href=re.compile(r"/view/\d+\.htm"))
        for link in links:
            new_url = link['href']
            new_full_url = urlparse.urljoin(page_url,new_url)
            new_urls.add(new_full_url)
        return new_urls
    
    
    def _get_new_data(self, page_url, soup):
        res_data = {}
        
        #url
        res_data['url'] = page_url
        
        #<dd class="lemmaWgt-lemmaTitle-title"> <h1>Python</h1>
        title_node = soup.find('dd',class_="lemmaWgt-lemmaTitle-title").find("h1")
        res_data['title'] = title_node.get_text()
        #<div class="lemma-summary">
        summary_node = soup.find('div',class_="lemma-summary")
        res_data['summary'] = summary_node.get_text()
        
        return res_data
    
    
    def parse(self, page_url,html_cont):
        if page_url is None or html_cont is None:
            return
        
        soup = BeautifulSoup(html_cont,'html.parser',from_encoding='utf-8')
        new_urls = self._get_new_urls(page_url,soup)
        new_data = self._get_new_data(page_url,soup)
        return new_urls, new_data
```

```python
# file: url_manager
# coding:utf8

class UrlManager(object):
    def __init__(self):
        self.new_urls = set()
        self.old_urls = set()
        
    
    def add_new_url(self,url):
        if url is None:
            return
        if url not in self.new_urls and url not in self.old_urls:
            self.new_urls.add(url)
        
    def add_new_urls(self,urls):
        if urls is None or len(urls) == 0:
            return
        for url in urls:
            self.add_new_url(url)
    
    def has_new_url(self):
        return len(self.new_urls) != 0

    
    def get_new_url(self):
        new_url = self.new_urls.pop()
        self.old_urls.add(new_url)
        return new_url
```

```python
# file: html_outputer.py
# coding:utf8

class HtmlOutputer(object):
    def __init__(self):
        self.datas = []
        
    
    def collect_data(self,data):
        if data is None:
            return
        self.datas.append(data)

    
    def output_html(self):
        fout = open('output.html','w')
        fout.write("<!DOCTYPE html>")
        fout.write("<html lang=\"en\">")
        fout.write("<head>")
        fout.write("<meta charset=\"UTF-8\" />")
        fout.write("<title>Document</title>")
        fout.write("</head>")
        fout.write("<body>")
        fout.write("<table>")
        
        for data in self.datas:
            fout.write("<tr>")
            fout.write("<td>%s</td>" % data['url'])
            fout.write("<td>%s</td>" % data['title'].encode('utf-8'))
            fout.write("<td>%s</td>" % data['summary'].encode('utf-8'))
            fout.write("</tr>")
        
        fout.write("</table>")
        fout.write("</body>")
        fout.write("</html>")
```

```python
# file: spider_main.py
# coding:utf8
#from baike_spider import url_manager, html_downloader, html_parser,html_outputer
import url_manager, html_downloader, html_parser,html_outputer
class SpiderMain(object):
    def __init__(self):
        self.urls = url_manager.UrlManager()
        self.downloader = html_downloader.HtmlDownloader()
        self.parser = html_parser.HtmlParser()
        self.outputer = html_outputer.HtmlOutputer()
    
    def craw(self, root_url):
        count = 1
        self.urls.add_new_url(root_url)
        while self.urls.has_new_url():
            try:
                new_url = self.urls.get_new_url()
                print 'carw %d : %s' % (count,new_url)
                html_cont = self.downloader.download(new_url)
                new_urls,new_data = self.parser.parse(new_url,html_cont)
                self.urls.add_new_urls(new_urls)
                self.outputer.collect_data(new_data)
                
                if count == 100:
                    break
                
                count = count + 1
            except:
                print 'craw failed'
        self.outputer.output_html()
        
if __name__=="__main__":
    root_url = "http://baike.baidu.com/view/21087.htm"
    obj_spider = SpiderMain()
    obj_spider.craw(root_url)
```
