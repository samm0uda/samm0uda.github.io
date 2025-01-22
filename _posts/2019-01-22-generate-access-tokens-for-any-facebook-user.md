---
id: 35
title: 'Generate Access Tokens for any Facebook user'
date: '2019-01-22T13:58:48+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=35'
categories:
    - Uncategorized
---

This bug could allowed a malicious user to generate access tokens for any Facebook user.  
I found this bug by mistake when I was testing some Facebook endpoints used in the Rights Manger dashboard which is a dashboard targeting videos’ publishers and editors.

The vulnerable endpoint returns a page access_token when making a POST request to it along with the parameter **page_id.** The issue here is that theendpoint doesn’t check if the provided value for the page_id is actually an id of a “**page**” and not another object like “**user**”. This allowed me to make the request and change the page_id value to any Facebook user id and as a response to this request I get the access token of that user.

<figure class="wp-block-image is-resized">[![](http://127.0.0.1:4000/assets/img/2019/01/1_-4LhCPJ07BYQlLlv4dD6xA.png)](http://127.0.0.1:4000/assets/img/2019/01/1_-4LhCPJ07BYQlLlv4dD6xA.png)  
---

**Impact**  
Due to the state of the Access Token (The scopes of the generated access_token are for pages and not users), I wasn’t able to read and modify some data about the user (like see messages) and I wasn’t able to full takeover the account. Nevertheless, I was able to read all private information like emails , credit cards, phones number , managed pages and their access_tokens , managed business/ad-accounts and private posts,photos and videos ….

---

---

**Fix** The Facebook security team fixed this issue by modifying their APIs to refuse those kind of tokens which are generated this way (user object instead of a page). Also after almost six months, they made a second fix by modifying this endpoint and some others to not generate these types of tokens in the first place by checking if the id provided doesn’t match a user object.

**Timeline**  
Feb 3, 2018 — Report Sent  
Feb 6, 2018 — Further investigation by Facebook  
Feb 6, 2018 – Clarification requested by Facebook  
Feb 6, 2018 — Clarification sent  
Feb 8, 2018 — Fixed by Facebook  
Feb 23, 2018 — Bounty Awarded by Facebook