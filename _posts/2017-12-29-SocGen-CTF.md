---
layout: post
title: Societe Generale CTF Write Up
---

Societe Generale Global Solution Centre (SG GSC) hosted the fourth edition of their hackathon, Brainwaves 2017-18 on [HackerEarth](https://www.hackerearth.com/challenge/competitive/brainwaves-17-3/). The challenge consisted of 11 questions which were to be answered in a theoretical manner. The challenge started on 4th December and ended on 24th December 2017. All the contestants had to perform attacks on a shared instance which lead to downtime various times and also some contestants started messing around with the flags of other contestants (which is not cool at all).

So I started the challenge on the evening of 4th december knowing that I had exam the next morning, I fired up my system to see what the challenge had to offer me. On first sight I knew that the application I'm dealing with is behind Cloudflare (thank you to their annoying captcha) and I knew that my job was going to be tough. So, I thought let’s try enumerating DNS by adding 0x10.info in Cloudflare and let it do my work 😜

![cloudflare socgen](https://blog.amarun.tech/images/socgen/dns.JPG)

I shot an arrow in dark and it hit the bullseye. So I had 4 different IP addresses one of which could the the CTF server.
```
74.125.236.67
38.109.218.93
51.15.45.71
54.202.128.175
```

 ![curl ip](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/server.JPG)
 
Alright, so 74.125.236.67 and 54.202.128.175 are dead. I visited 51.15.45.71 found Load Balancer/HA Proxy but it’s not related to CTF and 38.109.218.93 is hosting the CTF.

![meme time](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/giphy.gif)

Since Cloudflare is bypassed I can enumerate services running on the server. This was a tedious process because I had to scan all 65535 ports. So, I ran nmap on the IP Address to find all open ports and enumerate services running on them.
```
PORT      STATE    SERVICE
53/tcp    open     domain
80/tcp    open     http
81/tcp    open     hosts2-ns
82/tcp    open     xfer
135/tcp   filtered msrpc
137/tcp   filtered netbios-ns
139/tcp   filtered netbios-ssn
445/tcp   filtered microsoft-ds
497/tcp   filtered retrospect
593/tcp   filtered http-rpc-epmap
1024/tcp  open     kdm
2048/tcp  filtered dls-monitor
2220/tcp  open     ssh
2376/tcp  open     docker
2745/tcp  filtered urbisnet
3005/tcp  filtered deslogin
3380/tcp  open     sns-channels
3389/tcp  open     ms-wbt-server
7547/tcp  filtered cwmp
8000/tcp  open     http-alt
8080/tcp  open     http-proxy
8086/tcp  open     d-s-n
9090/tcp  open     zeus-admin
33389/tcp open     unknown
```
Finding webserver version seemed easy but it certainly was not. I thought I could simply generate an error page and that should give me webserver info but this wasn't the case. Because the error page said “This is not* a web server, look for ssh banner Server”. So, I need to grab ssh banner? Lets keep directory busting maybe I can find something like admin panel, config, backup folder. And I found phpinfo.php under /includes/ hmm interesting? Let's reveal it's secrets

 ![phpinfo](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/phpinfo.JPG)
 
 So, we have 14.04.1-Ubuntu with Apache2 and PHP Version 5.5.9 installed. So that solves our third question but what’s bugging me is “This is not* a web server, look for ssh banner Server”. Let’s head over to putty and find what secrets does the ssh header holds. But the ssh port is not the default one, let me take a look at nmap scan result to see if it found the “fuzzy” ssh port. And sure it did, the ssh was running on port 2220. Let's connect to it.
 
 ![putty](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/ssh.JPG)
 
 So, a bunch of random numbers huh? But at the end of it we have a few characters as well at this point I knew that it has to be Hex Encoding (almost all digits 0-9 and not alphabets greater than F?), let’s decode it. The funny part is the encoded string was encoded 4 times. So a few iterations I got the deciphered text.

![ssh banner](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/dicipher.JPG)

“ssh banner forward slash could lead you to a sh3ll.php”, okay so what I understand from this is that I need to put a forward slash sh3ll.php would lead me to sh3ll.php (I got excited because I thought it would be something like WSO or C99). Let’s see what’s present at https://socgen-ctf.0x10.info/sh3ll.php

![sh3ll](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/sh3ll.JPG)

Hurray I found a shell, let’s follow the instructions given and capture the flag. (Sorry for big image)

![sh3ll flag](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/screencapture-socgen-ctf-0x10-info-sh3ll-php-1512838868787.png)

Enough of all of this, let’s get into the main application and get answers for more questions. So, I head over to https://socgen-ctf.0x10.info/members.php?p=login but the application wants me to login. Alright so blindly I typed in ‘or’’=’ as username and password because that's what a logical man would do right? And guess what? 

![error](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/sqli.JPG)

The application is surely vulnerable but not stupidly vulnerable. Let’s see if there is something hidden in source maybe. And I found the following comment.

![source](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/comment.JPG)

Register field is commented out, how evil 😡. Let’s follow the link ourselves. https://socgen-ctf.0x10.info/index.php?p=. following this link brought me to a what seems like a broken page, but maybe something is hidden in source? But no just an empty comment to tease me.

![source tease](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/empty%20comment.JPG)

And then I saw the code for menu bar

![menu bar](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/menuodd.JPG)

Hyperlinks for FX Home, User login and Contact Us seem similar but my eyes picked the odd one out "FAQ". Maybe something at FAQ? Let’s follow the link the find out https://socgen-ctf.0x10.info/index.php?url=/faq&p=faq

![faq](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/faq.JPG)

Another puzzle huh? Do I need to decipher this? Doesn’t seem like a verse from Shakespeare Sonnet neither does it look like an excerpt from a book (I mean who would write a bunch of random shit in their novel?) and it is signed by faq.php, so do I look for faq.php in the same directory? Oh yes that’s what I did. https://socgen-ctf.0x10.info/faq.php but it redirects me to wireshark website. Do I need to use wireshark then? But what packets am i going to analyze? Let’s keep this url in our notes and we’ll get back to it later. {Note 1}
I kept exploring this menu in hopes of finding something and all I found was a Contact Page without CSRF protection (I mean that’s one of the questions in CTF and CSRF is in top 10 vulnerabilities in OWASP). https://socgen-ctf.0x10.info/index.php?p=contact-us 
I really didn't invest much time in this top 10 owasp question, maybe it was as interesting.

I wasn’t even a step closer to logging into the member portal but then I looked back to my directory buster results and see that it found /siteadmin/ at first I thought maybe this will the admin portal and /members.php is a portal for regular users. So, I opened https://socgen-ctf.0x10.info/siteadmin/ Voila! No login required seems like I’m in admin panel without any authentication. Another bug? I think so. Well it seemed like another broken page so let’s hope into source code. Nothing interesting but the menu again.

![admin menu](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/siteadminmenu.JPG)

Did you notice what I noticed? Take a closer look

![redir](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/redir.JPG)

redir.php takes url as a parameter and redirects the user to root directory. Clearly this is the open redirect. Another flag captured.

![meme time](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/33a7053d6bc5390c1c42af7232d3748b0aaa88789eda2112a5172121988ed552.jpg)

Let’s change redir.php?url=/ to redir.php?url=https://www.hackerearth.com and check if it redirects to HackerEarth. Yes it sure does redirect let's report it.
https://socgen-ctf.0x10.info/redir.php?url=https://www.hackerearth.com 

Moving on with exploring menu we have Currency Rate List which displays the data added through Add Currency Rate. Let’s go ahead add some data.

![meme time](https://i.pinimg.com/736x/fc/7a/91/fc7a91b58653cd38af9d46ad17c25784--nickelodeon-spongebob-spongebob-memes.jpg)

![add event](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/addevent.JPG)

AND it gets added

![added event](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/addeventconfirm.JPG)

Let’s go back to Currency Rate List and check if alert is working or if they have xss filter.

![pop up](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/event1.JPG)
![pop again](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/event2.JPG)

Boo Yeah! It works a simple Stored XSS.
Moving on to Add Customer I find another form which let’s me upload some text (maybe let me execute some script?).

![add customer](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/addcustomer.JPG)

And yes, yes, yes it was updated with a sweet alert. Let go ahead List Customer and see our pop up.

![customer pop](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/listcustomer.JPG)

![meme time](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/174910_7c6707f7f47832fa8cd3b3d591fce60c.jpg)

Next in line we have Limit and it doesn’t interests me (although there's a question about limit but it was validation bug) so let’s skip to Content.

![content](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/content.JPG)

List of articles? Ads? I don’t know, let’s follow the link and find out https://socgen-ctf.0x10.info/siteadmin/index.php?p=edit&id=admin2 opens up a page with a box with probably source of the content that I clicked on? I tried uploading some php code but of course the update function wasn’t functional. Let’s just embrace the beauty of this url for a few seconds and try figuring out what exactly is happening. So, there are two parameters p which means page I think and id which I think is the parameter for file to edit and not for index.php. So, id=admin2 and from my directory busting record I had found /content/config/admin2.php could these two be same? There’s only one way to find out let’s compare content of both. The above link outputs admin2 and so does https://socgen-ctf.0x10.info/content/config/admin2.php could the id be reading data from config directory? Let’s try setting id=contact.php and damn it does not work but wait let’s try removing the extension? So, I set id=contact and yes yes yes it works. So, what I found is an amazing tool to read php files in the web application. Let’s try reading index.php, so the current working directory is /content/config/ that means we need to go back two folder so I set id=../../index and it works I can view the source of index.php

![meme time](https://media.giphy.com/media/uQ8ba0fLX3cCA/giphy.gif)

![index source](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/dir%20traversal.JPG)

Sweet! Remember that faq.php from [Note 1]? Let’s try reading that file not for any reason but because I can 😜.
https://socgen-ctf.0x10.info/siteadmin/index.php?p=edit&id=../../faq

![faq source](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/faqsource.JPG)

I am certainly not a php expert or anything but what I can interpret here is that if the parameter for faq.php (which is news) is left empty then redirect to wireshark website and if it’s magic then show this string? Let’s try https://socgen-ctf.0x10.info/faq.php?news=magic but it redirects me to https://socgen-ctf.0x10.info/magic seems like I found another open redirect 😜. Let’s redirect it to HackerEarth https://socgen-ctf.0x10.info/faq.php?news=https://hackerearth.com but the mystery remains unsolved how do I make this string appear in web page? But then this thought struck my mind that why do I need to make in display when it’s already in front of me why not go ahead and decode it. On first look I knew it was base64 encoding because of infamous two trailing "=" symbols. I decoded the string and I got this as output
```
explored validate.php?
```
Yes! you are absolutely right I didn’t, so let’s explore validate.php. https://socgen-ctf.0x10.info/validate.php

![rce](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/rce%20validate.JPG)

I found another RCE. Hurray another flag captured.

I had explored almost everything but I wasn’t even a step closer to find login or SQLi in the web application. Then this idea struck my mind, since I can view source code of every php file why don’t I go through all functions and maybe fetch the database information. I already knew about /includes/ from earlier findings and since directory listing was enabled I looked inside to find something of use. I decided to look into /includes/classes/config.inc.php because the name is so appealing, as if it calls me to explore it and I do it.

https://socgen-ctf.0x10.info/siteadmin/index.php?p=edit&id=../../includes/classes/config.inc

![db info](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/dbinfo.JPG)

I got too excited and uploaded adminer on one of my domains to connect to this database remotely, my excitement then settled down quickly because developer made web application vulnerable not stupid. I decided to look into theme files to explore the pages which are not crawled. Let’s start with index.php and clearly I know where I should look start looking at

![index source](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/index.JPG)

https://socgen-ctf.0x10.info/siteadmin/index.php?p=edit&id=../../content/templates/default/center_home so being lazy I quickly press ctrl+f to directly look for $p == instead of going through whole code and I found a couple of interesting pages which were not crawled earlier with tools (see brain crawls better than tools) since I was tired and wanted to get relaxed I decided to go to https://socgen-ctf.0x10.info/index.php?p=tv_channels because what’s more relaxing than Tom and Jerry 😍. I click on Cartoon Network and I see a pop up, maybe Tom and Jerry is going to start soon but it never did. How disappointing! Let's see the code and figure out why Tom and Jerry is not playing, I opened up iframe url in a new tab and when I hover on Cartoon Network I was an interesting link. Any guess what it was? It was another url with parameters where I can append my apostrophe, Hurray!! 

https://socgen-ctf.0x10.info/ajax.php?cid=&p=view_channel&id=12&fid=tv

let's put apostrophe to &fid=tv, wow no error nothing happened, tom and jerry is still not playing. Let's remove fid from url and try injecting id because id and sqli are related from the beginning of the universe. I put an apostrophe and boo yeah, I got it sweet mysql error.

![sqli](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/tomn.JPG)

Let's fire up Havij and do sqli because who would do it manually when you can get it done automatically. https://socgen-ctf.0x10.info/ajax.php?cid=&p=view_channel&id=12’ (Affected url). Finally, I got the table I got the data now let’s login to members.php at last.

![fetched](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/sqlidata.JPG)

I log into /members.php using admin:admin123 and it redirected me to /siteadmin/ and I was like dude what are you doing I could explore this directory even before you redirected me. I want to see what's inside members.php, so I relogin and quickly click on Buy/Sell and there is a question related to it as well (good job). So, I had to exploit Order Limit and make it more than max, okay let's fire up developer console and see how they are validating the quantity I see a small and sweet 'value max="214"', at first I thought it won't be this easy but why not give it a try. I set max="999999" and then I put quantity as "999999" and insert it. Voila the row was inserted. Was making it more than max the only thing I was going to do? NOPE since I'm in developer console let’s try making an order for negative number, great there was no validation for negative numbers but in Price the min value was set to 1 so I did the most logical thing I remove min from input field parameters and yes yes yes it worked. Row was inserted with negative quantity and price. Should I stop here? My heart says try putting characters in quantity and I followed my heart I changed type="numbers" to type="text" for input field of quantity and placed a sweet little alert and inserted the row. Wow, it might be my lucky day the row was inserted. Now I even got stored XSS on the page.

While going through source code of /content/templates/default/member/center.php I found this interesting piece of code

![theme source](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/search.JPG)

Making dumb guess the code wants be to do this https://socgen-ctf.0x10.info/members.php?p=search&s=% but I can directly decode this to reveal the secrets because it is again base64. And what I think is some login credentials I am not where to user them but I have decoded the passwords for you.

![decoded](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/loginbase.JPG)

Master hash wasn't difficult to find, going through the source of members.php I saw a couple of setcookie statements they interest me because of the name of cookie so I grabbed "setcookie("master_hash", "b57f08daa5ddc7b443d15e0fb9b80d26", time()+1800);" extracted the encoded string and put it into hashkiller's md5 decoder because it seemed like md5 hash to me. Decode hash is  
```
admin?
```
![master hash](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/master%20hash.JPG)

While directory busting I found a folder name /apps/ with an exe, readme file. I downloaded adobelogin.exe and read the readme file and it said "* AdobeLogin.exe should display SECRET FORM, try to decipher network communication, and decompiling would also help! - AdobeLogin communicates to Web App for authentication, find the loophole in webapp as well." hmmm okay so reverse engineering? fine I fired up HexEditor to see what's going on and I found the web application it was calling to verify login, it was "http://pentest.0x10.info/info.php" and since I can view code of any php file I used "http://38.109.218.93/siteadmin/index.php?id=../../info&p=edit" looking at source I found that username and password for the application was adobe:photoshop. Flag captured?

![adobe](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/adobe.JPG)

While enumerating files on server I found /visit.html which had an image "dreaming_owl.jpg", image looks fine but there is a period with forward slash. I downloaded owl image and opened with my hex editor and at the end of the file I see "freenode ##security" seems like a portal to communicate? Maybe another flag? I don't know.

Now the last one and probably my favorite since I enumerated DNS I never even once checked what cname were present. So, I decided to give it a look. I saw that storage cname was pointing towards "storage.0x10.info.s3.amazonaws.com", I opened the bucket because I thought maybe it's another clue but wait there was no bucket. Did that mean I can claim the bucket and raise the flag? That's exactly what I did. http://storage.0x10.info/flag.txt

![flag](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/flag.JPG)

Not just AWS, there was also subdomain app.0x10.info which was pointing app.0x10.info.herokudns.com this domain could also be hijacked but since I am not having credit card, I could not verify my account and raise the flag on this subdomain. But it is possible to hijack this sub domain since cname is pointing towards an unclaimed Heroku app. I don't know if they would consider this as a finding or flag.
Alright folks that's how I did it.

![meme time](https://raw.githubusercontent.com/its-arun/its-arun.github.io/master/images/socgen/didit.png)

My methods, terminology and the way to writing this blog post might not be so fancy and technical. But it worked and that's what matters at the end of the day.

Kind Regards
Arun Chaudhary (felli0t)
[HackerEarth Profile](https://www.hackerearth.com/@felli0t)
