---
layout: post
title:  "Why So Paranoid"
date:   2015-11-03 20:14:00 +0530
categories: analysis
---

### The Philosophy

It is a well known fact in information security as well as the developer community that modern software especially something as personal as a smart phone collects a whole bunch of information. The guys at [Wikileaks](http://www.wikileaks.org) played their extra part in *highlighting* the kindness of state sponsored organizations in safe guarding our personal data. While it is not entirely possible for a home user like myself to prevent vendors and OEM to access my personal data if I *must* use their products and technologies, I would at least expect OEM and vendors to follow basic security guidelines and provide me at least a **false sense of security**, if not a choice for security.


### Research

After setting up the [Monitoring Infrastructure]({% post_url 2015-11-04-setting-up-monitoring-infrastructure %}), I observed the device activity for sometime with only a Google Account configured and without installing any additional app. Some of the activities that convinced me to try porting [CyanogenMod](http://www.cyanogenmod.org) for the device:

* IMEI leak over HTTP
* App analytics over HTTP
* Insecure CoolStore Meta Data Transfer
* Insecure OTA Update

#### Insecure CoolStore Update: Remote Pwnage

The Coolpad Note 3 comes pre-installed with a vendor maintained app store called Cool Store. It appears that CoolStore for a specific user is updated based on user's preferences and usage data as collected by the default [XPLOREE](http://www.xploree.com/) keyboard app.

![CoolStore Screen]({{ site.baseurl }}/images/coolstore1.png)

If any of the apps are clicked in the above screen, a new screen is provided with description and a download button. Download and installation appears to be handled by the CoolStore app itself.

However, the interesting part is how these app information is fetched over HTTP:

{% highlight http %}
GET /?user_id=IMEI1,IMEI2&product=ta&model=CP8676_I02&mcc=0&mnc=0&et=1446705228289&eTz=GMT%2B05%3A30&app_version=1.1&country_code=in HTTP/1.1
Host: ta.coolpadvas.net
Connection: Keep-Alive
{% endhighlight %}

Response to the above request:

{% highlight http %}
HTTP/1.1 200 OK
Server: nginx/1.4.6 (Ubuntu)
Date: Thu, 05 Nov 2015 06:33:56 GMT
Content-Type: application/json
Content-Length: 25762
Connection: keep-alive

[...]
{
      "activation_time": "-1", 
      "app_icon": "https://c26-coolpad-resources.s3-ap-northeast-1.amazonaws.com/var/www/c26-ta_data/static_files/apk/com.games2win.ultimateturbocricket4/Ultimate%20turboo%20cricket_300x300.png", 
      "app_url": "https://c26-coolpad-resources.s3-ap-northeast-1.amazonaws.com/var/www/c26-ta_data/static_files/apk/com.games2win.ultimateturbocricket4/utc__coolpad_prod.apk", 
      "banner": "https://c26-coolpad-resources.s3-ap-northeast-1.amazonaws.com/var/www/c26-ta_data/static_files/apk/com.games2win.ultimateturbocricket4/banner1.jpg", 
      "category": "News", 
      "description": "Ultimate Turbo Cricket is the game for Cricket game lovers!\r\nA nanoGame by Games2win! Its just 2.7MB Light in size!\r\nChallenge yourself in 3 different modes: 1 wicket, 3 wickets and 5 wickets.\r\nScore maximum runs in each mode and be on top of each leaderboard.\r\nUltimate Turbo Cricket features:\r\n1) 3 different modes: 1 wicket, 3 wickets and 5 wickets.\r\n2) Varierty of balls to face.\r\n3) 3 Leaderboards to compete for.\r\n4) Smooth and simple gameplay.\r\nAbout Games2win:\r\nWe are a company that believes in creating great fun-filled games for people of all ages. We have more than 800 proprietary games on both online and mobile devices, including our smash hits like Parking Frenzy and Super Mom. Today, we have 51 million downloads of our apps and 5 million gamers a month. And this is just the beginning!\r\nContact us at androidapps@games2win.com for any problems you may have with Ultimate Turbo Cricket.\r\nVISIT US: http://games2win.com\r\nFOLLOW US: http://twitter.com/games2win\r\nLIKE US: http://facebook.com/Games2win", 
      "developer": "Games2win.com ", 
      "download": false, 
      "fc": 0, 
      "ft": [], 
      "install": true, 
      "live": true, 
      "md5": "7e9988aa275aad670d770f9948d76550", 
      "package_name": "com.games2win.ultimateturbocricket4", 
      "premium": true, 
      "process": false, 
      "rating": "4", 
      "screenshot1": "https://c26-coolpad-resources.s3-ap-northeast-1.amazonaws.com/var/www/c26-ta_data/static_files/apk/com.games2win.ultimateturbocricket4/scr1.png", 
      "screenshot2": "https://c26-coolpad-resources.s3-ap-northeast-1.amazonaws.com/var/www/c26-ta_data/static_files/apk/com.games2win.ultimateturbocricket4/scr2.png", 
      "screenshot3": "https://c26-coolpad-resources.s3-ap-northeast-1.amazonaws.com/var/www/c26-ta_data/static_files/apk/com.games2win.ultimateturbocricket4/scr3.png", 
      "size": "2802742", 
      "title": "Ultimate Turbo Cricket", 
      "videoid": "", 
      "wifi": false
    }, 
[...]
{% endhighlight %}

The JSON data fetched in the above request contains description, images and URL for **APK** file for the app. While I have kept this as a future exercise, I think this scenario can definitely be exploited by an attacker who can intercept HTTP request to push malicious non-market apps. In fact this feature can be misused by OEM and their partners to push unexpected apps to the phone in an insecure manner.

#### IMEI Leak Over HTTP

The phone periodically sends an HTTP request as below:

{% highlight http %}
GET /?user_id=IMEI_11111111,IMEI_2222222&android_id=6f426cd22d0698bd&package_name=com.cube26.coolstore&model=CP8676_I02&country_code=in HTTP/1.1
Host: notification.coolpadvas.net
Connection: Keep-Alive
{% endhighlight %}

<small>The IMEI is replaced intentionally in the above request.</small>

The current response indicates that the backend infrastructure for this request is not operational yet:

{% highlight html %}
HTTP/1.1 502 Bad Gateway
Server: nginx/1.4.6 (Ubuntu)
Date: Thu, 05 Nov 2015 06:05:10 GMT
Content-Type: text/html
Content-Length: 181
Connection: keep-alive

<html>
<head><title>502 Bad Gateway</title></head>
<body bgcolor="white">
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx/1.4.6 (Ubuntu)</center>
</body>
</html>
{% endhighlight %}

Device IMEI appears to be leaked over HTTP in another request as below:

{% highlight http %}
GET /analytics?user_id=IMEI1,IMEI2&android_id=6f426cd22d0698bd&eT=1446708920504&eTz=GMT%2B05%3A30&product=ta&app_version=1.1&ver=2&country_code=in&model=CP8676_I02&eventName=downloaded&eventValue=com.app.zorooms HTTP/1.1
Host: ta.cube26coolpad-analytics.net
Connection: Keep-Alive
{% endhighlight %}

#### App Analytics over HTTP

The device periodically sends list of installed app names to a remote *shady* URL over HTTP. The response to this request appears to be a minimal gif file.

{% highlight http %}
POST /c.gif HTTP/1.1
Content-Length: 1353
Content-Type: application/x-www-form-urlencoded
Host: ta.cube26coolpad-analytics.net
Connection: Keep-Alive

user_id=<IMEI1,IMEI2>&product=ta&installed_apps=com.android.calculator2%2Bcom.yulong.android.calendar%2Bcom.android.camera%2Bcom.android.email%2Bcom.android.gallery3d%2Bcom.android.mms%2Bcom.yulong.android.ota%2Bcom.coolpad.music%2Bcom.yulong.android.contacts%2Bcom.yulong.android.videoplayer%2Bcom.mediatek.fmradio%2Bcom.android.settings%2Bcom.android.settings%2Bcom.android.settings.wifi%2Bcom.android.vending%2Bcom.google.android.gms%2Bcom.google.android.talk%2Bcom.google.android.apps.docs%2Bcom.google.android.videos%2Bcom.google.android.youtube%2Bcom.google.android.music%2Bcom.android.chrome%2Bcom.google.android.gm%2Bcom.yulong.android.coolshow%2Bcom.yulong.android.memo%2Bcom.yulong.android.contacts%2Bcom.yulong.android.soundrecorder%2Bcom.cube26.nearestservicecenter%2Bcom.cube26.coolstore%2Bcom.android.stk%2Bcom.yulong.android.filebrowser%2Bcom.yulong.android.fingerprintlock%2Bcom.android.quicksearchbox%2Bcom.yulong.android.xtime%2Bcom.google.android.apps.maps%2Bcom.kpt.xploree.app%2Bin.amazon.mShop.android.shopping%2Bcom.google.android.googlequicksearchbox%2Bcom.google.android.googlequicksearchbox%2Bcn.wps.moffice_eng%2Bcom.tencent.mm%2Bcom.whatsapp%2Bcom.adbdriver.adbtoggle%2Bcom.facebook.katana&model=CP8676_I02&country_code=in&android_id=6f426cd22d0698bd&eT=1446705290499&eTz=GMT%252B05%253A30&app_version=1.1
{% endhighlight %}

#### Insecure OTA Update

The device checks for OTA system update using the following HTTP request:

{% highlight http %}
GET /updsvr/ota/checkupdate?hw=CP8676_I02&hwv=P1&swv=5.1.015.P1.150929.8676_I02&serialno=<SERIAL>&userstart=0&autoupdate=0&curnetwork=0 HTTP/1.1
User-Agent: com.yulong.android.ota/V2.01.35091518_VER_2015.09.21_22:01:50 (Linux; Android)
Content-Type: text/html; charset=utf8
Host: www.51coolpad.com
Connection: Keep-Alive
{% endhighlight %}

The response XML contains appropriate meta-data along with URL for downloading update:

{% highlight http %}
HTTP/1.1 200 OK
Server: nginx/1.7.8
Date: Tue, 03 Nov 2015 07:05:55 GMT
Content-Type: text/html;charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Pragma: No-cache
Expires: Thu, 01 Jan 1970 00:00:00 GMT
Cache-Control: no-cache


2a5
COOLPAD_OTA
CHECK_UPDATE_RESULT:0
<?xml version="1.0" encoding="UTF-8"?><OTAPackage><srcVersion>5.1.015.P1.150929.8676_I02</srcVersion><dstVersion>5.1.019.P1.151101.8676_I02</dstVersion><description>&lt;![CDATA[Enhancing: Camera tuning\nModification: LED notification for third party applications\nEnhancing: Heating issue addressing\nModification: Exchange support]]&gt;</description><downloadURL>http://ota.coolyun.com/ota/Android/CP8676_I02/P1/5.1.015.P1.150929.8676_I02_5.1.019.P1.151101.8676_I02/update_1446480447774.zip</downloadURL><size>24798549</size><priority>Optional</priority><sessionId>a4e2dfa8-9099-4e12-85ed-833081783244</sessionId><md5>null</md5></OTAPackage>

0
{% endhighlight %}

The above HTTP transaction occurs over a plain-text insecure and untrusted channel. The response XML contains a link to the update archive:

{% highlight html %}
http://ota.coolyun.com/ota/Android/CP8676_I02/P1/5.1.015.P1.150929.8676_I02_5.1.019.P1.151101.8676_I02/update_1446480447774.zip
{% endhighlight %}

Luckily the archive appears to be signed. However I am yet to verify if the signature is verified correctly before applying the update.


