---
id: 12
title: 'Uploading files to api.techprep.fb.com'
date: '2019-01-22T12:01:10+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=12'
categories:
    - Uncategorized
---

This bug found in one of Facebook domains (api.techprep.fb.com) could allowed an attacker to upload files to the server. All filetypes were supported that could lead to XSS.

**Reproduction Instructions / Proof of Concept**

1-Sign up in [techprep.fb.com](http://techprep.fb.com/)  
2-After logging in, the attacker intercept any request to [api.techprep.fb.com](http://api.techprep.fb.com/)then get the _Applicationid  
3-The attacker make a POST request to [api.techprep.fb.com/parse/files/FILENAME.EXT](http://api.techprep.fb.com/parse/files/FILENAME.EXT) with the header X-Parse-Application-Id:+(_Applicationid)  
and the Content-Type: header then the file content (HTML File or image)  
The respond of the request contains the file path

[![](http://127.0.0.1:4000/assets/img/2019/01/1_Hq7t3QyB7UkdKUXFiIPw4w-1024x502.png)](http://127.0.0.1:4000/assets/img/2019/01/1_Hq7t3QyB7UkdKUXFiIPw4w.png)Request sent using Burp SuiteThis bug is found because the Parse Server was used without disabling the uploading files feature and without implementing ACL when configuring the server.

**Timeline**  
Dec 21, 2017 — Report Sent  
Dec 27, 2017 — Further investigation by Facebook  
Jan 22, 2018 — Fixed by Facebook  
Jan 24, 2018 — Bounty Awarded by Facebook