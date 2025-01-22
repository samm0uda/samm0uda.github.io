---
id: 620
title: 'Make recruiting referrals on behalf of employees'
date: '2021-02-17T09:24:49+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=620'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow a malicious user to make recruiting referrals on the behalf of a Facebook employee knowing its Facebook ID. This happens because a certain GraphQL mutation which was doing the referral, was verifying the maker of the request by only checking the actor_id supplied which could be manipulated and set to a different user id like an Employee id which would have the permission to do such action. This was possible by making the GraphQL mutation without cookies (not a logged-in user ). The response to the request would be an error however if we check the email inbox of the email address we supplied in the request , we would find a recruiting referral sent by “Employee name” to the target.

**Reproduction steps**

Send a POST request to <https://www.facebook.com/api/graphql/> without cookies and with this in the body of the request:

doc_id=2203847996377351&amp;

variables={“input”:{“client_mutation_id”:1,”actor_id”:”**EMPLOYEE_ID**“,”name”:”**RECEIVER_NAME**“,”referral_type”:”**REFERRAL**“,”email”:”**EMAIL_ADDRESS**“,”source”:”START_A_REFERRAL”,”referral_sharing_preference”:”NONE”,”actor_type”:”REFERRER”,”submission_status”:”SUBMITTED”,”referrer_id”:”EMPLOYEE_ID”,”preferred_name”:”RECEIVER_NAME”,”links”:\[{“type”:”GITHUB”,uri:”https://github.com/samm0uda”}\],”is_university”:false,”facebook_id”:”**FBID**“,”referral_campaign”:null,”location_ids”:null,”position_ids”:678109866167015,”save_as_test”:false,”total_yoe”:4,”time_spent_in_form”:500,”answers”:{“question_key”:”Q357348757785120″,”answer”:”YES”}}}

Where:

**EMPLOYEE_ID**: The employee FBID ( I used REDACTED as this was found in another bug to be published )  
**EMAIL_ADDRESS**: Target email address  
**RECEIVER_NAME**: Target name  
**FBID**: Target FBID

Here i say target however the attacker can make a recruiting request to himself after choosing the right position and all details.  
This request would return an error after some waiting time. However, if you check your inbox you’ll receive the recruiting email and a link to recruiting preferences also after few days you’ll receive an email from a Facebook employee about the recruiting process.

**Impact**

This could allow a malicious user to send recruiting referrals on behalf of any employee. This could also be used to send emails from “do-not-reply@recruiting.facebook.com“ with limited control over the email body.

**Timeline**

Dec 19, 2020— Report Sent   
Dec 21, 2020— Acknowledged by Facebook  
Jan 11, 2021— Fixed by Facebook  
Jan 29, 2021 — $3000 bounty awarded by Facebook (including bonus).