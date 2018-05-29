---
title:  "Part 1: A Most Curious Business Card"
search: true
excerpt: "SANS Holiday Hack 2016 Writeup - The Challenges - Part 1"
header:
  #image: /assets/images/foo-bar-identity.jpg
  teaser: /assets/images/sans-2016.png
categories:
  - SANS
  - Holday Hack
  - writeup
classes: wide
last_modified_at: 2016-03-06T19:56:50+01:00
---


Question 1: What is the secret message in Santa's tweets?
{: .notice--danger}
Question 2: What is inside the ZIP file distributed by Santa's team?
{: .notice--danger}

In order to answer these questions, you first had to know a little information about Santa.  Luckily, all the information needed was printed on his business card discovered by Josh and Jessica Dosis.

![Santa's Business Card](/assets/images/sans2016/santa_business_card.png)

Per the card, Santa is [@santawclaus](https://twitter.com/SantaWClaus) on twitter.  Let's check out his account.

![Santa's twitter](/assets/images/sans2016/santa_twitter.png)

Discovery of his hidden message required scraping and saving all his tweets.  To do this, I used a free service called [twlets](http://twlets.com).

![twlets](/assets/images/sans2016/grab_tweets_twitter.png)

I was able to export them to an Excel (XLSX) file. 

An image started to appear in `Column E` but it wasn't clear.  This was fixed by changing to a "Fixed Width" font.  
I chose the appropriately named font "Hack."

![animated_gif](/assets/images/sans2016/santa_tweets.gif)

What is the secret message in Santa's tweets?
**bugbounty**
{: .notice}

Answering the second question required performing some detective work on Santa's [@santawclaus](https://www.instagram.com/santawclaus) instagram account.

The first picture he uploaded contained the required information to download the ZIP file.

![instagram](/assets/images/sans2016/santa_instagram.png)

(enhance...)

Combining the URL on the paper (1) with the file name on the monitor (2) gave the direct location for the ZIP file.

![instagram](/assets/images/sans2016/zip_location.png)

Grab the ZIP file and extract the contents.

```yaml
josh@MacBook-Pro ~/Downloads $ curl -O www.northpolewonderland.com/SantaGram_v4.2.zip
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 1917k  100 1917k    0     0  4678k      0 --:--:-- --:--:-- --:--:-- 4687k
josh@MacBook-Pro ~/Downloads $ file SantaGram_v4.2.zip
SantaGram_v4.2.zip: Zip archive data, at least v2.0 to extract
josh@MacBook-Pro ~/Downloads $ unzip SantaGram_v4.2.zip
Archive:  SantaGram_v4.2.zip
[SantaGram_v4.2.zip] SantaGram_4.2.apk password:
  inflating: SantaGram_4.2.apk
```

What is inside the ZIP file distributed by Santa's team?
**[SantaGram_4.2.apk](/assets/files/SantaGram_4.2.apk)** (MD5 = bdb7ca46ce95e9652616852d7c1cf127)
{: .notice}
