---
layout: post
title:  "Building a NewsCrawler: pt1"
date:   2014-05-09 01:39:44
categories: Server Python
---

#Introduction

This tutorial/walk through assumes that you are already familiar with python and the terminal. I would also strongly advice anyone remotely interested in understanding scrapy to have a look at their [documentation](http://doc.scrapy.org/en/latest/intro/overview.html) and preferably follow the quick tutorial and setup instruction outlined.

The example in the documentation is a simple spider. Basic spiders are good if we intend on retrieving information from a list of pages that we currently know the URLs of. This is no good in our scenario where a lot of pages need to be scraped. This is where the crawler comes in, crawlers provide a convenient mechanism to follow links, you can refer to the [documentation](http://doc.scrapy.org/en/latest/topics/spiders.html#crawling-rules) here to refresh your memory.

###Pick a website

We are going to experiment with a Kenyan news website. Please feel free to substitue this website with one that suits your taste. [Daily Nation](www.nation.co.ke) For those of you keen enough, you will realise that I have an app for this news source on the [App Store](https://itunes.apple.com/gb/app/habari/id509329627?mt=8)

####Install scrapy
Follow the [Installation Guide](http://doc.scrapy.org/en/latest/intro/install.html) here, but the gist of it all is basically.

	pip install Scrapy

####Create a new project

***Have you installed scrapy yet? I struggled to get it to work on osx, if you get __libxml__ issue, just know you are in for a long ride ***

	scrapy startproject hermes

This will create a project called `hermes` in the directory you ran it in ***tutorial*** in this case.
Explore this directory, its pretty boring at first.
	
		tutorial/ 
		    scrapy.cfg  	#config file for deployment
		    tutorial/    	#project dir/module
		        __init__.py
		        items.py		#Model
		        pipelines.py	
		        settings.py		#project settings
		        spiders/		#spider directory
		            __init__.py
		            ...

Scrapy did the burden of work for us. There are only two files we have to be concerned with at the moment, thats the __items.py__ and the spiders. The spider directory will hold all the spiders.

###Define the model.

In scrapy, the models are defined in the __items.py__ file. All items have a __Field()__ type. We create our model named `NationmediaItem` our items class looks like the following.

		from scrapy.item import Item, Field

		class NationmediaItem(Item):
			title = Field()
			link = Field()
			summary = Field()
			content = Field()
			author = Field()	
			
	
###Defining our Spider 
Spiders are user defined and they are usually quite specific to a particular website, or a group of websites with the same structure.

A spider in its most basic form  will have a list or URLS to download, rules that define how links are followed and a method to parse the content.

A spider must subclass  `scrapy.spider.Spider`   and define the following:

- `name:` Unique identifier for the spider.
- `start_urls:` a list of URLS where crawling will start from.
- `parse():` call back method called when the content has been downloaded, a `Response` object returned and is passed to the method as an argument.

Navigate into the `spider` directory and create an empty python file. Call this file **nmdspider.py**.

Our class looks like the following:

	from scrapy.spider import BaseSpider
	from scrapy.selector import HtmlXPathSelector
	from nationmedia.items import NationmediaItem
	from scrapy.contrib.spiders import CrawlSpider, Rule
	from scrapy.contrib.linkextractors.sgml import SgmlLinkExtractor
	from scrapy.selector import Selector
	
	class MainParser():
	    def __init__(self, response, tags):
	        self.response = response
	        self.tags = tags
	
	    def scrapNationOnline(self):
	        hxs = Selector(self.response)
	        item = NationmediaItem()
	        item ["link"] = self.response.url
	        item ["title"]  = ''.join(hxs.xpath('//div[@class=\"story-view\"]/header/h1/text()').extract()) #this worked by taking the page title, contains tags we dont need/want ''.join(hxs.css('title::text').extract())
	        item ["summary"] = ''.join(hxs.xpath('//div/ul/li/text()').extract())
	        item ["content"] = '\n'.join(hxs.xpath('//article/section/div/p/text()').extract())
	
	        #retrieve the author. There are numerous formatting issue with this tag
	
	        author = hxs.xpath('//section[@class=\"author\"]/strong/text()').extract()
	        if not author:
	            author = hxs.xpath('//section[@class=\"author\"]/text()').extract()
	        item ["author"] = ''.join(author)
	
	        return item
	
	class PoliticsSpider(CrawlSpider):
	    """Scrapes Polics News"""
	    name = "politics"
	    allowed_domains = ["nation.co.ke"]
	    start_urls = ["http://www.nation.co.ke/news/politics/-/1064/1064/-/3hxffhz/-/index.html"]
	    rules = (
	        Rule(SgmlLinkExtractor(allow=('news\/politics\/',), deny=('\/view\/asFeed\/', '\/-\/1064\/1064\/-\/3hxffhz\/-\/index.html')), callback='parse_page', follow=True),
	    )
	    def parse_page(self, response):
	        mainParser = MainParser(response, ["News", "Politics"])
	        item = mainParser.scrapNationOnline()
	        return item
	
	  
	  
Take a look at the code from the repository [here](https://github.com/edwinbosire/hermes) Its going to be clearer than the mess above.

###Time to run the crawler

	scrapy crawl politics
	
Its as simple as that. To run the spider and save the results in a json file run
	
	scrapy crawl politics -o politics.json -t json
	
###Output

The output should be a json file of all the scrapped articles. In this case from the politics section of nation.co.ke

### Some Explanation

As you can see from the code, we are using xpath to extract information from the html response, this is the part that makes scrapping such a fragile task, if the website you are scrapping updates its html files, then you have to do the same here.

#####PoliticsSpider [Class:]
This is our main class, if you are going to have multiple crawlers running, this is the class to duplicate. It takes a few parameters.

1. **Name**
2. **Allowed Domains**
3. **Start_urls**
4. **rules**.

All these are self explanatory apart from the last one. This are rules to follow while crawling links originating from the Start_urls. 

**Allow:** In our example, we are only accepting URLS containing *news/politics*, as we sure that any article in such a URL is a politics item.

**Restricted_xpaths:** Restricts the link to a certain xpath, we are currently not implementing this.

**Deny:** We also deny /view/asFeed as this is the extension found in the RSS feed page, there are plenty of other rules we can apply, but these two seem to be of utmost importance.

**Callback:** This property, this is the method we call back to after parsing is complete, please** DO NOT** name your method 'parse' as this is already in use internally.

**Follow:** Instructs the program to continue following links as long as they exist, this is how we recursively crawl entire websites.
	
######Parse_page [Method]

This method is called once we have retrieved the contents of a page, this is where our parsing occurs.

We delegate the actual parsing to the **MainParse** class, just trying to observe some OOP principles. To execute the actual parsing, we make the following call. `item = mainParser.scrapNationOnline()`. It returns an **Item** object, this item object defines a news article.

###Conclusion.

That is all that you need to parse a website, I would suggest you read the documentaion for Scrapy as I have left quite a lot unanswered here.

Website scraping is fun, but don't be a douche bag, don't run 100 scrapers 24/7 on someone else's site. Also don't steal information and claim it as your own, this really pisses some people off.

In the next web scraping tutorial, we will look at how we deploy our scrapy to the cloud and automate the entire process.

