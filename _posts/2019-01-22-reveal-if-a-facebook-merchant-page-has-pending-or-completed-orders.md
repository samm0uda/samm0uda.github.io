---
id: 42
title: 'Reveal if a Facebook merchant page has pending or completed orders.'
date: '2019-01-22T14:36:16+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=42'
categories:
    - Uncategorized
---

This bug could be used by a malicious user to determine if a certain Facebook merchant page has pending or completed orders at the current time. Only admins of the page should know this because it can expose the merchant activity.

To exploit this bug, the attacker should send a **POST** request to the endpoint https://www.facebook.com/commerce/merchant/orders/download/ (it’s originally used to download the orders list) with the parameters as follow:

- merchant_settings_id **:** this is a unique id for each page which we can get by sending a simple GraphQL query as follow : https://graph.facebook.com/graphql?access_token=&amp;q=**node(target_page_id){preferred_merchant_settings{id}}** (big thanks to [phwd](https://philippeharewood.com/deactivate-facebook-page-shop-as-an-analyst/))
- start_date &amp; end_date : these parameters should specify the period of time we want to look in. (in epoch format)
- order_states : this value of this parameter could be “**70**” which reefers to competed orders or “**78**” for pending orders.
- 

The response to this request reveals if the page has orders or not :

- If the response contains an empty **CSV** file with columns like Order ID,Date,Buyer,Price,Tax,Shipping… then the merchant has no orders.
- if the response status is 500 Error then the merchant has pending or completed orders in the specified period of time.

**Timeline**  
  
Mar 17, 2018 — Report Sent  
Mar 20, 2018—Clarification requested by Facebook  
Apr 5, 2018 — Clarification sent  
Apr 13, 2018 — Acknowledged by Facebook  
Jun 19, 2018 — Fixed by Facebook  
Jun 20, 2018 — Bounty Awarded by Facebook