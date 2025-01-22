---
id: 404
title: 'Generate valid signatures for files hosted in Facebook CDNs.'
date: '2020-03-11T19:06:38+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=404'
categories:
    - Uncategorized
---

**Description**

This bug could allow an attacker to get valid URL signature of uploaded files in the Facebook CDN. This happens because a certain Facebook endpoint takes the file URL without the signature parameters and return a valid one with signature parameters after cropping.

**Details**

To those who don’t understand how Faceboook CDN urls works, here is an example URL and explanation of the signature parameters:

https://scontent.ftun3-1.fna.fbcdn.net/v/t39.2365-6/10734332_858877080824977_1850649729_n.jpg?_nc_cat=100&amp;_nc_sid=ad8a9d&amp;_nc_ohc=OqjMAvXzPM4AX-iWuw3&amp;_nc_ht=scontent.ftun3-1.fna&amp;**oh**=eb51bfb048f02e851026515680b1f94b&amp;**oe**=5E922573

The interesting parameters in this URL are **oh** and **oe** which represents the signature to URL and expiration time of this signature. If you try to access the file without those parameters you’ll get “<font size="2">**Bad URL timestamp**</font>” or if you try to access it with wrong signature (or after changing something in the URL) you’ll get “<font size="2">**URL signature mismatch**</font>” ( or in some cases if the signature is valid but expired or if the file is not accessible anymore “<font size="2">**URL signature expired**</font>” ).

By exploiting this bug we can bypass those restrictions by generating valid signature to any file we choose by only supplying the file URL without parameters like this one below:

https://scontent.ftun3-1.fna.fbcdn.net/v/t39.2365-6/10734332_858877080824977_1850649729_n.jpg

**Exploitation**

<font size="2">`POST /ads/tools/text_overlay_validation_with_crops/async/ <br></br>Host: www.facebook.com<br></br><br></br>__a=<strong>1</strong>&<br></br>fb_dtsg=<strong>YOUR_CSRF_TOKEN</strong>&<br></br>image_position_to_crops[0]=<strong>{"url":"<URL> ","width":960,"height":960,"crops":[100,0,526,395]}</strong>`</font>

where &lt;URL&gt; is the URL to a CDN file. The response should return the same URL with the signature parameters added to it. Here is an example of the returned URL:

https://scontent.xx.fbcdn.net/hphotos-xap1/v/t39.2365-6/**s526x395**/10734332_858877080824977_1850649729_n.jpg?**oh**=b1fa2b10cef8aaee779dbac57f23db6f&amp;**oe**=5E812361

The highlighted part shows the cropping options chosen in **crops** field in the request. and the added oh and oe parameters. This may not work sometimes unless we specify valid crops options which are compatible with the supplied image file.

**Impact**

This could be used to access files uploaded by Facebook users which are deleted or their privacy settings have been changed. Also, this could be used to massively download files by searching for URLs with old timestamp and generating new ones. This may also work with files hosted in internal CDNs like interncache-frc.fbcdn.net ( but no way to access the content from outside even with correct signature unless we have an SSRF bug or a way to internally pull the content)

**Timeline**

Feb 4, 2020 — Report Sent  
Feb 18, 2020 — Acknowledged by Facebook  
Feb 26, 2020 — Fixed by Facebook  
Feb 26, 2020 — Bounty Awarded by Facebook