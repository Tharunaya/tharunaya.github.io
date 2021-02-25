---
layout: post
title: Tenable CTF Writeup
tags: [ctf, writeup, stego, misc]
---

Tenable's first CTF took place past weekend. I really enjoyed the challenges! In the following, you can find some writeups for the Stego and Misc category.

# Stego

## Easy Stego - 25 points
##### Message
Can you find the flag?
##### Content
![blm](/assets/img/01_TenableCTF/blm1.png)
##### Solution
The image consisted of three seperate images, each in a different colour. I decided to use the tool stegsolve for getting a better look at it. A download can be found here: https://github.com/eugenekolo/sec-tools/tree/master/stego/stegsolve/stegsolve. By executing the command ``` java -jar stegsolve.jar  ```, stegsolve is started. Now, the image can be opened and viewed in different filters. 
![blm](/assets/img/01_TenableCTF/blm_combined.png)
On the images, we can now see the flag:
``` flag{Bl4ck_liv3S_MATTER} ```

## Secret Images - 125 points
##### Message
We discovered Arasaka operatives sending wierd pictures to each other. It looks like they may contain secret data. Can you find it?
##### Content
![crypted1.png](/assets/img/01_TenableCTF/crypted1.png) ![crypted2.png](/assets/img/01_TenableCTF/crypted2.png)
##### Solution
The two files look like they have both been encrypted with the same XOR key. Here's a nice explanation for what XOR is and what properties it has: https://accu.org/journals/overload/20/109/lewin_1915/. 
While XOR encrypting one image with a key is quite secure, repeating the same key twice makes it possible to find out the contents of the original, unencrypted images. By assuming that both files have been encrypted with the same key, we can figure the following equations.
```sh
img1 ^ key = enc_img1
img2 ^ key = enc_img2
```
As we have crypted1.png and crypted2.png given, we can make a new equation and solve it for the original images:
```sh
enc_img1 ^ enc_img2 = img1 ^ key ^ img2 ^ key
```
As XORing something with itself always returns zero, the two occurences of the key cancel each other out.
```sh
enc_img1 ^ enc_img2 = img1 ^ img2
```
This means that by XORing the two images given with each other, we'll end up with an image of the unencrypted images XORed with each other. In practice, this can be done with the following command: (1)
```convert crypted1.png crypted2.png -fx "(((255*u)&(255*(1-v)))|((255*(1-u))&(255*v)))/255" decrypted.png```
Running the command confirmed the hypothesis that both images had been encrypted with the same key and I ended up with the following image:
![decrypted](/assets/img/01_TenableCTF/decrypted.png)
```
flag{otp_reuse_fail}
```
(1) Note that you have to install imagemagick first, e.g. by running ```sudo apt install imagemagick-6.q16``` (or any other version)
##### Resources
General information on XOR: https://accu.org/journals/overload/20/109/lewin_1915/
Command for XORing two images: https://stackoverflow.com/questions/8504882/searching-for-a-way-to-do-bitwise-xor-on-images

# Misc

## Esoteric - 25 points
##### Content
```
--[----->+<]>.++++++.-----------.++++++.[----->+<]>.----.---.+++[->+++<]>+.-------.++++++++++.++++++++++.++[->+++<]>.+++.[--->+<]>----.+++[->+++<]>++.++++++++.+++++.--------.-[--->+<]>--.+[->+++<]>+.++++++++.>--[-->+++<]>.
```
##### Solution
By simply putting the given string into a Brain Fuck converter (such as https://www.dcode.fr/brainfuck-language), one could decipher the flag.
```
flag{wtf_is_brainfuck}
```

## One Byte at a Time - 50 points
##### Message
challenges.ctfd.io:30468
##### Solution
Via ``` nc challenges.ctfd.io 30468 ```, one could enter the part of the flag one already knew and if it was correct, the site returned the following:
![next letter XORed with random octet of unknown IPv4 adress](/assets/img/01_TenableCTF/onebyteatatime.png)
My first approach was trying to brute force the site, but as it reacted extremely slowly I tried another approach. By knowing that the flags start with "flag{", I figured I could obtain at least parts of the unknown IPv4 XORed with the following character of the flag. As XORing is associative and commutative, I could just XOR the suggested hex value for the parts of the flag I already knew. This way, I figured out three of the four octets in the IPv4 address: 119, 16 and 2.
Now, I XORed all three values with each hint the site gave me, chose the most plausible outcome, checked whether it was right and then repeated that process for the next letter. I ended up with the following flag:
```
flag{f0ll0w_th3_whit3_r@bb1t}
```

## Broken QR - 100 points
##### Message
Can you scan this QR code for me?
##### File
![task image](/assets/img/01_TenableCTF/broken_qr.png)
##### Solution
In this task, above QR code had to be restored into a scannable format. My main resource for this task was the article "Wounded QR codes" by DataGenetics (https://www.datagenetics.com/blog/november12013/index.html).
![image from the article](https://www.datagenetics.com/blog/november12013/anat.png)
I started by restoring the positioning markers in the top left and top right corner by simply copying the one from the bottom left corner into their positions. Then, I fixed the left timing mark by copying the other one, rotating it by 90 degrees and putting it in the correct position. My last step was to copy a single black square and insert it at any position where remains indicated former black markers. However, that step was probably optional as it restored no markers essential for scanning a QR code. I used the open source image editor Krita.
![progress image](/assets/img/01_TenableCTF/qr_reconstruction.png)
Upon scanning the resulting QR code, I found the flag:
```sh
flag{d4mn_it_w0nt_sc4n}
```

## Forwards from Grandma - 100 points
##### Message
My grandma sent me this email, but it looks like there's a hidden message in it. Can you help me figure it out?
##### Content
[Mail](/assets/img/01_TenableCTF/tmp.eml)
##### Solution
By having a closer look at the subject of the mail, one could see a large amount of FWD: and RE:'s - which is not too unusual in a mail from grandma, but the swirly brackets in between caught my attention as it resembled the flag format. 
My initial guess was it being binary with the FWD: as a 0 or 1 and the RE: as a 1 or 0, respectively, but as that lead to nothing I figured it was morse code instead. Knowing that the morse code before the opening of the swirly brackets had to be "flag", I could confirm my hypothesis and find out the correct mapping.
```sh
"FWD:"  => "."
"RE:"   => "-"
```
### Quick Note: If you paid enough attention to the header, you probably noticed that there were in fact double spaces in the header indicating the end of a morse code letter, meaning you could just decode it. If you are interested in reading how to decode morse without spaces, you can read on. Otherwise, just skip ahead to the flag.

The resulting morse code was: “..-..-...---.{....--.---..........--.-.----.-..}”
The difficulty here was that no spaces were in between the morse code letters, meaning that there were a ton of possible solutions (like 1628277960) for the text between the curly brackets due to the ambiguity (e.g. "..." can mean eee, ie, ei or s).

After some while and running a function that would return me every possible combination of spaces inserted for more than six hours (Spoiler Alert: it didn't finish), I noticed that words in the flags were usually seperated by underscores. The morse code for underscores is quite recognizable (..--.-) and occured two times in the encoded flag. With that information, I could now split my flag into three seperate words of much shorter length and thus less ambiguities.
```
“flag{.._--........_.----.-..}”
```
With the initial function for inserting spaces in combination with a function for decoding the morse code, I could now determine all possible solutions for my three words. As I thought the words would either be in a dictionary or be leetspeak and thus containing numbers, I implemented optional checks for both possibilities. (1)
By picking the most plausible words from the resulting list, I ended up with the right flag.
```
flag{i_miss_aol}
```
(1) That would have probably worked great, but when transcribing the FWD:'s and RE:'s to morse I had noted down one dot too much which lead to no sensical dictionary entries. 
##### Resources
Function for inserting spaces: https://www.geeksforgeeks.org/print-possible-strings-can-made-placing-spaces/
Function for decrypting the morse code: https://www.geeksforgeeks.org/morse-code-translator-python/

