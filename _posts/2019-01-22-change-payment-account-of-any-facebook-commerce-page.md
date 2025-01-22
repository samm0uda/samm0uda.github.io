---
id: 50
title: 'Change payment account of any Facebook commerce page'
date: '2019-01-22T20:49:05+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=50'
categories:
    - Uncategorized
---

This bug could have allowed an user with no role in a page to replace the page shop payment account and receive the its payments.

**Reproduction Instructions:**

1. First the attacker should create a payment account provided by “**2C2P by Qwik**” service which is payment provided for Thailand users. To create the account, we should send a **POST** request to the endpoint https://www.facebook.com/commerce/2C2P/create/ with the parameters as follow:
    - page_id: ID of a page owned by the attacker
    - password: 6 digits password
    - email: Email owned by the attacker which we will use later to verify the account
    - name, business_name: Any random strings
    - document_id: Verification id corresponding to the selected type of document
    - document_type: thai_national_id
    -

Now a 2C2P account should be created and automatically linked to the page owned by the attacker.   
Next step is to browse to https://www.facebook.com/{attacker_page_id}/settings/?tab=payments and we can see that the 2C2P account is linked to the page  
Then we can verify the account by providing the OTP code received in the email and then adding a bank account to that payment account.  
Please notice that the creation of the account is done this way because the option to select this payment provider (like selecting Paypal or Stripe) to receive money is only available in some countries only.

2\. Now after we have valid credentials and verified payment account, we unlink the created account from our page. (The account still exists after we unlink it and we will use it later to connect it to the targeted page).   
To unlink it, we browse to https://www.facebook.com/{attacker_page_id}/settings/?tab=payments

3.The final step is to link the created payment account to the targeted Facebook page. To accomplish that we send a POST request to https://www.facebook.com/commerce/2C2P/connect/ with the following parameters.

- page_id: ID of the Targeted page
- email: Email of the account created earlier
- password: Password of that account
- 

As a response we get an error “**A problem occurred when trying to complete your request. This can be caused by entering incorrect information or trying to complete an unauthorized action**” but the request is completed and the account is now linked to the page and the previous payment method selected by the admin is removed.  
At this point, the attacker should automatically receive incoming payments because he has already added a bank account to his payment account so no more configuration are needed by the victim.

**Impact**  
This bug is critical because the actual payment method (Paypal or Stripe) of the victim page is automatically replaced by the 2C2P account added by the attacker and no interaction from the page admin is needed .  
Also the attacker can see the page commerce payments in the Qwik by 2C2P website after logging in with his payment account credentials.

**Timeline**  
  
Mar 20, 2018 — Report Sent  
Mar 21, 2018—Acknowledged by Facebook  
Apr 10, 2018 — Fixed by Facebook  
May 8, 2018 — Bounty Awarded by Facebook