---
id: 609
title: 'Delete linked payments accounts of a Facebook page (or user)'
date: '2021-02-15T08:43:56+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=609'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow a malicious user to delete payment accounts linked to a Facebook page. This bug affects Facebook pages which have payment account linked to them to receive payments from different sources like (Shop, Donations, Fan funding…). These payment accounts from different providers like Paypal, Stripe, Braintree, 2C2P by Qwik, could be deleted.  
Also this bug could maybe affect other payment platform like User payments (Peer to Peer payments) or Fundraiser or Marketplace payments. I didn’t test the bug in these platforms because they are not available in my country but i believe that the attack also works for them.

**Impact**

This bug could be exploited is mass scale against many commerce pages or maybe fundraisers which could lead to inactivity of payments transfers to the owners accounts in the platforms mentioned above.

**Reproduction Steps**

Setup  
===  
To demonstrate the bug, First we need to create a page and link a payment account to it and which we will try to test the attack against. The payment account could be created by linking to a payment provider like Paypal.

Steps  
===

**Link the Paypal account to the Page ( Victim ):**

Send a POST request to the endpoint <https://www.facebook.com/payment/paypalonboard/> with the parameters receiver_id=XXXXX and payment_type=NMOR where XXXXX is the id of an owned page.( parameters fb_dtsg / __a=1 are needed too along with valid cookies)

As response we should get a special link to Paypal. Now we can connect to our Paypal account after we open that link.

After we link the account, if we send another POST request to [https://www.facebook.com/payment/paypal_connect/set_has_paypal_account/](https://www.facebook.com/payment/paypal_connect/set_has_paypal_account/) with the same parameters as before we should get {“has_onboarded”:true} in response.

This means that the payment account is linked. (Also you can check that the Paypal account is linked if you have a US-page and you navigate to [https://www.facebook.com/{page_token}/settings/?tab=payments](https://www.facebook.com/{page_token}/settings/?tab=payments).

**Deleting the account ( Attacker ) :**   
To delete the account, make a POST request to <https://www.facebook.com/api/graphql/> with the parameters doc_id=1235989003182816 and variables={“data”:{“client_mutation_id”:1,”actor_id”:YOUR_USER_ID,”receiver_id”:TARGET_ID,”nmor_provider_type”:”TARGET_PROVIDER”}} where:  
YOUR_USER_ID is the id of the account making the request  
TARGET_ID is the id of the victim. It could be an id of a user , fundraiser or a Page in our case.  
TARGET_PROVIDER is the name of the provider which could be as follow:  
  
TOKEN_BRAINTREE, TOKEN_DISCOVER, TOKEN_STRIPE, PAYPAL, TWO_C_TWO_P, STRIPE, NON_NATIVE, BANK_ACCOUNT  
In our case it’s PAYPAL.

As response we should get {“viewer”: {“is_fb_employee”: false}} and if we make the request again it should return an error which means that the payment account from this provider (PAYPAL) doesn’t exist anymore.   
  
Also if we send again the POST request to [https://www.facebook.com/payment/paypal_connect/set_has_paypal_account/](https://www.facebook.com/payment/paypal_connect/set_has_paypal_account/) we should get a {“has_onboarded”:false} and the Paypal account is not visible anymore in the payments page of the Facebook page.

**Timeline**

Dec 16, 2018— Report Sent   
Dec 19, 2018— Acknowledged by Facebook  
Jan 31, 2019 — $1000 bounty awarded by Facebook.  
Mar 5, 2019— Fixed by Facebook