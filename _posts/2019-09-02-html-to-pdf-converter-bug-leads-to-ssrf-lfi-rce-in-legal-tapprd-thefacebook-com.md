---
id: 280
title: 'HTML to PDF converter bug leads to RCE in Facebook server.'
date: '2019-09-02T14:56:00+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=280'
categories:
    - Uncategorized
---

**Description**

This bug could have allowed an attacker to execute HTML code in the server-side of the server tapprd.legal.thefacebook.com which would lead to RCE. This happens because HTML tags in filled inputs in the vulnerable page aren’t escaped and passed directly to the “HTML to PDF converter”.  
This converter will parse the HTML tags internally (not including Javascript) and the result is converted to a PDF document which will be emailed later to the attacker.

**Demonstration**

1. When you create a Facebook workplace account, you get an email from **legal_noreply@fb.com**. This email contains a URL to an online agreement that should be signed by the owner of the account.  
    This URL is in this format with a special token : **https://legal.tapprd.thefacebook.com/tapprd/Portal/ShowWorkFlow/AnonymousShowStage?token=**  
    That page contains a long agreement and some inputs that should be filled by the user like Name, Address, Email, Occupation …   
    If you try to inject some HTML code in these inputs, you’ll notice that the application HTML encoded all the text. First, i tried to manipulate the request but unfortunately it was blocked. However, what i noticed later is that the application is HTML encoding the text then decoding it later in the server-side while converting it to the PDF format.
2. The next step is to try to escalate this. As i said Javascript wasn’t an option so i tried to read local files by using the “file://” URL scheme in an IFRAME.  
    Then i scanned the internal network using the same method ( by viewing the converted IFRAME in the PDF file later, you can distinguish the difference from an exiting IP or an open/closed port.
3. At this point i had too many options to escalate this to an RCE:  
      
    – Due to another bug in the server, i was able to get an internal system path of the web application running in the server. Using that i could extract web.config files and try to extract juicy stuff to get more access to the web application.  
      
    – After scanning the local network, i found some WebLogic servers that only accessible internally which may be vulnerable and exploitable.  
      
    – While playing with different URL schemes, i tried the “about://” scheme and what i found in the PDF file a Internet Explorer page listing all preferences and version of IE. I’m not familiar with ASP.NET web applications but what i assumed back then is that the application is opening the HTML page in IE using some kind of Windows API and in that page a Javascript code is included that screenshot/convert the page to PDF format ( **jsPDF** maybe?? ). So what i tried is embedding payloads of some IE exploits ( I can’t disclose full details of this part)

All the attacks mentioned above were not fully launched because of Facebook bug bounty program policy ( and receiving a notice from them to stop the attack)  
  
If you read my previous write-ups, you may find [this bug](https://ysamm.com/?p=308) which has been found in the same server, has the same bug (not escaping HTML tags) which leads to HTML injection in emails.  
  
Facebook responded to this by informing the third-party vendor to fix the bug and push an update. I wasn’t well awarded because the web-app wasn’t coded by Facebook but some of you may know what this server is responsible of and what’s the valuable stuff you may find in it if you access it.

**Timeline**

Apr 7, 2019— Report Sent  
Apr 10, 2019—Clarification requested by Facebook  
Apr 10, 2019 — Clarification sent  
May 1, 2019— More clarification requested by Facebook  
May 1, 2019 — Clarification sent  
May 21, 2019 — Acknowledged by Facebook  
Jun 18, 2019 — Fixed by Facebook  
Jul 3, 2019 — 1000$ Bounty Awarded by Facebook ( -_- )