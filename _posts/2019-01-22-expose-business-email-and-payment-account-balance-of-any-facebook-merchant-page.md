---
id: 45
title: 'Expose business email and payment account balance of any Facebook commerce page.'
date: '2019-01-22T14:51:14+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=45'
categories:
    - Uncategorized
---

This bug when exploited could allowed a malicious user to see business email and account balance of any shop page that uses “2C2P by Qwik” as a payment provider.   
This could affected a limited number of pages because this payment provider is available to use only to Thailand users.

**Reproduction Instructions**   
Send a GET request to this endpoint https://www.facebook.com/commerce/contact_merchant_onboard_status/ with the parameter:

- page_id: this should contain the id of the targeted page

A successful attack returns the page 2C2P details like the business email ( the email chosen by the user when creating the 2C2P account ) and the account current balance.  
Please notice that the same endpoint is used to return these information for other payment providers used by the admin (Paypal or Stripe) but the attack only worked for 2C2P provider.

**Impact**  
The account balance should be only known by admins of the page and also the email which could be used to takeover the payment account (This email is used to login to the payment provider dashboard).

**Timeline** Mar 17, 2018 — Report Sent  
Mar 20, 2018—Acknowledged by Facebook  
Apr 10, 2018 — Fixed by Facebook  
May 16, 2018 — Bounty Awarded by Facebook