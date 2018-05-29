---
title:  "Part 2: Awesome Package Konveyance"
search: true
excerpt: "SANS Holiday Hack 2016 Writeup - The Challenges - Part 2"
header:
  #image: /assets/images/foo-bar-identity.jpg
  teaser: /assets/images/sans-2016.png
categories:
  - SANS
  - Holday Hack
  - writeup
classes: wide
last_modified_at: 2016-03-06T19:56:50+01:00
---

Question 3: What username and password are embedded in the APK file?
{: .notice--danger}
Question 4: What is the name of the audible component (audio file) in the SantaGram APK file?
{: .notice--danger}

Now that the APK file has been obtained, the next step is to extract the contents.  To do this, I used [apktool](https://ibotpeaches.github.io/Apktool/) to decode the resources within the APK file itself.  

```sh
josh@MacBook-Pro ~/HolidayHack2016 $ java -jar apktool_2.2.1.jar d SantaGram_4.2.apk
I: Using Apktool 2.2.1 on SantaGram_4.2.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
I: Loading resource table from file: /Users/josh/Library/apktool/framework/1.apk
I: Regular manifest package...
I: Decoding file-resources...
I: Decoding values */* XMLs...
I: Baksmaling classes.dex...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...

josh@MacBook-Pro ~/HolidayHack2016 $ ls -la
total 18040
drwxr-xr-x    5 josh  staff      170 Dec 27 21:07 .
drwxr-xr-x+ 116 josh  staff     3944 Dec 27 21:07 ..
drwxr-xr-x    8 josh  staff      272 Dec 27 21:07 SantaGram_4.2
-rw-r--r--    1 josh  staff  2257390 Dec 27 21:06 SantaGram_4.2.apk
-rw-r--r--@   1 josh  staff  6972627 Dec 27 21:07 apktool_2.2.1.jar
```

A new directory was created (SantaGram_4.2) containing all the files extracted from the APK.  I did a case insensitive (-i), recursive (-r), `grep` command to find text that matches `username` in the SantaGram_4.2 folder.  Since I don't know if the username and password will be on the same line, using the `-A10` flag will display 10 lines after the match, and `-B10` flag will display 10 lines before the match. This will give a little buffer to manually search within.  

```yaml
josh@MacBook-Pro ~/HolidayHack2016 $ grep -i -r -A10 -B10 'username' SantaGram_4.2
--
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-.end method
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-.method public static a(Landroid/content/Context;Ljava/lang/String;)V
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-    .locals 4
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-    new-instance v0, Lorg/json/JSONObject;
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-    invoke-direct {v0}, Lorg/json/JSONObject;-><init>()V
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-    :try_start_0
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali:    const-string v1, "username"
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-    const-string v2, "guest"
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-    invoke-virtual {v0, v1, v2}, Lorg/json/JSONObject;->put(Ljava/lang/String;Ljava/lang/Object;)Lorg/json/JSONObject;
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-    const-string v1, "password"
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-    const-string v2, "busyreindeer78"
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-
SantaGram_4.2/smali/com/northpolewonderland/santagram/b.smali-    invoke-virtual {v0, v1, v2}, Lorg/json/JSONObject;->put(Ljava/lang/String;Ljava/lang/Object;)Lorg/json/JSONObject;
--
--
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-.end method
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-.method private postDeviceAnalyticsData()V
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-    .locals 4
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-    new-instance v0, Lorg/json/JSONObject;
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-    invoke-direct {v0}, Lorg/json/JSONObject;-><init>()V
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-    :try_start_0
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali:    const-string v1, "username"
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-    const-string v2, "guest"
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-    invoke-virtual {v0, v1, v2}, Lorg/json/JSONObject;->put(Ljava/lang/String;Ljava/lang/Object;)Lorg/json/JSONObject;
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-    const-string v1, "password"
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-    const-string v2, "busyreindeer78"
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-
SantaGram_4.2/smali/com/northpolewonderland/santagram/SplashScreen.smali-    invoke-virtual {v0, v1, v2}, Lorg/json/JSONObject;->put(Ljava/lang/String;Ljava/lang/Object;)Lorg/json/JSONObject;
--
```

After manually reviewing the output, it looked like there were matches in 2 different files. Having the extra buffer really helped.

Additionally, the URL information for the targets in [Part 4](/challenges/#part-4-my-gosh-it-s-full-of-holes) was discovered within the `./SantaGram_4.2/res/values/strings.xml` file.  The APK file will be revisted again later on.

```yaml
josh@MacBook-Pro ~/HolidayHack2016 $ cat ./SantaGram_4.2/res/values/strings.xml | grep url
    <string name="analytics_launch_url">https://analytics.northpolewonderland.com/report.php?type=launch</string>
    <string name="analytics_usage_url">https://analytics.northpolewonderland.com/report.php?type=usage</string>
    <string name="banner_ad_url">http://ads.northpolewonderland.com/affiliate/C9E380C8-2244-41E3-93A3-D6C6700156A5</string>
    <string name="debug_data_collection_url">http://dev.northpolewonderland.com/index.php</string>
    <string name="dungeon_url">http://dungeon.northpolewonderland.com/</string>
    <string name="exhandler_url">http://ex.northpolewonderland.com/exception.php</string>
```

Question 3: What username and password are embedded in the APK file?
{: .notice}

|Location |File Name| Username      | Password      |
| ------------- | ------------- |:-------------:|-------------:|
|./SantaGram_4.2/smali/com/northpolewonderland/santagram/|b.smali		  | guest         | busyreindeer78|
|./SantaGram_4.2/smali/com/northpolewonderland/santagram/|SplashScreen.smali | guest         | busyreindeer78|


Using another `grep` command, we'll be able to find the audio file too.  When I think of audio formats, MP3 is arguably the most popular format, so let's see if there are any MP3 files in the extracted SantaGram_4.2 directory.

```yaml
josh@MacBook-Pro ~/HolidayHack2016 $ grep -ir 'mp3' SantaGram_4.2
SantaGram_4.2/original/META-INF/CERT.SF:Name: res/raw/discombobulatedaudio1.mp3
SantaGram_4.2/original/META-INF/MANIFEST.MF:Name: res/raw/discombobulatedaudio1.mp3
```

There you have it.  The location of the first audio file flag has been discovered.

Question 4: What is the name of the audible component (audio file) in the SantaGram APK file?
{: .notice}

|Location| File Name | MD5|
|:------------- |:-------------:|:-------------:|
| ./SantaGram_4.2/res/raw/  | [discombobulatedaudio1.mp3](/assets/files/discombobulatedaudio1.mp3) | b7aca2f218c39b997bfd61b83856aed2|

