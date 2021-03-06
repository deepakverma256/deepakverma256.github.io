---
layout: post
title: "Getting Started with Scrapinghub spiders"
date: 2017-01-19
---
Getting Started with Scrapinghub spiders
Purpose of the document
Create a new spider project.
Our first Spider
Crawling a URL
What just happened under the hood?
A shortcut to the start_requests method
Scraping your first element.
XPath: a brief intro
Extracting quotes and authors
Extracting data in our spider
Item explored
Declaring Items
Deploying the spider to Scrapy Cloud
Scrapy cloud
Shub
Debug your spider
Parse Command
Scrapy Shell
Open in browser
Logging
Purpose of the document
This documents gives a basic knowledge for creating, running a scrapinghub spiders and then deploying them to scrapy cloud.
Document also talks about helpful debugging tricks and some useful tips.
Create a new spider project.
For this you must have following installed on your local machine.
scrapinghub library
For this your can execute following command:
pip install scrapinghub
scrapy
For this you can execute following command:
pip install scrapy
So,Before you start scraping, you will have to set up a new Scrapy project. Enter a directory where you’d like to store your code and run:
scrapy startproject tutorial
This will create a tutorial directory with the following contents:
tutorial/
scrapy.cfg # deploy configuration file
tutorial/ # project's Python module, you'll import your code from here
__init__.py
items.py # project items definition file
pipelines.py # project pipelines file
settings.py # project settings file
spiders/ # a directory where you'll later put your spiders
__init__.py
Our first Spider
Spiders are classes that you define and that Scrapy uses to scrape information from a website (or a group of websites). They must subclass
scrapy.Spider and define the initial requests to make, optionally how to follow links in the pages, and how to parse the downloaded page
content to extract data.
This is the code for our first Spider. Save it in a file named quotes_spider.py under the tutorial/spiders directory in your project:
import scrapy
class QuotesSpider(scrapy.Spider):
name = "quotes"
def start_requests(self):
urls = [
'http://quotes.toscrape.com/page/1/',
'http://quotes.toscrape.com/page/2/',
]
for url in urls:
yield scrapy.Request(url=url, callback=self.parse)
def parse(self, response):
page = response.url.split("/")[-2]
filename = 'quotes-%s.html' % page
with open(filename, 'wb') as f:
f.write(response.body)
self.log('Saved file %s' % filename)
As you can see, our Spider subclasses scrapy.Spider and defines some attributes and methods:
name: identifies the Spider. It must be unique within a project, that is, you can’t set the same name for different Spiders.
start_requests(): must return an iterable of Requests (you can return a list of requests or write a generator function) which the
Spider will begin to crawl from. Subsequent requests will be generated successively from these initial requests.
parse(): a method that will be called to handle the response downloaded for each of the requests made. The response parameter
is an instance of TextResponse that holds the page content and has further helpful methods to handle it.
The parse() method usually parses the response, extracting the scraped data as dicts and also finding new URLs to follow and
creating new requests (Request) from them.
Crawling a URL
To put our spider to work, go to the project’s top level directory and run:
scrapy crawl quotes
This command runs the spider with name quotes that we’ve just added, that will send some requests for the quotes.toscrape.com doma
in. You will get an output similar to this:
... (omitted for brevity)
2016-12-16 21:24:05 [scrapy.core.engine] INFO: Spider opened
2016-12-16 21:24:05 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items
(at 0 items/min)
2016-12-16 21:24:05 [scrapy.extensions.telnet] DEBUG: Telnet console listening on 127.0.0.1:6023
2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (404) <GET
http://quotes.toscrape.com/robots.txt> (referer: None)
2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/>
(referer: None)
2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/2/>
(referer: None)
2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-1.html
2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-2.html
2016-12-16 21:24:05 [scrapy.core.engine] INFO: Closing spider (finished)
...
Now, check the files in the current directory. You should notice that two new files have been created: quotes-1.html and quotes-2.html, with
the content for the respective URLs, as our parse method instructs.
Note
If you are wondering why we haven’t parsed the HTML yet, hold on, we will cover that soon.
What just happened under the hood?
Scrapy schedules the scrapy.Request objects returned by the start_requests method of the Spider. Upon receiving a response for
each one, it instantiates Response objects and calls the callback method associated with the request (in this case, the parse method)
passing the response as argument.
A shortcut to the start_requests method
Instead of implementing a start_requests() method that generates scrapy.Request objects from URLs, you can just define a start_
urls class attribute with a list of URLs. This list will then be used by the default implementation of start_requests() to create the initial
requests for your spider:
import scrapy
class QuotesSpider(scrapy.Spider):
name = "quotes"
start_urls = [
'http://quotes.toscrape.com/page/1/',
'http://quotes.toscrape.com/page/2/',
]
def parse(self, response):
page = response.url.split("/")[-2]
filename = 'quotes-%s.html' % page
with open(filename, 'wb') as f:
f.write(response.body)
The parse() method will be called to handle each of the requests for those URLs, even though we haven’t explicitly told Scrapy to do so.
This happens because parse() is Scrapy’s default callback method, which is called for requests without an explicitly assigned callback.
Scraping your first element.
The best way to learn how to extract data with Scrapy is trying selectors using the shell Scrapy shell. Run:
scrapy shell 'http://quotes.toscrape.com/page/1/'
Note
Remember to always enclose urls in quotes when running Scrapy shell from command-line, otherwise urls containing arguments (ie. & charac
ter) will not work.
On Windows, use double quotes instead:
scrapy shell "http://quotes.toscrape.com/page/1/"
You will see something like:
[ ... Scrapy log here ... ]
2016-09-19 12:09:27 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/>
(referer: None)
[s] Available Scrapy objects:
[s] scrapy scrapy module (contains scrapy.Request, scrapy.Selector, etc)
[s] crawler <scrapy.crawler.Crawler object at 0x7fa91d888c90>
[s] item {}
[s] request <GET http://quotes.toscrape.com/page/1/>
[s] response <200 http://quotes.toscrape.com/page/1/>
[s] settings <scrapy.settings.Settings object at 0x7fa91d888c10>
[s] spider <DefaultSpider 'default' at 0x7fa91c8af990>
[s] Useful shortcuts:
[s] shelp() Shell help (print this help)
[s] fetch(req_or_url) Fetch request (or URL) and update local objects
[s] view(response) View response in a browser
>>>
Using the shell, you can try selecting elements using CSS with the response object:
>>> response.css('title')
[<Selector xpath='descendant-or-self::title' data='<title>Quotes to Scrape</title>'>]
The result of running response.css('title') is a list-like object called SelectorList, which represents a list of Selector objects that
wrap around XML/HTML elements and allow you to run further queries to fine-grain the selection or extract the data.
To extract the text from the title above, you can do:
>>> response.css('title::text').extract()
['Quotes to Scrape']
There are two things to note here: one is that we’ve added ::text to the CSS query, to mean we want to select only the text elements
directly inside <title> element. If we don’t specify ::text, we’d get the full title element, including its tags:
>>> response.css('title').extract()
['<title>Quotes to Scrape</title>']
The other thing is that the result of calling .extract() is a list, because we’re dealing with an instance of SelectorList. When you know
you just want the first result, as in this case, you can do:
>>> response.css('title::text').extract_first()
'Quotes to Scrape'
As an alternative, you could’ve written:
>>> response.css('title::text')[0].extract()
'Quotes to Scrape'
However, using .extract_first() avoids an IndexError and returns None when it doesn’t find any element matching the selection.
There’s a lesson here: for most scraping code, you want it to be resilient to errors due to things not being found on a page, so that even if
some parts fail to be scraped, you can at least get some data.
Besides the extract() and extract_first() methods, you can also use the re() method to extract using —regular expressions:
>>> response.css('title::text').re(r'Quotes.*')
['Quotes to Scrape']
>>> response.css('title::text').re(r'Q\w+')
['Quotes']
>>> response.css('title::text').re(r'(\w+) to (\w+)')
['Quotes', 'Scrape']
In order to find the proper CSS selectors to use, you might find useful opening the response page from the shell in your web browser using v
iew(response). You can use your browser developer tools or extensions like Firebug.
Selector Gadget is also a nice tool to quickly find CSS selector for visually selected elements, which works in many browsers.
XPath: a brief intro
Besides CSS, Scrapy selectors also support using XPath expressions:
>>> response.xpath('//title')
[<Selector xpath='//title' data='<title>Quotes to Scrape</title>'>]
>>> response.xpath('//title/text()').extract_first()
'Quotes to Scrape'
XPath expressions are very powerful, and are the foundation of Scrapy Selectors. In fact, CSS selectors are converted to XPath
under-the-hood. You can see that if you read closely the text representation of the selector objects in the shell.
While perhaps not as popular as CSS selectors, XPath expressions offer more power because besides navigating the structure, it can also
look at the content. Using XPath, you’re able to select things like: select the link that contains the text “Next Page”. This makes XPath very
fitting to the task of scraping, and we encourage you to learn XPath even if you already know how to construct CSS selectors, it will make
scraping much easier.
Extracting quotes and authors
Now that you know a bit about selection and extraction, let’s complete our spider by writing the code to extract the quotes from the web page.
Each quote in http://quotes.toscrape.com/ is represented by HTML elements that look like this:
<div class="quote">
<span class="text">“The world as we have created it is a process of our
thinking. It cannot be changed without changing our thinking.”</span>
<span>
by <small class="author">Albert Einstein</small>
<a href="/author/Albert-Einstein">(about)</a>
</span>
<div class="tags">
Tags:
<a class="tag" href="/tag/change/page/1/">change</a>
<a class="tag" href="/tag/deep-thoughts/page/1/">deep-thoughts</a>
<a class="tag" href="/tag/thinking/page/1/">thinking</a>
<a class="tag" href="/tag/world/page/1/">world</a>
</div>
</div>
Let’s open up scrapy shell and play a bit to find out how to extract the data we want:
$ scrapy shell 'http://quotes.toscrape.com'
We get a list of selectors for the quote HTML elements with:
>>> response.css("div.quote")
Each of the selectors returned by the query above allows us to run further queries over their sub-elements. Let’s assign the first selector to a
variable, so that we can run our CSS selectors directly on a particular quote:
>>> quote = response.css("div.quote")[0]
Now, let’s extract title, author and the tags from that quote using the quote object we just created:
>>> title = quote.css("span.text::text").extract_first()
>>> title
'“The world as we have created it is a process of our thinking. It cannot be changed without changing
our thinking.”'
>>> author = quote.css("small.author::text").extract_first()
>>> author
'Albert Einstein'
Given that the tags are a list of strings, we can use the .extract() method to get all of them:
>>> tags = quote.css("div.tags a.tag::text").extract()
>>> tags
['change', 'deep-thoughts', 'thinking', 'world']
Having figured out how to extract each bit, we can now iterate over all the quotes elements and put them together into a Python dictionary:
>>> for quote in response.css("div.quote"):
... text = quote.css("span.text::text").extract_first()
... author = quote.css("small.author::text").extract_first()
... tags = quote.css("div.tags a.tag::text").extract()
... print(dict(text=text, author=author, tags=tags))
{'tags': ['change', 'deep-thoughts', 'thinking', 'world'], 'author': 'Albert Einstein', 'text': '“The
world as we have created it is a process of our thinking. It cannot be changed without changing our
thinking.”'}
{'tags': ['abilities', 'choices'], 'author': 'J.K. Rowling', 'text': '“It is our choices, Harry, that
show what we truly are, far more than our abilities.”'}
... a few more of these, omitted for brevity
>>>
Extracting data in our spider
Let’s get back to our spider. Until now, it doesn’t extract any data in particular, just saves the whole HTML page to a local file. Let’s integrate
the extraction logic above into our spider.
A Scrapy spider typically generates many dictionaries containing the data extracted from the page. To do that, we use the yield Python
keyword in the callback, as you can see below:
import scrapy
class QuotesSpider(scrapy.Spider):
name = "quotes"
start_urls = [
'http://quotes.toscrape.com/page/1/',
'http://quotes.toscrape.com/page/2/',
]
def parse(self, response):
for quote in response.css('div.quote'):
yield {
'text': quote.css('span.text::text').extract_first(),
'author': quote.css('span small::text').extract_first(),
'tags': quote.css('div.tags a.tag::text').extract(),
}
If you run this spider, it will output the extracted data with the log:
2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
{'tags': ['life', 'love'], 'author': 'André Gide', 'text': '“It is better to be hated for what you are
than to be loved for what you are not.”'}
2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
{'tags': ['edison', 'failure', 'inspirational', 'paraphrased'], 'author': 'Thomas A. Edison', 'text':
"“I have n
Item explored
The main goal in scraping is to extract structured data from unstructured sources, typically, web pages. Scrapy spiders can return the
extracted data as Python dicts. While convenient and familiar, Python dicts lack structure: it is easy to make a typo in a field name or return
inconsistent data, especially in a larger project with many spiders.
To define common output data format Scrapy provides the Item class. Item objects are simple containers used to collect the scraped data.
They provide a dictionary-like API with a convenient syntax for declaring their available fields.
Various Scrapy components use extra information provided by Items: exporters look at declared fields to figure out columns to export,
serialization can be customized using Item fields metadata, trackref tracks Item instances to help finding memory leaks, etc.
Declaring Items
Items are declared using a simple class definition syntax and Field objects. Here is an example:
import scrapy
class Product(scrapy.Item):
name = scrapy.Field()
price = scrapy.Field()
stock = scrapy.Field()
last_updated = scrapy.Field(serializer=str)
Deploying the spider to Scrapy Cloud
Scrapy cloud
Scrapy Cloud is a hosted, cloud-based service by Scrapinghub, the company behind Scrapy.
Scrapy Cloud removes the need to setup and monitor servers and provides a nice UI to manage spiders and review scraped items, logs and
stats.
To deploy spiders to Scrapy Cloud you can use the shub command line tool.
Scrapy Cloud is compatible with Scrapyd and one can switch between them as needed - the configuration is read from the scrapy.cfg file
just like scrapyd-deploy.
Shub
The Scrapy Cloud command line client is called shub . It allows you to deploy projects (and dependencies), run spiders, retrieve scraped
data and watch logs, all without leaving the command line.
If you have pip installed on your system, you can install shub from the Python Package Index by simply running:
pip install shub
To deploy a project to the scrapy cloud execute following command from inside the project directory:
shub deploy
Debug your spider
This document explains the most common techniques for debugging spiders. Consider the following scrapy spider below:
import scrapy
from myproject.items import MyItem
class MySpider(scrapy.Spider):
name = 'myspider'
start_urls = (
'http://example.com/page1',
'http://example.com/page2',
)
def parse(self, response):
# collect `item_urls`
for item_url in item_urls:
yield scrapy.Request(item_url, self.parse_item)
def parse_item(self, response):
item = MyItem()
# populate `item` fields
# and extract item_details_url
yield scrapy.Request(item_details_url, self.parse_details, meta={'item': item})
def parse_details(self, response):
item = response.meta['item']
# populate more `item` fields
return item
Basically this is a simple spider which parses two pages of items (the start_urls). Items also have a details page with additional information,
so we use the meta functionality of Request to pass a partially populated item.
Parse Command
The most basic way of checking the output of your spider is to use the parse command. It allows to check the behaviour of different parts of
the spider at the method level. It has the advantage of being flexible and simple to use, but does not allow debugging code inside a method.
In order to see the item scraped from a specific url:
$ scrapy parse --spider=myspider -c parse_item -d 2 <item_url>
[ ... scrapy log lines crawling example.com spider ... ]
>>> STATUS DEPTH LEVEL 2 <<<
# Scraped Items ------------------------------------------------------------
[{'url': <item_url>}]
# Requests -----------------------------------------------------------------
[]
Using the --verbose or -v option we can see the status at each depth level:
$ scrapy parse --spider=myspider -c parse_item -d 2 -v <item_url>
[ ... scrapy log lines crawling example.com spider ... ]
>>> DEPTH LEVEL: 1 <<<
# Scraped Items ------------------------------------------------------------
[]
# Requests -----------------------------------------------------------------
[<GET item_details_url>]
>>> DEPTH LEVEL: 2 <<<
# Scraped Items ------------------------------------------------------------
[{'url': <item_url>}]
# Requests -----------------------------------------------------------------
[]
Checking items scraped from a single start_url, can also be easily achieved using:
$ scrapy parse --spider=myspider -d 3 'http://example.com/page1'
Scrapy Shell
While the parse command is very useful for checking behaviour of a spider, it is of little help to check what happens inside a callback,
besides showing the response received and the output. How to debug the situation when parse_details sometimes receives no item?
Fortunately, the shell is your bread and butter in this case:
from scrapy.shell import inspect_response
def parse_details(self, response):
item = response.meta.get('item', None)
if item:
# populate more `item` fields
return item
else:
inspect_response(response, self)
Open in browser
Sometimes you just want to see how a certain response looks in a browser, you can use the open_in_browser function for that. Here is an
example of how you would use it:
from scrapy.utils.response import open_in_browser
def parse_details(self, response):
if "item name" not in response.body:
open_in_browser(response)
open_in_browser will open a browser with the response received by Scrapy at that point, adjusting the base tag so that images and styles
are displayed properly.
Logging
Logging is another useful option for getting information about your spider run. Although not as convenient, it comes with the advantage that
the logs will be available in all future runs should they be necessary again:
def parse_details(self, response):
item = response.meta.get('item', None)
if item:
# populate more `item` fields
return item
else:
self.logger.warning('No item received for %s', response.url)
