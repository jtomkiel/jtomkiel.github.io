---
title:  "Terminals"
search: true
excerpt: "SANS Holiday Hack 2016 Writeup - The Terminals"
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

### There are a total of five (5) terminals that you can interact with while playing the [Quest](https://quest2016.holidayhackchallenge.com).

![terminal_icon](/assets/images/sans2016/terminal0_welcome.png)

This year, the [SANS Holiday Hack Challenge](https://www.holidayhackchallenge.com/2016/) included in game "terminals." After collecting the parts below to build a Cranberry Pi and talking with the elf, Holly Evergreen, you can interact with the terminals. 


| SD Card | Pi Board | HDMI Cord | Heatsink | Power Cord
|:------------- |:-------------:|:-----------|:-----------|:-----------|
|![sd_card](/assets/images/sans2016/sd_card.png) | ![pi_board](/assets/images/sans2016/pi_board.png) | ![hdmi](/assets/images/sans2016/hdmi.png) | ![heatsink](/assets/images/sans2016/heatsink.png) | ![power](/assets/images/sans2016/power_cord.png)


To interact with a terminal, simply walk up to one of these ![terminal_icon](/assets/images/sans2016/terminal_icon.png) icons in the game.  You will be greeted with a command prompt on your own private [Docker](https://www.docker.com/) container. This feature was awesome!  Each terminal presented its own challenge. Without further ado, let me walk you through how I completed each one.  


## Tcpdump Challenge

![image](/assets/images/sans2016/terminal_pcap_room.png)

### Challenge Objective: Get Both Parts of the Passphrase

```yaml
*******************************************************************************
*                                                                             *
*To open the door, find both parts of the passphrase inside the /out.pcap file* 
*                                                                             *
*******************************************************************************
```

Let's take a look at the default directory I was are placed in.

```yaml
ls -la
total 1136
drwxr-xr-x  46 root  root     4096 Dec 31 19:55 .
drwxr-xr-x  46 root  root     4096 Dec 31 19:55 ..
-rwxr-xr-x   1 root  root        0 Dec 31 19:55 .dockerenv
drwxr-xr-x   2 root  root     4096 Dec  1 21:18 bin
drwxr-xr-x   2 root  root     4096 Sep 12 04:09 boot
drwxr-xr-x   5 root  root      380 Dec 31 19:55 dev
drwxr-xr-x  46 root  root     4096 Dec 31 19:55 etc
drwxr-xr-x   5 root  root     4096 Dec  7 20:22 home
drwxr-xr-x  10 root  root     4096 Dec  1 21:18 lib
drwxr-xr-x   2 root  root     4096 Nov  4 18:29 lib64
drwxr-xr-x   2 root  root     4096 Nov  4 18:28 media
drwxr-xr-x   2 root  root     4096 Nov  4 18:28 mnt
drwxr-xr-x   2 root  root     4096 Nov  4 18:28 opt
-r--------   1 itchy itchy 1087929 Dec  2 15:05 out.pcap
dr-xr-xr-x 370 root  root        0 Dec 31 19:55 proc
drwx------   2 root  root     4096 Nov  4 18:28 root
drwxr-xr-x   3 root  root     4096 Nov  4 18:28 run
drwxr-xr-x   2 root  root     4096 Nov  4 18:30 sbin
drwxr-xr-x   2 root  root     4096 Nov  4 18:28 srv
dr-xr-xr-x  13 root  root        0 Dec 14 14:13 sys
drwxrwxrwt   2 root  root     4096 Dec  7 20:22 tmp
drwxr-xr-x  15 root  root     4096 Dec  1 21:18 usr
drwxr-xr-x  17 root  root     4096 Dec  2 15:13 var
```

The file `out.pcap` is visible but it is owned by user "itchy".  Our current user is "scratchy" and "scratchy" does not have rights to access it.

The `sudo` command will come in handy here.  I don't know the passwords for either account but let's see what other options there are with `sudo`.

```yaml
sudo -h
sudo - execute a command as another user
usage: sudo -h | -K | -k | -V
usage: sudo -v [-AknS] [-g group] [-h host] [-p prompt] [-u user]
usage: sudo -l [-AknS] [-g group] [-h host] [-p prompt] [-U user] [-u user] [command]
usage: sudo [-AbEHknPS] [-r role] [-t type] [-C num] [-g group] [-h host] [-p prompt]
            [-u user] [VAR=value] [-i|-s] [<command>]
usage: sudo -e [-AknS] [-r role] [-t type] [-C num] [-g group] [-h host] [-p prompt]
            [-u user] file ...
Options:
  -A, --askpass               use a helper program for password prompting
  -b, --background            run command in the background
  -C, --close-from=num        close all file descriptors >= num
  -E, --preserve-env          preserve user environment when running command
  -e, --edit                  edit files instead of running a command
  -g, --group=group           run command as the specified group name or ID
  -H, --set-home              set HOME variable to target user's home dir
  -h, --help                  display help message and exit
  -h, --host=host             run command on host (if supported by plugin)
  -i, --login                 run login shell as the target user; a command may also be
                              specified
  -K, --remove-timestamp      remove timestamp file completely
  -k, --reset-timestamp       invalidate timestamp file
  -l, --list                  list user's privileges or check a specific command; use
                              twice for longer format
  -n, --non-interactive       non-interactive mode, no prompts are used
  -P, --preserve-groups       preserve group vector instead of setting to target's
  -p, --prompt=prompt         use the specified password prompt
  -r, --role=role             create SELinux security context with specified role
  -S, --stdin                 read password from standard input
  -s, --shell                 run shell as the target user; a command may also be
                              specified
  -t, --type=type             create SELinux security context with specified type
  -U, --other-user=user       in list mode, display privileges for user
  -u, --user=user             run command (or edit file) as specified user name or ID
  -V, --version               display version information and exit
  -v, --validate              update user's timestamp without running a command
  --                          stop processing command line arguments
```

Let's see if I can list the privileges that our current user (scratchy) has.

```yaml
sudo -l -U scratchy
sudo: unable to resolve host fd1441e110fc
Matching Defaults entries for scratchy on fd1441e110fc:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin
User scratchy may run the following commands on fd1441e110fc:
    (itchy) NOPASSWD: /usr/sbin/tcpdump
    (itchy) NOPASSWD: /usr/bin/strings
```

The NOPASSWD setting is not great for security as it allows the command to be run without prompting for a password and it runs with the permissions of the owner.  In this case I can run a `sudo` command as "itchy" and have that account read the pcap with `tcpdump` and `strings`.  First, start by telling `tcpdump` to output in ASCII (-A) and read from a file (-r) then finally greping through that output to match words containing "part." I chose to first look for "part" because the welcome banner said "**find both parts** of the passphrase inside the /out.pcap file."

```yaml
sudo -u itchy tcpdump -A -r out.pcap | grep 'part'
sudo: unable to resolve host fd1441e110fc
reading from file out.pcap, link-type EN10MB (Ethernet)
< input type="hidden" name="part1" value="santasli" />
```



Nailed it.  Part 1 of the passphrase is `santasli`. 

Part 2 can be found using `strings` command.  However, just running `strings` will not show the answer.  Let's take a look at what options `strings` has. 

```yaml
strings -h
Usage: strings [option(s)] [file(s)]
 Display printable strings in [file(s)] (stdin by default)
 The options are:
  -a - --all                Scan the entire file, not just the data section [default]
  -d --data                 Only scan the data sections in the file
  -f --print-file-name      Print the name of the file before each string
  -n --bytes=[number]       Locate & print any NUL-terminated sequence of at
  -<number>                   least [number] characters (default 4).
  -t --radix={o,d,x}        Print the location of the string in base 8, 10 or 16
  -w --include-all-whitespace Include all whitespace as valid string characters
  -o                        An alias for --radix=o
  -T --target=<BFDNAME>     Specify the binary file format
  -e --encoding={s,S,b,l,B,L} Select character size and endianness:
                            s = 7-bit, S = 8-bit, {b,l} = 16-bit, {B,L} = 32-bit
  @<file>                   Read options from <file>
  -h --help                 Display this information
  -v -V --version           Print the program's version number
strings: supported targets: elf64-x86-64 elf32-i386 elf32-x86-64 a.out-i386-linux pei-i3
86 pei-x86-64 elf64-l1om elf64-k1om elf64-little elf64-big elf32-little elf32-big pe-x86
-64 pe-bigobj-x86-64 pe-i386 plugin srec symbolsrec verilog tekhex binary ihex
Report bugs to < http://www.sourceware.org/bugzilla/ >
```

By specifying the encoding character size as 16-bit (-el) the cleartext passphase for Part 2 can be seen!

```yaml
sudo -u itchy strings -el out.pcap
sudo: unable to resolve host fd1441e110fc
part2:ttlehelper
```

Complete passphase:  `santaslittlehelper`

The two user accounts were "itchy" and "scratchy", it's nice to see that they kept with [The Simpsons](https://en.wikipedia.org/wiki/The_Simpsons) theme... 

![challenge_coin](/assets/images/sans2016/Coin_tcpdump.png)



## OUTATIME Challenge

![terminal](/assets/images/sans2016/terminal_outtatime.png)

### Challenge Objective:  Ride the Train 

I had the most fun with this challenge.  The terminal opens to a preconfigured menu with limited options.

```yaml
Train Management Console: AUTHORIZED USERS ONLY
                ==== MAIN MENU ====
STATUS:                         Train Status
BRAKEON:                        Set Brakes
BRAKEOFF:                       Release Brakes
START:                          Start Train
HELP:                           Open the help document
QUIT:                           Exit console
menu:main> 
```

Type `HELP` and press enter.

```yaml
Help Document for the Train

**STATUS** option will show you the current state of the train (brakes, boiler, boiler temp, coal level)

**BRAKEON** option enables the brakes.  Brakes should be enabled at every stop and while the train is not in use.

**BRAKEOFF** option disables the brakes.  Brakes must be disabled before the **START** command will execute.

**START** option will start the train if the brake is released and the user has the correct password.

**HELP** brings you to this file.  If it's not here, this console cannot do it, unLESS you know something I don't.

Just in case you wanted to know, here's a really good Cranberry pie recipe:

Ingredients
1 recipe pastry for a 9 inch double crust pie
1 1/2 cups white sugar
1/3 cup all-purpose flour
1/4 teaspoon salt
1/2 cup water 
1 (12 ounce) package fresh cranberries
1/4 cup lemon juice
1 dash ground cinnamon
2 teaspoons butter
:
```

Seeing the `:` at the bottom of the file immediately reminded me of a past SANS blog post about [Escaping Restricted Linux Shells](https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells).  To escape this and access a shell prompt, simply put an exclamation mark (`!`) and press enter.

```yaml
Train Management Console: AUTHORIZED USERS ONLY
                ==== MAIN MENU ====
STATUS:                         Train Status
BRAKEON:                        Set Brakes
BRAKEOFF:                       Release Brakes
START:                          Start Train
HELP:                           Open the help document
QUIT:                           Exit console
menu:main> HELP
sh-4.3$ 
```

Now I have a shell.  A quick `ls -la` will show the other files in the current directory.

```yaml
sh-4.3$ ls -la
total 40
drwxr-xr-x 2 conductor conductor  4096 Dec 10 19:39 .
drwxr-xr-x 6 root      root       4096 Dec 10 19:39 ..
-rw-r--r-- 1 conductor conductor   220 Nov 12  2014 .bash_logout
-rw-r--r-- 1 conductor conductor  3515 Nov 12  2014 .bashrc
-rw-r--r-- 1 conductor conductor   675 Nov 12  2014 .profile
-rwxr-xr-x 1 root      root      10528 Dec 10 19:36 ActivateTrain
-rw-r--r-- 1 root      root       1506 Dec 10 19:36 TrainHelper.txt
-rwxr-xr-x 1 root      root       1588 Dec 10 19:36 Train_Console
sh-4.3$ 
```

To activate the train, all I need to do is run `./ActivateTrain` and I will be transported back to 1978.  

Santa is hiding out in this area too.

![challenge_coin](/assets/images/sans2016/Coin_time_travel.png)

Back to the future!

![challenge_coin](/assets/images/sans2016/terminal_1978.png)

One last thing, if you view the `Train_Console` script, you can see how the game sends a user back in time.

To test if I could send another user back in time, I quickly created a second account, collected the Cranberry Pi parts required and logged into this terminal on a different browser.  Here is the output of the original `Train_Console` script.

```yaml
sh-4.3$ cat Train_Console
```

```yaml
#!/bin/bash
HOMEDIR="/home/conductor"
CTRL="$HOMEDIR/"
DOC="$HOMEDIR/TrainHelper.txt"
PAGER="less"
BRAKE="on"
PASS="24fb3e89ce2aa0ea422c3d511d40dd84"
print_header() {
        echo ""
        echo "Train Management Console: AUTHORIZED USERS ONLY"
        echo ""
}

print_main_menu() {
        echo ""
        echo "                  ==== MAIN MENU ===="
        echo ""
        echo "STATUS:                   Train Status"
        echo "BRAKEON:                  Set Brakes"
        echo "BRAKEOFF:                 Release Brakes"
        echo "START:                    Start Train"
        echo "HELP:                     Open the help document"
        echo "QUIT:                     Exit console"
        echo ""
        echo -n "menu:main> "
}

# MAIN

trap "exit" SIGHUP SIGINT SIGTERM SIGQUIT

print_header

while(true); do
        print_main_menu
        read ARG
        echo ""

        if [[ ! $ARG ]] ; then
                echo "Please select an number"
                continue
        fi
        case "$ARG" in
                STATUS)
                        echo "Brake:                            $BRAKE"
                        echo "BoilerOn:                         Yes"
                        echo "BoilerTemp:                       Normal"
                        echo "Coal Capacity Level:              97%"
                        echo "FluxCapacitor:                    Fluxing"
                        echo "Top Speed:                        88mph"
                        ;;
                BRAKEON)
                        sleep 1
                        BRAKE="on"
        echo "QUIT:                             Exit console"
                        echo "The brake has been applied."
                        echo $BRAKE
                        ;;
                BRAKEOFF)
                        sleep 1
                        BRAKE="off"
                        echo "*******CAUTION*******"
                        echo "The brake has been released!"
                        echo "*******CAUTION*******"
                        echo $BRAKE
                        ;;
                START)
                        echo  "Checking brakes...."
                        sleep 3
                        if [ $BRAKE == "on" ] ; then
                                echo "Brake must be off to start the train."
                        else
                                read -s -p "Enter Password: " password
                                [ "$password" == "$PASS" ] && QUEST_UID=$QUEST_UID ./ActivateTrain || echo "Access denied"
                        fi
                        continue
                        ;;
                HELP) $PAGER $DOC
                        ;;
                QUIT) echo "Exiting" ; exit
                        ;;
        esac
done
```

Running the `env` command will print out the environmental variables which includes your unique `QUEST_UID` value.

My Primary Account:
```yaml
sh-4.3$ env
```

```yaml
HOSTNAME=2a3d1803b361
TERM=xterm
QUEST_UID=4152024d099e6dfa4649f7016a860dxxxxxxxxx  # redacted
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
_=/usr/bin/env
PWD=/home/conductor
HOME=/home/conductor
SHLVL=12
```

My Test Account:
```yaml
sh-4.3$ env
```

```yaml
HOSTNAME=64fd641742ec
TERM=xterm
QUEST_UID=35ca097769d78a92364e3caff3cb75xxxxxxxxxx  # redacted
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
_=/usr/bin/env
PWD=/home/conductor
SHLVL=3
HOME=/home/conductor
```

In my primary account, I made a copy of the `Train_Console` script and called it `TC1`.  I modified the `PASS="24fb3e89ce2aa0ea422c3d511d40dd84"` value to be simply `123` to save from typing that long original password, set the `BREAK="Off"` and finally changed the `QUEST_UID=$QUEST_UID` to be the hardcoded `QUEST_UID=35ca097769d78a92364e3caff3cb75xxxxxxxxx` of my test account.

Executed the script `./TC1` then typed `START` and pressed enter.  My test account was sent back in time!  Fortunately there is enough entropy in the `QUEST_UID` value that I don't see people sending random users back to 1978.  If you could social engineer someone in to giving you their `QUEST_UID` value, you could send them back in time.  

**Although I'm sure this goes against the terms of service and I do not recommend anyone doing this!** 




## Wumpus Challenge 

![terminal](/assets/images/sans2016/terminal1_workshop_wumpus.png)

### Challenge Objective:  Kill the Wumpus 

```yaml
*******************************************************************************
*                                                                             *
* Find the passphrase from the wumpus.  Play fair or "cheat"; it's up to you. * 
*                                                                             *
*******************************************************************************
```

I started the game and successfully killed the Wumpus on the first try.

```yaml
./wumpus
Instructions? (y-n) n
You're in a cave with 20 rooms and 3 tunnels leading from each room.
There are 3 bats and 3 pits scattered throughout the cave, and your
quiver holds 5 custom super anti-evil Wumpus arrows.  Good luck.
You are in room 2 of the cave, and have 5 arrows left.
*sniff* (I can smell the evil Wumpus nearby!)
There are tunnels to rooms 5, 7, and 19.
Move or shoot? (m-s) s19

*thwock!* *groan* *crash*
A horrible roar fills the cave, and you realize, with a smile, that you
have slain the evil Wumpus and won the game!  You don't want to tarry for
long, however, because not only is the Wumpus famous, but the stench of
dead Wumpus is also quite well known, a stench plenty enough to slay the
mightiest adventurer at a single whiff!!
Passphrase:
WUMPUS IS MISUNDERSTOOD
```

Passphrase: `WUMPUS IS MISUNDERSTOOD`

This is great! I completed the challenge and obtained the passphase.  However, the challenge banner stated that it was possible to cheat and I wanted to know how to do so.  I ran strings on the `wumpus` file but when nothing jumped out at me, I turned to the Internet and found what seems the be the [original source code](https://github.com/vattam/BSDGames/blob/master/wump/wump.c) for the game "wump."  I wasn't sure how much the source code had been modified in our version but at least it gave me an idea of what the game's logic could be.  

I also noticed that lines 791-795 seem to be command line arguments but I wasn't sure what they did yet.

```yaml
{
 (void)fprintf(stderr,
 "usage: wump [-h] [-a arrows] [-b bats] [-p pits] [-r rooms] [-t tunnels]\n");
 exit(1);
}
```

Digging a little deeper I found an explanation for those options in an [archived wumpus man page](https://web.archive.org/web/20090214233010/http://linux.die.net/man/6/wump) thanks to ![web.archive.org](https://web.archive.org).

![wumpus_man](/assets/images/sans2016/wumpus_man.png)

Let's create a cave with only 6 rooms (-r 6) for the Wumpus to hide and 100 arrows (-a 100) instead of the default 20 rooms and 5 arrows. I like my odds...

```yaml
./wumpus -r 6 -a 100
Instructions? (y-n) n
You're in a cave with 6 rooms and 3 tunnels leading from each room.
There are 3 bats and 3 pits scattered throughout the cave, and your
quiver holds 100 custom super anti-evil Wumpus arrows.  Good luck.
You are in room 6 of the cave, and have 100 arrows left.
*rustle* *rustle* (must be bats nearby)
*whoosh* (I feel a draft from some pits).
*sniff* (I can smell the evil Wumpus nearby!)
There are tunnels to rooms 1, 4, and 5.
Move or shoot? (m-s) s1

*whoosh* (I feel a draft from some pits).
*sniff* (I can smell the evil Wumpus nearby!)
There are tunnels to rooms 1, 4, and 5.
Move or shoot? (m-s) s2

*thunk*  The arrow can't find a way from 6 to 2 and flys randomly
into room 4!
You are in room 6 of the cave, and have 98 arrows left.
*rustle* *rustle* (must be bats nearby)
*whoosh* (I feel a draft from some pits).
*sniff* (I can smell the evil Wumpus nearby!)
There are tunnels to rooms 1, 4, and 5.
Move or shoot? (m-s) s3

*thunk*  The arrow can't find a way from 6 to 3 and flys randomly
into room 5!
*thwock!* *groan* *crash*
A horrible roar fills the cave, and you realize, with a smile, that you
have slain the evil Wumpus and won the game!  You don't want to tarry for
long, however, because not only is the Wumpus famous, but the stench of
dead Wumpus is also quite well known, a stench plenty enough to slay the
mightiest adventurer at a single whiff!!
Passphrase:
WUMPUS IS MISUNDERSTOOD
Care to play another game? (y-n) 
```

Now that's what I'm talking about.

![challenge_coin](/assets/images/sans2016/Coin_WUMPUS.png)




## Doormat Challenge

![terminal](/assets/images/sans2016/terminal2_workshop_passphrase_cat_door.png)

### Challenge Objective:  Find the Hidden Passphrase

```yaml
*******************************************************************************
*                                                                             *
* To open the door, find the passphrase file deep in the directories.         * 
*                                                                             *
*******************************************************************************
```

Passwords (passphrases) are typically found in text files, so I'll see if I can find any .txt files first.

```yaml
$ find / -name *.txt
/home/elf/.doormat/. / /\/\\/Don't Look Here!/You are persistent, aren't you?/'/key_for_the_door.txt
```

Now that the location for the key is known, I just have to get to it.  It's not as straightforward as simply changing to that directory and viewing the file. Certain characters need to be escaped.  Escaping is a method of quoting single characters. The escape (`\`) preceding a character tells the shell to interpret that character literally.

Here is a one liner with each character being escaped.  Running this command will print out the passphrase:

`cat "/home/elf/.doormat/. / /\/\\\\/""Don't Look Here"'!'""/"You are persistent, aren't you?"/"'"/"key_for_the_door.txt"`

Which prints out:  `key: open_sesame`

Or you can achieve the same thing manually browsing directory by directory...

```sh
cd /home/elf/
ls -la  # to see all directories including hidden ones 
cd .doormat
ls -la  # to see all directories including hidden ones 
cd ". /"
ls -la  # to see all directories including hidden ones 
cd " "
ls -la  # to see all directories including hidden ones 
cd "\\"
ls -la  # to see all directories including hidden ones 
cd \\\\
ls -la  # to see all directories including hidden ones 
cd "Don't Look Here!"
ls -la  # to see all directories including hidden ones 
cd "You are persistent, aren't you?"
ls -la  # to see all directories including hidden ones 
cd "'"
ls -la  # to see all directories including hidden ones 
cat key_for_the_door.txt
key: open_sesame
```

![coin](/assets/images/sans2016/Coin_doormat_challenge.png)




## War Games Challenge 

![wargames](/assets/images/sans2016/terminal_santa_office.png)

### Challenge Objective:  Start a Global Thermonuclear War

To win this challenge, you need to enter the dialog from the 1983 movie [WarGames](https://en.wikipedia.org/wiki/WarGames). It has to be EXACT.  Now you can watch the [movie clips on YouTube](https://www.youtube.com/watch?v=KXzNo0vR_dU) and type it manually or use the [wargames.sh](https://github.com/abs0/wargames/blob/master/wargames.sh) script found on GitHub.  That script will almost get you past this challenge with one exception, line 216 says "Later. Lets play Global Thermonuclear War." It should be "Later. Let's play Global Thermonuclear War."

The text in ALL CAPTIALS is from the terminal.  The text in lowercase is what I entered.

```
GREETINGS PROFESSOR FALKEN.

Hello.

HOW ARE YOU FEELING TODAY?

I'm fine. How are you?

EXCELLENT.  IT'S BEEN A LONG TIME.  CAN YOU EXPLAIN
THE REMOVAL OF YOUR USER ACCOUNT NUMBER ON 6/23/73?

People sometimes make mistakes.

YES THEY DO.  SHALL WE PLAY A GAME?

Love to. How about Global Thermonuclear War?

WOULDN'T YOU PREFER A GOOD GAME OF CHESS?

Later. Let's play Global Thermonuclear War.

FINE

     WHICH SIDE DO YOU WANT?
  1.   UNITED STATES
  2.   SOVIET UNION
      PLEASE CHOOSE ONE:
2

PLEASE LIST PRIMARY TARGETS BY
CITY AND/OR COUNTY NAME:

Las Vegas

LAUNCH INITIATED, HERE'S THE KEY FOR YOUR TROUBLE: 
LOOK AT THE PRETTY LIGHTS
Press Enter To Continue

`LOOK AT THE PRETTY LIGHTS` is the passphase to enter The Corridor hidden behind Santa's bookshelf.
```

Passphase is `LOOK AT THE PRETTY LIGHTS`


![coin](/assets/images/sans2016/Coin_wargames.png)
