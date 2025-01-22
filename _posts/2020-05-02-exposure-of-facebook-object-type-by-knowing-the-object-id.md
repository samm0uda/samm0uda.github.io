---
id: 444
title: 'Exposure of Facebook object type by knowing the object ID'
date: '2020-05-02T00:07:03+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=444'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug allows an attacker to expose the entity name or the object type of Facebook objects (including internal objects or ones with no viewing permissions) through errors/exception returned in GraphQL queries when making wrong or malformed Mutations/Queries with different id of an object.

**Details**

Different GraphQL Mutations/Queries were found which could allowed me to to extract the information as described above. However, all of them had the same origin problem which is the lack of exception handling ( data returned ) when ids of unrecognized/unexpected objects were supplied as input variables:

As an example we can try to send a POST request to https://graph.facebook.com/graphql with a first-party access_token and these parameters:

doc_id=REDACTED  
&amp;variables={“cmsIDs”:XXX1840153115444}

The id XXX1840153115444 would be an id of an object that we don’t have viewing permission to see its content or type. Randomly fuzzing the ids would allow us to crawl Facebook objects and get their types. An example of types returned for objects i found are:  
EntAgoraPostCollection  
EntAYMTTip  
EntCatalogItemCategory

**Impact**

All Facebook objects ( users, photos, logs { anything in Facebook infrastructure basically } ) are identified by a unique id. Crawling through these ids and knowing each id type would be a huge security risk because a malicious user could label them after getting a huge dataset and then exploit any potential bug he finds against a specific object type.  
( eg: if an attacker finds an IDOR bug that enables him to read tasks/support/vulnerabilities reports. To do that he needs ids to these reports. A normal approach would be to bruteforce the ids in the vulnerable parameter which would take time and increase the chance of detecting the attack. However, having a list of all Facebook object ids and the corresponding type needed, it would allow him to launch the attack directly against these ids)

**Timeline**

Nov 14, 2019 — Report Sent  
Nov 14, 2019 — Acknowledged by Facebook  
Nov 27, 2019 — Fixed by Facebook  
Jan 21, 2020 — Bounty awarded by Facebook