---
id: 588
title: 'Expose the email address of Workplace users'
date: '2021-01-03T16:21:55+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=588'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

This bug could allow a malicious user to expose the email address of any workplace user only by knowing his id. This happens because a certain endpoint handling one-time nonce logins would redirect to an URL with the user email address if the nonce provided was expired or invalid. The id of the targeted workplace user should be changed in the request to this endpoint.

**Technical details**

Workplace by FACEBOOK introduced a new login flow where the workplace user would enter his/her email address and receive an email with a one-time login link. The link received in the email would redirect to an endpoint with the workplace user id and the login nonce. If the nonce is expired then the email address of the user would be returned ( the user which its id was supplied in the login link ).

This flow was vulnerable to manipulation and an Insecure direct object reference (IDOR) vulnerability was present in the user id supplied in the link which allowed an attacker to change it and get the email address associated with another user. This happened due to few missing security checking :  
– The nonce value supplied was assumed to be only valid or expired in case it was wrong. In the case of an expired login nonce, the endpoint would return the email address of the user we supplied its id and ask the user to send another one-time login link. No checks were made to make sure that the nonce was actually previously generated in the back-end and then expired and not just invalid.  
– The exploitation of the previously mentioned logic bug could have been avoided by adding another parameter that would contain a hash which would be verified later in the back-end to check for manipulation of parameters names and their values ( instead of saving all previously generated login nonces for a certain user to distinguish later between an expired and a wrong one )

**Reproduction Steps**

1. Visit https://TARGET.workplace.com/
2. Enter any email in the email field an click on “Continue”
3. Check burp history, you should find that a POST request was made to /work/landing/do/new/.
4. Modify the endpoint to /work/signin/magic_link/login/ and add these parameters to the request body nonce=RANDOM&amp;uid=TARGET_ID&amp;request_id=RANDOM where:  
    TARGET_ID: the id of a user in this workplace  
    RANDOM: Any random strings
5. You should get the email address in the Location header in the response.

**Impact**

This could lead to the disclosure of the email address of a user in a workplace if the attacker knew the victim user_id and the workplace he’s in. The information required for the attack were easy to gather since Facebook allows multi-workplace groups to be created which would contain multiple users from different workplaces. Also this could be used in a single Workplace where the attacker is part of the organization and could use the “Directory” feature to get all the users in the workplace.

**Timeline**

Nov 26, 2020— Report Sent   
Nov 27, 2020— Acknowledged by Facebook  
Dec 9, 2020— Fixed by Facebook  
Dec 14, 2020 — $5K bounty awarded by Facebook.