---
id: 437
title: 'Add draft subtitles to any Facebook video and Full Path Disclosure'
date: '2020-05-02T00:06:08+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=437'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could have allowed an attacker to add draft subtitles to any video (private or public), or enable automatically generated captions for any video. Add to that, the same endpoint had another bug regarding exception handling which lead to the disclosure of internal paths and functions.  
  
**Details**

1\) Using this script after adding a correct Facebook CSRF token in “fb_dtsg” parameter and the targeted video id in “video_id” parameter:

`<html><br></br><body><br></br><form action="https://www.facebook.com/video/captioneditor/captionupload/?video_id=XXXX&__user=0&__a=1&fb_dtsg=YYYYY" enctype="multipart/form-data" method="post"><br></br><input type="file" name="captions_file0" required /><br></br><input type="submit" value="submit" /><br></br></form><br></br></body><br></br></html>`

( Make sure your srt file is valid and named in this format filename.es_LA.srt )

After the submission, you should get this message **“Unable to Upload Caption”,”errorDescription”:”We are unable to upload your SRT file at this time. Please try again later”**

The message says that the upload failed however if you try to submit the request again you’ll get a different error: **“Unable to Add Caption”,”errorDescription”:”You have already added a caption file for fr_FR. Please remove the older caption file before adding a new one.”** which indicates that the upload was successful.

Another way to check if the subtitle was added is trying to edit the targeted video ( using the owner account ) and under “Subtitles &amp; Captions (CC)”, you should see the new added subtitle.  
  
2\) Another endpoint was vulnerable which caused the same thing to be done but in a limited way:  
Sending a POST request to **https://www.facebook.com/video/captioneditor/speechnonspeech/getresults/** with the parameters video_id=**{target_video_id}** and locale=**{subtitle_locale}**.

as a response to this request, you should get **“status”:2,”error”:”speech_non_speech_fallback”** and if you make the request again, you’ll get **“Unable to Add Caption: You have already added a caption file for en_US. Please remove the older caption file before adding a new one.”** which confirms that the subtitle was added. However, this time the subtitle was auto-generated based on the audio in video and not selected by the attacker (we can only select the generated locale) .

3\) Another bug was found in the endpoint ***https://www.facebook.com/video/captioneditor/speechnonspeech/getresults/*** which allowed me to expose internal server paths and functions after raising an exception. This exception is raised if we send a POST request to the endpoint and setting the “local” parameter to unrecognized local format ( random string would do the trick ).

**Timeline**

Dec 14, 2019 — Report Sent  
Dec 18, 2019 — Acknowledged by Facebook  
Jan 6, 2020 — Fixed by Facebook  
Feb 6, 2020 — Bounty awarded by Facebook

</body></html>