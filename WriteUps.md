# Huntress CTF 2023

## Intro
These are my write ups for the Huntress CTF 2023 challenges. I've already finished a few of the beginner challenges, so I'll go ahead and sum them up, but they may not be as detailed as the rest. The CTF runs the entire month of October, but the challenges are released in batches once a day.

When the challenge is over, I'll update the results section with my standing

## Results

## Challenges

### Read the rules
I accidentally found this one before realizing there was a challenge related to the rules. When I first created my account, I went to the rule page to make sure I didn't miss anything. In the rules it states that there is a flag hidden somewhere on the page. 

After reading through the rules, I hadn't found it, so I viewed the page source and found it hidden as a comment there.

### Technical Support
This one was pretty easy. It's just an introductory flag to teach the contestants how to use the technical support system. 

To get it, just join the discord, verify you are human by reacting to the rules with the specified emoji. This will allow you to see the other pieces of the discord, if you enter the technical support channel, you will see the flag.

### String Cheese
Becuase it's a picture, I figued it was going to be some kind of steganography. So I broke out steghide.

I ran `steghide --extrac -sf cheese.jpg` which prompted me for a password. I obviously didn't have one. Thinking there might be some sort of hint in the prompt, I returned to the website and read it again. The author mentions that their mom used a different name for cheese sticks. This seemed like it might be a hint so I started trying other names like: string, string cheese, etc. I didn't have any luck with this, so I looked online for other names but couldn't find anything I hadn't already tried.

My next idea was to look for passwords or flags in the metadata. Using `exiftool cheese.jpg` I was able to view a lot of info, but there was nothing that stood out.

Finally, I used `xxd cheese.jpg` to view the hex dump for the file. Hidden in the seeming jumble of characters, I foudn the exif data, and right above it was the flag.

### Notepad
This one was very straight forward. I downloaded the file and opened it in notepad. The flag was there in plaintext.

### Book by its Cover
The file downloaded for this challenge was a .rar file. My first instinct was attempt to extract the contents. When I tried that, I got an error basically saying that the file was not supported. This, along with the title of the challenge, made me think that it was just mislabeled as a .rar.

I ran `file book.rar` to find out what it really was. Turns out it was actually a .png. So I changed the file extension and opened it in an image viewer. The image was of the flag.

### Query Code
My first move was to `cat query_code` but this jsut output a bunch of gobbledygook.

So then I used `file query_code` to see if I could learn anything else about it. It seems to think that this is a .png.  As in the 'Book by its Cover' challenge, I tried changing the file to a .png and opening it with an image viewer. I found a QR Code! 

Using my phone, I scanned the QR code to get the flag. (Note: this is a security company and I figured I could trust their QR code, but it's probably not a good idea to scan unknown QR codes in the wild.)

### BaseFFFF+1
The prompt seems to allude to decoding the file using another base. Probably BaseFFFF+1. 

FFFF in hexadecimal = 65535 in decimal. 65535 + 1 = 65536. So I think we are boing to decode using 65536. Now I just needed to find out how to decode with that base.

Online, I found [this repository](https://github.com/Parkayun/base65536) that somebody creatd to encode & decode base65536 using python. After installing this with pip, I wrote this small python script to decode our file:

```
import base65536

file = open("baseffff1","r")

toBeDecoded = file.read()

print(base65536.decode(toBeDecoded))
```

This gave me this message (flag redacted of course):
`b'Nice work! We might have played with too many bases here... 0xFFFF is 65535, 65535+1 is 65536! Well anyway, here is your flag:\r\n\r\nflag{INSERT_FLAG_HERE}'`

### Traffic
Looks like we're analyzing network traffic. I was thinking I could just use wireshark which I'm comfortable with, but it's not the right file type. The prompt suggests using Rita or Zeek.

I chose to try Rita.

Rita stands for Real Intelligence Threat Analytics and is a framework to analyze network traffic to look for malicious traffic.

The hardest part of this challenge was setting up Rita in WSL. I'll go over those steps since that's what took the longest.

1. Download the [repo](https://github.com/activecm/rita.git) from GitHub
2. If  you are on supported OS use the install.sh script to install `sudo ./install.sh` Theoretically you could use --disable-zeek to save some time.
3. WSL is kind of supported for debian. You will see an error about systemd not being usable or something. This is becuase WSL uses some other mechanism. We'll get past this issue in a minute.
4. Modify /etc/rita/config.yaml to include the correct address for your local subnet

If you try to use rita import \<log directory\> at this point, you will get and error saying the databases can't be reached. This is because MongoDB could not be setup to automatically run.

5. Run `sudo mongod` to start the MongoDB server. (Put in background or open another terminal)
6. Now run `rita import \<log directory\> \<name of new DB\>`

It should now work, the logs will be analyzed (Note rita can handle the .gz files so no need to extract them)

7. `rita html-report \<name of DB\>`

You can now view the html report by opening the generated index.html file in your browser.

The trick to this CTF is to look through each line searching for  "sketchy site". In the "Beacons SNI" tab of the report you can see the website names and looking through the list we see "sketchysite.github.io".

Going to this site produces the flag

### CeaserMirror
The prompt here gives the hint that it is a clever ROT13. So I took the contents of the file and dropped them into CyberChef, chose ROT13, and baked it.

The output was half readable, but the other half was backwards. Rather than try to flip the text to make it easier to read, I just started to read it backwards.

You have to start with the firts forward-facing line, and when you get to the backwards piece, start reading from the very end. It's kind of a fun exercise, if a little slow going. The flag has been split into three parts and the parts in the backwards text will have to be reversed in the actual flag.

### I Won't Let You Down
Okay, so this one took me much longer than it should have. I went to the website and watched the video. The website basically tells you to use nmap. So I did that and found 4 ports open:

```
PORT     STATE    SERVICE
22/tcp   open     ssh
80/tcp   open     http
646/tcp  filtered ldp
8888/tcp open     sun-answerbook
```

The first 3 look normal to me, and I know the CTF said don't use ssh. So my immediate thought was to try going to port 8888. It didn't load in the browser, or at least it just spun, so I decided to try to just use curl. 

Curl gave me an error saying http 0.9 isn't allowed. When I looked this error up, I found people saying to check if wget gave you more. 

When I ran wget on that port, it started fetching an html file. I let this download for a while before I decided it was probably just spinning. But when I checked the html file I had pulled down, it contained the lyrics to the song and at the very end the flag.

### Zerion
This is the first "Malware Analysis" challenge of the competition and I'm pretty excited. I haven't done much malware analysis to this point, but it's always been something I'm super interested in.

So I downloaded the file onto my VM and then cut the network to it to make sure I was being at least a little safe. There was a warning in the challenge that this is based on a real malware sample that they have defanged, but they still urge us to use caution. I think that is so cool.

So my first step was to run `file` on the file to see what it actually is. It's a php script. 

I then used `cat zerion` to find out what is in the script to see if I can find out what they are doing. This returned a massive jumble of characters that started with this a intelligible text:

```
<?php $L66Rgr=explode(base64_decode("Pz4="),file_get_contents(__FILE__)); $L6CRgr=array(base64_decode("L3gvaQ=="),base64_decode("eA=="),base64_decode(strrev(str_rot13($L66Rgr[1]))));$L7CRgr = "d6d666e70e43a3aeaec1be01341d9f9d";preg_replace($L6CRgr[0],serialize(eval($L6CRgr[2])),$L6CRgr[1]);exit();?>
```

This was followed by the seemingly gobbledygook. I have left it out since it's partly real malware. But we'll come back to it soon.

I recognized a few things right off the bat: the php heading, a couple of variables, some encoding, etc.

I broke it into it's variables to decipher what is going on.

$L66Rgr is pretty easy to decipher. It's creating an array from a string using the explode function, the delimeter is just encoded in base64 but it turns out it is "?>". The string to be broken is loaded using file_get_contents and is using this php file. So it's grabbing that log string of gobbledygook.

$L6CRgr is an array. The first two entries are just base64 encoded as well so quickly decoded to be /x/i, and x. The last entry was a little more fun. It's a combination of a few different obfuscation techniques.

`base64_decode(strrev(str_rot13($L66Rgr[1])))`

So it's referring back to the gobbledygook. I dropped it into cyberchef and then worked backwards on obfuscation methods. Rot13 -> reverse order -> base64 decode. Now I see it is actually a rather large chunk of code.

$L7CRgr is just a string. Nothing special there yet. I'm sure we'll get to using it.

Then the script calls `preg_replace($L6CRgr[0],serialize(eval($L6CRgr[2])),$L6CRgr[1]);` before finishing.

This referes back to our previous work. preg_replace is a php function that uses a pattern to serach and replace a string. It is case-insensitive. And is called like so:

`preg_replace(<Pattern>,<Replacement String>,<String to act upon>)`


So the pattern we are searching for is the first entry of $L6CRgr which is /x/i  
We are replacing it with the serialized version of the gobbledygook (evaluated as php code), and the string to be replaced is x. Interesting. I'm not exactly sure what this will do, because from what I can tell it's replacaing things in string 'x' which doesn't exist. At least not that I can see at this point. Maybe I'll come back to it.

Coming back to $LCRgr, I see it in the php function that we have already decoded, so I drop it's value in.

Then I just have to decode the php function I have found.

For my sanity, I like to start out by making the code more readable by stylizing it. I just go through and add returns and indents in ways that make sense for reading softare. For example, after a semi-colon, there will be a return, curly brackets should be lined up with the corresponding bracket, etc. 

As I went through the code doing this, I stumbled across the flag!