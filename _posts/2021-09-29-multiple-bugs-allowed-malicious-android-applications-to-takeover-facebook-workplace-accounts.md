---
id: 729
title: 'Multiple bugs allowed malicious Android Applications to takeover Facebook/Workplace accounts'
date: '2021-09-29T22:24:59+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=729'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

These bugs could allow malicious actors who owns Android Applications installed in the victim device alongside Facebook owned Android Applications ( Workplace, Facebook, Messenger .. ) to steal a first-party access token and use it to takeover the user Facebook/Workplace account.

**Details**

First of all, i’d like to point out how these attacks are carried in Android.   
A malicious Android application would first register/add a deep link with a URI scheme, Then we force the Facebook/Workplace application to go through the OAuth flow and instead of redirecting to a website with the access_token, we would redirect to one of the enabled URI schemes ( eg: fb{FB_APP_ID}://authorize , fbconnect://sucess ), the malicious Android Application already registered this URI scheme and would receive the access_token and then use it to takeover the Facebook/Workplace account.   
  
This was the case and it was actually reported before multiple times by me and others so Facebook decided to fix this applying multiple defenses:

- (1) if the attack is carried from inside the Facebook/Workplace application WebView, they whitelisted redirects to certain URIs with specific schemes like fb and fb-work and blocked fbconnect, fb{FB_APP_ID} for example
- (2) If the attack is carried from a mobile browser, they’ll check the user agent and if it’s mobile, they’ll ask for confirmation ( normal behavior is immediate redirect to the URI ). So here, they’re not actually implementing any security solution besides “gambling” on the user not confirming the usage of a popular application like Instagram.   
      
    There’s another possibility to carry such an attack which consists of somehow abusing the Facebook/Workplace activity responsible to authenticate Facebook users who chose to authenticate to an Android application using Facebook. Of course, the third-party application should include the [Facebook Android SDK](https://developers.facebook.com/docs/facebook-login/android/) and use certain APIs. I won’t go into details about the entire flow but i’ll focus on how they ensued security.   
      
    The third-party application using the Facebook Android SDK would end-up creating an Intent with required information as extra info and send it using [`startActivityForResult`](https://developer.android.com/reference/android/app/Activity#startActivityForResult(android.content.Intent,%20int)) to the Facebook Activity handling authentication requests ( eg : com.facebook.katana.ProxyAuth ). This activity use the data and craft a GET request to **https://www.facebook.com/dialog/oauth** asking for an access_token, the redirect uri specified would always be fbconnect://success. It would wait for a redirect to fbconnect://success and if it happens, it would pass the information it got ( access_token ) to the registered callback in the Activity who started the ProxyAuth Activity ( in the third-party Android Application )  
      
    The information sent consists of many things but the most interesting one is the Facebook application ID to be set in the the **client_id** parameter in the request to **/dialog/oauth** .   
      
    The important thing here is how to verify that the Android application specifying this Facebook application ID is actually the owner of it, for example a malicious Android application could specify the Instagram application ID and get an access_token that could be used to takeover the account. Facebook implemented these security solutions which mainly checks in the server-side in the request to **/dialog/oauth**:
- (3) Check for headers and headers values like **X-Fb-Is-Native-Platform-Login** and **X-Requested-With: com.facebook.katana**
- (4) If that’s the case, check for parameters in the requests like **calling_package_key** , **android_key**. These are the main security measurements to protect against Android applications setting Facebook application id that doesn’t belong to them.   
      
    **calling_package_key** is the Android application name which should match the one set while configuring the Facebook application, it would be fetched by the Facebook Android application from the application that started the ProxyAuth Activity.   
      
    **android_key** is a release key hash which is calculated and unique for each Android application ( read [here](https://developers.facebook.com/docs/android/getting-started/#release-key-hash) and [here](https://developer.android.com/studio/publish/app-signing) ). This should be added in your Facebook application configuration. With each call to ProxyAuth activity, it would calculate the key for the Android application that called ProxyAuth and send it in the android_key parameter, in the server-side we would check if this key is actually in the list of trusted keys in the Facebook application we specified its id in **app_id** parameter

  
**1) Malicious Android application could steal a Workplace account first party access_token**  
  
 This bug could allow a malicious Android app installed in the victim phone to gain access to his Workplace account by stealing a first party token. This is possible because the Workplace OAuth flow didn’t implement the measurement (2), so from the malicious Android application we would :

. Register fbWP_APP_id://authorize URI scheme in AndroidManifest.xml, WP_APP_ID is a first-party Workplace application.  
. Open : https://work.workplace.com/dialog/oauth?app_id=WP_APP_ID&amp;redirect_uri=fbWP_APP_id://authorize  
. Receive the access_token an use it to takeover the Workplace account ( more complicated than Facebook account takeover but i won’t mention the steps here )

**2)** **Abusing com.facebook.katana.ProxyAuth activity in Facebook/Workplace Android applications**   
  
The activity **ProxyAuth** is exported and used by Facebook to enable other apps to login with Facebook as i mentioned before. The activity verifies the Android application which called it and append **android_key** parameter to the request to https://m.facebook.com/dialog/oauth after calculating it. The malicious android ( / instant ) application would start this activity and append to it a first party application id in the client_id parameter.   
The problem here is that **ProxyAuth** fails to generate the key hash for the certain Android applications ( best guess here ones unsigned or that didn’t use the Facebook Android SDK and directly started the ProxyAuth Activity) which would result to making the request to **/dialog/oauth** with android_key set to empty. There are different server-side behaviors here,   
if we target Facebook OAuth endpoint https://m.facebook.com/dialog/oauth, reeving an empty **android_key** there would result in a confirmation dialog showing asking if you would like to authorize Instagram application for example ( Same to the old flow of Linking Facebook and Instagram accounts from the Instagram Android Application ).   
The server-side behavior in Workplace.com is different though, providing an empty android_key parameter resulted in a direct redirect to fbconnect://success with an access_token. Here we bypassed (4)

```java
 public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Intent intent  =new Intent()
                .setClassName("com.facebook.katana","com.facebook.katana.ProxyAuth")
                .putExtra("app_id", "124024574287414");

        intent.putExtra("scope", "email,");
        intent.putExtra("e2e", "{}");
        intent.putExtra("return_via_intent_url", "1");
        intent.putExtra("legacy_override", "v2.3");
        intent.putExtra("response_type","token");
        intent.putExtra("return_scopes", "true");
        intent.putExtra("default_audience","friends");
        intent.putExtra("auth_type", "rerequest");
        intent.putExtra("original_redirect_uri", "https://www.instagram.com/accounts/signup/");
        intent.putExtra("source_ref", "FB_BROWSER_IX");
        if (intent != null) {
            startActivityForResult(intent,2);
        }

    }
      @Override
      protected void onActivityResult(int requestCode, int resultCode, Intent data)
      {
          setContentView(R.layout.activity_main);
          super.onActivityResult(requestCode, resultCode, data);
          if(requestCode==2)
          {
              TextInputEditText et = (TextInputEditText) findViewById(R.id.editT);
              String message=data.getStringExtra("access_token");
              et.setText(message);
          }
      }
  }
```

**3) Bypassing Webview blockage of certain URI schemes**

As i mentioned before, Facebook/Workplace Webview blocks redirects to not whitelisted URI schemes . Among the whitelisted URI schemes there are **fb** and **fb-work**. I have managed to exploit a weird behavior in the OAuth flow redirects which bypassed this restriction and i was able to steal a first-party access_token.

Here’s an example of an URL that should be opened by the malicious android application:

```markup
fb-work://faceweb/f?href=https://work.workplace.com/dialog/oauth/?app_id=WP_APP_ID&display=page&redirect_uri=fb://www.workplace.com/dialog/return/arbiter?relation%3Dopener%23origin%3DfbWP_APP_ID%3A%2F%2Fauthorize&scope=openid&response_type=token
```

Notice that redirect_uri is **fb://www.workplace.com/dialog/return/arbiter?relation=opener#origin=fb1665147590233624://authorize**  
  
I found that if we specify **xd_arbiter** or **dialog/return/arbiter** endpoints in redirect_uri and we change the URI scheme ( instead of **https://www.facebook.com/dialog/return/arbiter**, we choose **fb://www.facebook.com/dialog/return/arbiter** ) it would redirect us with the access_token to **fb://www.facebook.com/** with the access_token in this case.   
  
Some of the whitelisted schemes are ( fb-ama , fb-messenger-group-thread , fb-messenger-public , fb-messenger , fb-pma , fb-workchat , fb1196383223757595 , fb124024574287414 , fb124024574287414master , fb124024574287414rc , fb1576585912599779 , fb1775440806014337 , fb236786383180508 , fb929757330408142 , fb , fbatwork , fbcf , fbconnect , fblite , fbrpc , fbstaging , ftp , itms-apps , itms , market , ms-app , pebblejs , svn%2bssh ) .

This attack would work too with Facebook OAuth.   
In order for this attack to work, the malicious app should register the **“fb**” URI scheme if it’s going to target a Workplace application. So for the sake of the attack to be without user interaction, we’ll suppose that the victim only has Workplace app installed and not Facebook ( so he won’t be promoted to selected which app should handles the **fb** deep link). The Workplace WebView would redirect to **fb** deep link and the malicious app would catch the token.  
  
The same attack could be launched against Facebook application but we would register this time the “**fb-work**” URI scheme in the malicious app and select **fb-work**://www.workplace.com for redirecting ( also here we assume that only Facebook is installed and not Workplace).

**Reporting to Facebook**

To this date, this was my worst experience with the the Facebook bug bounty program and made me decide to quit the program for a while. So here’s how it went:  
  
First they accepted all the bugs as valid and triaged all of them. I initially reported bug 3) then i found 1) and decided to include it in the same report to address them quicker. I reported 2) separately.   
  
After more than 2 and a half month, i got payed for only one bug in the first report, i didn’t argue with them about this since it was actually my fault trying to make things easier for them.   
  
Few days later, i got a reply to the second report ( about bug 2) ), they said this was actually known internally and security problems surrounding fbconnect://success when used as a redirect URI. The funny thing here is that me and some other researchers are the ones who discovered the issues about fbconnect://success in Mobile and reported bugs related to it. It was obvious and known for me from experience with older bugs that this was nothing related to fbconnect://success but about how Facebook/Workplace applications failing to calculate a correct hash key for the malicious Android application and supplying an empty android_key.  
   
I argued with them about this in length and eventually they said that this was actually a duplicate to the first report since Workplace.com OAuth didn’t have any security measurements about attacks targeting Mobile users at all. BUT wait, there are two bugs in the first report.  
   
Long story short, Facebook marked 2) as a duplicate of 1) ( which in itself doesn’t make sense to me since these are two separate issues from my experience ) but besides that, they only paid for bug 3) in the first report and didn’t pay for bug 1) since it was included in the same report ( but used it to mark 2) as duplicate ).  
  
I wasn’t even paid for two, i was paid for one! ( I’m sure about this since a typical bug like this that resulted in an account takeover in mobile with the help of a malicious Android application would receive a 10,000$ bounty and i only received 10,000$ )

**Timeline**

Report 1:

1. Oct 24, 2020— Report Sent including Bug 3)  
    Nov 10, 2020— More information sent and Bug 1)  
    Feb 4, 2021 — $10,000 bounty awarded by Facebook.  
    Sep 20, 2021— Fixed by Facebook
2. Nov 14, 2020— Report Sent including Bug 2)  
    Nov 17, 2020— Acknowledged by Facebook  
    Feb 2, 2021 — Closed as Duplicate.  
    Sep 20, 2021— Fixed by Facebook