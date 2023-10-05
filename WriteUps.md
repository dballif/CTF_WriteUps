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

Online, I found (this repository)[https://github.com/Parkayun/base65536] that somebody creatd to encode & decode base65536 using python. After installing this with pip, I wrote this small python script to decode our file:

```
import base65536

file = open("baseffff1","r")

toBeDecoded = file.read()

print(base65536.decode(toBeDecoded))
```

This gave me this message (flag redacted of course):
`b'Nice work! We might have played with too many bases here... 0xFFFF is 65535, 65535+1 is 65536! Well anyway, here is your flag:\r\n\r\nflag{INSERT_FLAG_HERE}'`

### Traffic
Looks like we're analyzing network traffic. I was thinking I could just use wireshark which I'm comfortable with, but it's not the right file type. The prompt suggests using rita or zeek.

I chose to try zeek because it starts with a z which is kind of fun.

