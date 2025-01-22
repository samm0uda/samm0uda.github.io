---
id: 272
title: 'Disclose files content from Facebook internal CDNs'
date: '2019-05-25T18:16:01+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=272'
categories:
    - Uncategorized
---

This bug could have allowed an attacker to download files which have been uploaded previously by employees or normal users to Facebook platforms. This is done by exploiting an endpoint which takes an encrypted string representing the file to pull.

**Description**  
The selection of the file to download is done by specifying an encrypted string which i was able to decode and modify it to point to any file of my chose. This encryption algorithm is widely used in Facebook platforms and fixing is still in progress for other endpoints so i’m not going to discuss how i broke it until a fix is pushed to all potential vulnerable endpoints .   
  
**Demonstration**  
  
The domain range **scontent.\*.fbcdn.net** represents Facebook public CDNs which hosts all user’s uploaded content like photos/videos/files and more.  
Let’s take a look on an example file hosted in these CDNs :  
**https://scontent.ftun12-1.fna.fbcdn.net/v/t39.2365-6/21276262_1737282336573228_8492096834724954112_n.jpg?_nc_cat=108&amp;oh=4e4c1ebd627f4a10dbbb3dd05a898499&amp;oe=5D665C03**   
To access the file in this link, you should have a valid URL signature which is ensured by the “**oe**” and “**oh**” parameters. This signature is changed each period of time or based on a privacy change of the file. This security measurement is used to prevent from scraping the CDN and downloading content you don’t have permission to see.  
The endpoint **https://lookaside.facebook.com/redrawable/** allows mobile users to get updated applications icons by selecting an encrypted string that points to the file id **21276262_1737282336573228_8492096834724954112** then it would grab its content from the CDN and return it to the user. Because i knew how the encrypted string is generated i changed the file id to another one ( a video for example) and i got the content in the response.   
  
The same method worked with some modifications to retrieve files from internal Facebook CDNs ( **interncache-\*.fbcdn.net** for example which is not publicly accessible ). This allowed me to get some very serious files like partial source code and application crash logs of Facebook mobile users.  
  
The ids of the internal files could be collected by looking in commits to Facebook open source projects or by looking in old versions of Facebook different mobile applications.  
More details about this bug and bugs linked to it will be published soon once they’re all fixed  
  
  
**Timeline**  
Mar 26, 2019 — Report Sent  
Apr 15, 2019— Acknowledged by Facebook  
Apr 16, 2019— Fixed by Facebook  
Apr 29, 2019 — 12500$ Bounty Awarded by Facebook