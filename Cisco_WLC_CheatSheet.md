## Cisco WLC
---

### Client Trace in AP
---
```markup
# config ap client-trace address add MAC
# config ap client-trace filter all enable
# config ap client-trace output console-log enable
# config ap client-trace start
# term mon
# config ap client-trace stop
```

### Client Trace on WLC
---
```markup
# show client detail MAC
# debug client MAC
```
### Add/Remove/Description Disabled clients
---
```c
(Cisco Controller) >config exclusionlist
add            Creates a local exclusion-list entry.
delete         Deletes a local exclusion-list entry.
description    Sets the description for an exclusion-list entry.
(Cisco Controller) config exclusionlist delete MAC		<== Removes MAC from exclusion list
```

### WLC Upgrade
---
```markup
(Cisco Controller) >transfer download datatype code

(Cisco Controller) >transfer download filename AIR-CT5500-K9-8-3-143-0.aes

(Cisco Controller) >transfer download path .

(Cisco Controller) >transfer download serverip 10.240.40.250

(Cisco Controller) >transfer download mode ftp 

(Cisco Controller) >transfer download start

Mode............................................. FTP   
Data Type........................................ Code          
FTP Server IP.................................... 10.240.40.250
FTP Server Port.................................. 21
FTP Path......................................... ./
FTP Filename..................................... AIR-CT5500-K9-8-3-143-0.aes
FTP Username..................................... ftpwlc
FTP Password..................................... *********

This may take some time.
Are you sure you want to start? (y/N) y

FTP Code transfer starting.
FTP receive complete... extracting components.
Checking Version Built.
Image version check passed.
Informing the standby to start the transfer download process
Waiting for the Transfer & Validation result from Standby.
Standby - Standby receive complete... extracting components.
Standby - Checking Version Built.
Standby - Image version check passed.
Executing backup script.
Writing new RTOS to flash disk.
Standby - Writing new FP to flash disk.
Writing new FP to flash disk.
Standby - Writing new AP Image Bundle to flash disk.
Writing new AP Image Bundle to flash disk.
Writing AVC Files to flash disk.
Standby - Writing AVC Files to flash disk.
Executing fini script.
Reading AP IMAGE version info.
FTP File transfer successful on Active Controller
Standby - Executing fini script.
File transfer is successful 
Reboot the controller for update to complete 
Optionally, pre-download the image to APs before rebooting to reduce network downtime.
Transfer Download complete on Active & Standby
(Cisco Controller) >config ap image predownload primary all
(Cisco Controller) >show ap image all
Total number of APs.............................. 42
Number of APs
	Initiated....................................... 0
	Downloading..................................... 0
	Predownloading.................................. 0
	Completed predownloading........................ 42
	Not Supported................................... 0
	Failed to Predownload........................... 0
                                                 Predownload     Predownload                                  Flexconnect
AP Name            Primary Image  Backup Image   Status          Version        Next Retry Time  Retry Count  Predownload
------------------ -------------- -------------- --------------- -------------- ---------------- ------------ ---------
AP_XXX             8.0.152.0      8.3.143.0      Complete        8.3.143.0      NA               NA          
(Cisco Controller) >config ap image swap all
(Cisco Controller) >show ap image all
Total number of APs.............................. 42
Number of APs
	Initiated....................................... 0
	Downloading..................................... 0
	Predownloading.................................. 0
	Completed predownloading........................ 42
	Not Supported................................... 0
	Failed to Predownload........................... 0
                                                 Predownload     Predownload                                  Flexconnect
AP Name            Primary Image  Backup Image   Status          Version        Next Retry Time  Retry Count  Predownload
------------------ -------------- -------------- --------------- -------------- ---------------- ------------ ---------
AP_XXX             8.3.143.0      8.0.152.0      Complete        8.3.143.0      NA               NA          
(Cisco Controller) >show boot
Primary Boot Image............................... 8.3.143.0 (default)
Backup Boot Image................................ 8.0.152.0 (active)
(Cisco Controller) >reset system both 
The system has unsaved changes.
Would you like to save them now? (y/N) y
Configuration Saved!
System will now restart! 
```
