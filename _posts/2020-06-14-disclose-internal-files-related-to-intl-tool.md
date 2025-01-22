---
id: 450
title: 'Disclose internal files related to testing of some Facebook tools'
date: '2020-06-14T19:14:31+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=450'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow an attacker to disclose a list of videos/photos/files which includes internal information regarding testing of certain Facebook apps/tools. This information is disclosed through a dashboard named “Media Repository” which is by my understanding restricted to Intl translation employees but somehow a gatekeeper wasn’t set properly to avoid normal users from seeing it. Along with the information disclosed in the files, other information were disclosed in the source of the page like Intl folders IDs and group media IDs.

**Reproduction Steps**

Visiting **https://www.facebook.com/translations/media/** would return a dashboard which contains sections where each one contains a list of files that could be downloaded based on the Facebook project we choose (Instagram, Oculus or Portal). Also, more valuable information could be disclosed in the page source.

**Impact**

This would allow a malicious actor to view ongoing projects and testing cases and errors for certain Facebook tools.

**Timeline**

May 5, 2020 — Report Sent  
May 8, 2020 — Acknowledged by Facebook  
May 15, 2020 — Fixed by Facebook  
May 21, 2020 — Bounty awarded by Facebook