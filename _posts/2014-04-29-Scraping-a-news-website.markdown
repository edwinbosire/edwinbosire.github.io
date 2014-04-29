---
layout: post
title:  "Introduction: Scraping a news website"
date:   2014-04-29 22:39:44
categories: Server Python
---

##Introduction

The first project that I ever wanted to build when I was learning to program for mobile was a news app for my university paper [Leeds Student](http://www.leedsstudent.org/). I got started, but I soon realised that an app like that requires more than a Objective-C know-how.

###Motivation
I wanted to experiment with a new programming language to solve a problem I had. In this series, I will experiment with creating my own backend service that scrapes data from a news site using python, store the structured data in a mongoDB instance and serve the results from an API written on top of a python framework.

This tutorial is not just about scraping websites, its my journey into the world of big data, creating a news app is the most basic example I could think of, but the content collected can also be analysed for sentiments and trends and used by researchers and other PR/Media companies for a wide range of reasons. Is this all legal? Probably, but don't take my word for it. Least we forget google is one big scrapper as well! It all depends on what you do with the information.

###Background
I have an app out there that currently gets news articles from an RSS feed on [Nation Media](http://www.nation.co.ke), the app is called Habari and its available on the [app store](https://itunes.apple.com/gb/app/habari/id509329627?mt=8) . It is not a very well written piece of software. The backend, just like the app itself is not very well polished, I would like to think of it as a work-in-progress



This series will be divided into three parts. There's no gurantee that I will finish writting all the parts, if anyone is reading and wants some help, I would be more than glad if you shoot me an email or find me on [twitter](www.twitter.com/edwinbosire)

This series will include:

1. Building the **Scrapper** and Crawler
2. Saving our structured data using **mongoDB**
3. Implementing the API using **flask**


Ultimately, this post Raison d'être is to keep track of my learning activities. There will be plenty of bugs.