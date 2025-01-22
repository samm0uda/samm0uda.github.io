---
id: 642
title: 'Expose Facebook object type (including private objects)'
date: '2021-02-18T08:53:19+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=642'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow an attacker to disclose the object type of a Facebook object ID supplied. This works for all objects supported by the Facebook API ( depending on the application which generated the access token too). This would also bypass any permission policies to view this object which means private objects like photos/videos … will work.

**Impact**

Due to the ability to brute-force the ids fast, this could allow an attacker to generate a dataset which contains a pair of Facebook object id and object type. Having that, he can label these objects for future attacks. ( eg: the attacker finds a bug that can get private photos, he can directly exploit this with the ids he has that match the object Photo and which are private or public)

**Reproduction Steps**

Setup  
===  
1\) For this bug, it’s preferred to use a test account because the test account doesn’t have viewing capabilities for objects like pages/photos/users in the main Facebook. However, using this bug this could be bypassed which means that the bug works.

2\) It’s preferred to use a first-party access_token in the bruteforcing below since it doesn’t have number of requests limitations.

Steps  
==  
1\) Visit [https://graph.facebook.com/v7.0/schema/\*XXXX](https://graph.facebook.com/v7.0/schema/*XXXX/)[/..](https://graph.facebook.com/v7.0/schema/*XXXX/..) ( dots are included)

where **XXXX** is the Facebook object id.

You should get “message”: “No such class: OBJECT_TYPE” where **OBJECT_TYPE** is the type of the object we supplied its id before

(PS : The dots can be any string. )

Please note that these ids will return null if we query them using [graph.facebook.com/graphql?q=node(ID){__typename}](http://graph.facebook.com/graphql?q=node%28ID%29%7B__typename%7D)

Some of the objects types i got:

AdAccountActivity, AdCampaignActivity, AdgroupActivity, AdReportRunV2, Album, AppLinkHost, AudioPublishingRightsData, BusinessPersona, CatalogItemOverride, CommerceMerchantSettings, ContactField, ContextualProfileBase, CrisisUserInfo, CTCert, CustomDataProperty, DeviceNotification, DynamicPriceConfigByDate, FileDescriptor

**Timeline**

May 22, 2020— Report Sent   
May 28, 2020— Acknowledged by Facebook  
Jun 10 , 2020— Fixed by Facebook  
Jun 11, 2020 — $500 bounty awarded by Facebook.