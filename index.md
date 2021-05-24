

![](https://github.com/Rshrimali17/Delivery_HTB/blob/main/banner.png)

 

<img src="assets/images/htb.png" style="margin-left: 20px; zoom: 60%;" align=left />    	<font size="10">Delivery</font>

​		20<sup>th</sup> May 2021

​		Machine Author(s): ippsec

​		

 



### Description:

Delivery is an easy-level machine active on HackTheBox. 

### Difficulty:

`easy`

### Flags:

User: `Find it yourself and see how it feels`

Root: `Find it yourself and see how it feels`

# Enumeration


1) So first of all I perfomed an nmap scan using `nmap -v -sC -sV -oN intial 10.10.10.222` . The following result popped out 

` Nmap 7.80 scan initiated Thu May 20 01:10:38 2021 as: nmap -v -sC -sV -oN initial_scan 10.10.10.222`
`Nmap scan report for 10.10.10.222`
`Host is up (0.065s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 9c:40:fa:85:9b:01:ac:ac:0e:bc:0c:19:51:8a:ee:27 (RSA)
|   256 5a:0c:c0:3b:9b:76:55:2e:6e:c4:f4:b9:5d:76:17:09 (ECDSA)
|_  256 b7:9d:f7:48:9d:a2:f2:76:30:fd:42:d3:35:3a:80:8c (ED25519)
80/tcp open  http    nginx 1.14.2
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.14.2
|_http-title: Welcome
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
 Nmap done at Thu May 20 01:11:03 2021 -- 1 IP address (1 host up) scanned in 24.72 seconds `

I also did a an all ports scan and found another port -> 8065.

2) As we can see from the result it has OpenSSH 7.9p1 as well as nginx/1.14.2 running. I went on searchspolit to check exploits on nginx but there wasnt any there.

3) Lets go check out what is on 10.10.10.222. It redirects to a delivery website. 

![](https://github.com/Rshrimali17/Delivery_HTB/blob/main/Delivery/Delivery_website.png)


4) Hereafter I thought of performing a directory scan via gobuster to check if there are any other directories present. I found this...

![](assets/images/Luanne1.png)

5) After I went over to /forecast, I found a JSON message which said the following...

![](assets/images/Luanne_forecast.png)

city=list?..I guess it must be a parameter that is being passed to the URL

what if we pass a reverse shell instead of list to the variable city?...worth a shot..


# Foothold

So I constructed a reverse shell payload and then url-encoded it.

![](assets/images/Luanne_shell.png)


![](assets/images/Luanne_urlencode.png)



and i passed this to the city variable in the URL and opened a netcat listener on port 4242 on my local machine.

I got a shell as httpd thereafter. And then I checked what files were present in the current directory, I found a .htpasswd file which were the credentials needed to access the web service on port 80.

![](assets/images/Luanne_webapiuser.png)

I used John to crack the hash with rockyou as the dictionary and found the password. 

![](assets/images/Luanne2.png)

Thereafter I accessed the web service served on port 80 by inputting the credentials found.
 
![](assets/images/Luanne_web80.png)

There I saw some usual things,which I already have tried once before.

# Lateral Movement
I then remembered, that when I got the authentication error, I had also got a redirection link which was running on localhost.
I checked if the process was actually running on the server or no using the netstat -aan command.

![](assets/images/Luanne_netstat.png)


After a little enumeration (in the /home directory) I found out that there is a user named r.michaels, so I thought of CURLing the ssh keys of r.michaels using the webapi_user credentials obtained previously.
 
 ![](assets/images/Luanne3.png)
 
Boom! I got the id_rsa keys of r.michaels user. I saved it with a 600 permission (search file permissions on google for further details) using the chmod command on my local machine. 
 
Post which, I was ready to login into the user..

And we've logged in as r.michaels

![](assets/images/LuanneUSERFLAG.png)

Wuhuu...you got the user flag now!
 

# Privilege Escalation

Moving forward to root, there is a backup file which is encrypted so let’s decrypt it and then extract it.

![](assets/images/LuanneZipEnc.png)


![](assets/images/LuanneHash2.png)

We got another hash..hmm..resorting to the previous ways (like we used john in the previous case) we'll extract the hash as shown.

![](assets/images/Luanne4.png)

With the new found password, I tried use sudo su, but it didn’t work.

It was a easy machine so I thought this may be the password of root but the machine did not have sudo.

![](assets/images/LuanneSudo.png)

But I searched for the alternative of sudo in openBSD and got to know that we can use doas. And it worked...

THE END.
