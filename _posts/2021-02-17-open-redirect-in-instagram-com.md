---
id: 625
title: 'Open redirect in Instagram.com'
date: '2021-02-17T09:49:59+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=625'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow a malicious user to redirect a user from [www.instagram.com](https://www.instagram.com) to any desired website. The endpoint [https://www.instagram.com/accounts/convert_to_professional_account/](https://www.instagram.com/accounts/convert_to_professional_account/) doesn’t check if the redirect_uri supplied is Facebook owned which leave it vulnerable. One thing to note is that for this attack to work without user interaction, the Instagram account should be a business account.

**Reproduction Steps**

Setup  
===  
Login to a business Instagram account in mobile or web.

Steps  
==  
1\) Visit [https://www.instagram.com/accounts/convert_to_professional_account/?redirect_uri=https%3A%2F%2Fevilzone.org](https://www.instagram.com/accounts/convert_to_professional_account/?redirect_uri=https%3A%2F%2Fevilzone.org&fbclid=IwAR3RsgcnmsSrjtQm6GVM5l3DgpdxsD6uhPFtEqnc6-3ISCLGuwa6ui29f0g)

You should be redirected to [https://evilzone.org/](https://l.facebook.com/l.php?u=https%3A%2F%2Fevilzone.org%2F%3Ffbclid%3DIwAR1tIQLNj-QfugwhWLFN13uzVSz_FYZWykri5y5ezybajOlCwPb7iCzAtl8&h=AT1-xR6vm11ZbgHkZlZM-Wl6YtofeHlnWL9mbWDKGkScWc0fpIqY96fUha99aX7IRn4CxP-h_n4m9lyS3f68lIp5dJ6ycmsH9H8fyROybQb8luSHvCiO8BHWHECmMg)

**Impact**

This could be used to redirect Instagram users to malicious websites from inside Instagram website or mobile application.

**Timeline**

Nov 14, 2020— Report Sent   
Nov 16, 2020— Acknowledged by Facebook  
Jan 15, 2021— Fixed by Facebook  
Jan 15, 2021 — $500 bounty awarded by Facebook (including bonus).