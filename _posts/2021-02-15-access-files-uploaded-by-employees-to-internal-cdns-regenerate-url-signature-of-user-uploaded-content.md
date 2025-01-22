---
id: 606
title: 'Access files uploaded by employees to internal CDNs / Regenerate URL signature of user uploaded content.'
date: '2021-02-15T08:27:01+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=606'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow an attacker to download files which have been uploaded previously by employees or normal Facebook users. The vulnerability allows the attacker to download files from the CDN **scontent.{\*}.fna.fbcdn.net** and also from internal CDNs like **interncache-cln.fbcdn.net** which can’t be accessed by a normal user even with a valid file URL (points to local ips)

The selection of the file to download is done by specifying an encrypted string which represent the ids of the file (HANDLE) . I broke the encoding before (More details will be provided in future writeups) which made it easier for me to download users files like privacy_changed/deleted photos by only knowing an old URL.

This is the same bug as [this one](https://ysamm.com/?p=404) which allowed me to download internal files like partial source code, crush log files of Facebook mobile users.

**Impact**

This bug could allow an attacker to regenerate a valid URL signature for user uploaded content (photos/videos/attachments) which is not available anymore or its privacy has been changed. Also this could allow him to see employees uploads hosted in internal CDNs after knowing a previous URL or the HANDLE (which consists of encrypted file ids).

**Reproduction Steps**

Setup  
===  
1\) Go to <https://www.facebook.com/collabsmanager/start/> and apply as a Creator ( your page should meet the standards so you should bypass this or use an eligble page)

2\) After getting accepted, go to <https://www.facebook.com/collabsmanager/> and hit “Login” in the top right corner of the page. You should be redirected to a similar URL: [https://business.facebook.com/collabsmanager/creator/creator_portfolio/{ID}/](https://business.facebook.com/collabsmanager/creator/creator_portfolio/{ID}/)

Steps  
===

1\) Start a proxy like BurpSuite to repeat requests.  
2\) Click on “Edit” in “Introduction” section  
3\) Upload a “Video” by clicking “Add Video”  
4\) After the uploading is complete, Click “Delete” to delete the uploaded video

5\) Now if you go to BurpSuite history, you should have 3 POST requests made to these endpoints in this order:

**https://vupload-edge.facebook.com/ajax/video/upload/requests/start/  
https://vupload-edge.facebook.com/ajax/video/upload/requests/receive/  
https://business.facebook.com/branded_content/creator_profile/pitch_deck_video/async/save/**

6\) Repeat the request to <https://vupload-edge.facebook.com/ajax/video/upload/requests/start/> after changing the “file_extension” parameter in the body to “jpg”

You should get a video_id in the response noted **XXXXXXX**

7\) Replay the request to <https://vupload-edge.facebook.com/ajax/video/upload/requests/receive/> after changing the “video_id” parameter to XXXXXXX

AND changing the parameter “**fbuploader_video_file_chunk**” in the body to a different value as described below:

you should have a value like this one 1:dW5kZWZpbmVk:application/octet-stream:GBapFAFK2bAstIUBAAAAAACthEIbbj07AACa:e:1568148336:ARa5BNCPUwoIKBVqyao

The interesting part of this string is GBapFAFK2bAstIUBAAAAAACthEIbbj07AACa. This the HANDLE that i talked about above. It refers to uploaded files to your CDNS. Now we should change this handle to any other handle that refers to a file we want to download. For demonstration purposes, i’ll use this handle GFxcvkCRANjMRgABANB4MEoAAAAAbnsvAAAB which refers to an internal file (crash logs) :

The final new value should be 1:dW5kZWZpbmVk:application/octet-stream:GFxcvkCRANjMRgABANB4MEoAAAAAbnsvAAAB:e:1568148336:ARa5BNCPUwoIKBVqyao (of course this value is different in your side) . In the response, we should get “start_offset” and “end_offset” values. (irrelevant but indicates that the uploading was successful)  
  
8\) Replay the request to [https://business.facebook.com/branded_content/creator_profile/pitch_deck_video/async/save/](https://business.facebook.com/branded_content/creator_profile/pitch_deck_video/async/save/) after changing “video_id” to XXXXXXX  
  
9\) Visit [https://business.facebook.com/collabsmanager/creator/creator_portfolio/{ID}/](https://business.facebook.com/collabsmanager/creator/creator_portfolio/%7BID%7D/) and you’ll find a broken video player ( internal file pulled is not a video ). If you check the value inside “src” attribute in the source tag, you’ll find an URL to a file in the scontent.\*.fbcdn.net domain. The internal file was copied to another file accessible in a public CDN.

**Timeline**

Sep 6, 2019— Report Sent   
Sep 9, 2019— Acknowledged by Facebook  
Sep 21, 2019— Fixed by Facebook  
Sep 21, 2019 — $12500 bounty awarded by Facebook.