---
id: 636
title: 'Disclose internal CMS objects content'
date: '2021-02-18T08:30:26+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=636'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug was found in the CMS WYSIWYG tool which is used to create CMS objects internally and was implemented recently with limitation in Facebook Workplace exactly in the “Knowledge Library” tool.   
The bug i found was about the possibility to reference other CMS object inside the CMS object we’re creating which would leak the its content. Another scenario is the ability to use other CMS nodes which are only used to create Facebook CMS objects and which are not available in the Knowledge Library editor. Example nodes would be “viewer-name” which would return viewer name in the document. Others nodes are possible to include video/asset/gatekeeper/faq. More about these nodes were reported that resulted to serious bugs being found ( i’ll write about them soon )

**Reproduction Steps**

Setup  
===  
Create a Facebook Workplace account and then visit as Admin,  
**https://TESTING.workplace.com/work/knowledge/** Run a proxy like Burpsuite to intercept requests made

Steps  
==  
1\) Create a Category and then select “Blank page” and then click on “Publish” located in the top-right corner of the page.

2\) If you check your Burpsuite history tab, you would find a request made to **/api/graphql/** endpoint with doc_id=2674517216004073. We’re interested in the value (id) of “document” key inside variables parameter.

3\) Send a POST request to **https://TESTING.workplace.com/api/graphql/** with a logged-in user cookies and those parameters in the request body:

fb_dtsg=YOUR_CSRF_TOKEN&amp;  
__a=1&amp;  
doc_id=3078946742160079&amp;

variables={“input”:{“client_mutation_id”:”23″,”actor_id”:”ACTOR_ID”,”xml”:”XXX”,”document”:”DOC_ID”,”status”:”DRAFT”}}

where ACTOR_ID is your user_id and DOC_ID is the document id you got from step 2)

XXX is the content of the CMS object. To test for the internal CMS objects content leakage, you can put this content (thanks @phwd):

&lt;palette xmlns:palette-work-knowledge-toolkit=\\”palette-work-knowledge-toolkit\\”&gt;  
&lt;cms-ref id=\\”**REDACTED**\\”&gt;&lt;/cms-ref&gt;  
&lt;/palette&gt;

**Impact**

An attacker could use this to disclose internal CMS objects like for example hire documents or could be used to create forms or tags that may lead to launch CSRF attacks.

**Timeline**

May 4, 2020— Report Sent   
May 7, 2020— Acknowledged by Facebook  
May 13, 2020— Fixed by Facebook  
May 14, 2020 — $500 bounty awarded by Facebook.