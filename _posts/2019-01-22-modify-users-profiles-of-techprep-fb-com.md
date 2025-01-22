---
id: 30
title: 'Modify users profiles of techprep.fb.com'
date: '2019-01-22T12:46:56+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=30'
categories:
    - Uncategorized
---

This bug reported to Facebook could allowed an attacker to read the list of all Facebook users who signed up in https://techprep.fb.com/ then he could modify the content of any profile linked to the Facebook user.

**Reproduction Instructions / Proof of Concept**

1-Signup in [techprep.fb.com](http://techprep.fb.com/) by using my Facebook account.

2-After logging in,When navigating to my profile,([https://techprep.fb.com/profile/PTqqSXXXXX/](https://techprep.fb.com/profile/PTqqSChCNX/)) an api call to [api.techprep.fb.com](http://api.techprep.fb.com/) is made requesting user’s data.  
The request is captured by burp .

[![](http://127.0.0.1:4000/assets/img/2019/01/1_PQWEAqLw2XCIo9L7tFh_5Q.png)](http://127.0.0.1:4000/assets/img/2019/01/1_PQWEAqLw2XCIo9L7tFh_5Q.png)3-I replay the same request from the previous step but i remove the “where” and the “limit” from the body , and as a respond i get all users profiles options (name, user.objectId ,facebookId ,tp_resource_competency, tp_target)

[![](http://127.0.0.1:4000/assets/img/2019/01/1_Rsom8vVhh_xjnwtmsUfy8w.png)](http://127.0.0.1:4000/assets/img/2019/01/1_Rsom8vVhh_xjnwtmsUfy8w.png)(my interest from this request is to gather all profile ids (noted “user.objectId”)

*In my Example:* (the user.objectid is PTqqSxxxxX (“user”:{“objectId”:”PTqqSxxxxX”}) is the same id in the profile url the [http://techprep.fb.com/profile/PTqqSxxxxX](http://techprep.fb.com/profile/PTqqSChCNX))

4-In my profile,I change the age from (12–17) to (8–11) (example change to make the request of modifying)  
I intercept the request (Picture 3) ( <http://api.techprep.fb.com/parse/classes/Profile/TfluqwN0cT> )  
Then i replace “tp_resource_age”:”8–11″ with user”:{“__type”:”Pointer”,”className”:”_User”,”objectId”:”thqsXXXX4i”}

[![](http://127.0.0.1:4000/assets/img/2019/01/1_32nxtz-KBDI5Iym5tMRLzA.png)](http://127.0.0.1:4000/assets/img/2019/01/1_32nxtz-KBDI5Iym5tMRLzA.png)thqsXXXX4i is the user.objectid (profile id) of the victim

Now we visit the victim profile and we notice that it’s showing our profile and the victim’s profile is gone.  
In this case the victim can’t do anything (he can’t update his age or his account privacy settings “to private or public”)

**Impact**

To conclude, this bug allowed people to edit users profiles with more permissions than expected.

**Timeline:**

Dec 21, 2017 — Report Sent  
Jan 2, 2018 — Further investigation by Facebook  
Jan 22, 2018 — Fixed by Facebook  
Jan 24, 2018 — Bounty Awarded by Facebook