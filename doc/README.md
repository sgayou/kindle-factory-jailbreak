# Kindle Factory Firmware Arbitrary File Injection
Details of the ;installHtml Kindle mechanism.

## Discovery
Recently, NiLuJe on the mobileread forums discovered that factory images could be recovered from a new Kindle. The factory firmware files were marked as deleted but not overwritten. As such, they can be recovered. Other members of the forum banded together to recover factory images from all of the devices and discovered that the Kindle would allow itself to downgrade using these special images. Normally, downgrades are blocked by the update mechanism.

The factory images have quite a bit of additional functionality, most likely for testing of the devices after they are assembled. The original plan was to release these and allow users to downgrade to a vulnerable firmware and use existing jailbreaks. This would have worked for older devices, but not newer ones such as the Oasis and Kindle 8th generation. I figured a factory test image wasn't secured and spent a bit of time digging through the firmware hoping we could get something generic to all devices.

## Auditing
As per the 5.6.5 JB [write-up](https://github.com/sgayou/kindle-5.6.5-jailbreak/tree/master/doc), we know that there are scripts referenced in debug_cmds.json that reference non-existing locations. I made the connection that these were probably wiped after some sort of factory provisioning process and immediately located them all in `/usr/local/bin/`.

There were a few interesting ones. One enabled some form of USB networking mechanism that may allow for trivial SSH access to the device. Another script controls iptables and can be used to disable the firewall entirely.

One of the most interesting is `";installHtml" : "/usr/local/bin/installHtmlViewer.sh",`

```
#!/bin/sh
# root path after main-htmlviewer.tar.gz is extracted
FILE_PATH=/mnt/us/transferfiles
 
# copy files from tar to folders on device
mntroot rw
cd /mnt/us
tar xvf /mnt/us/main-htmlviewer.tar.gz
…
```

### Background
It's important to note that a previous [jailbreak](http://www.mobileread.com/forums/showthread.php?p=1902438) by ixtab used what appears to be a flaw in the older version of Busybox Amazon is currently using on the Kindle. The tar function extracts files with absolute paths...absolutely. As the Busybox version Amazon is still using is from 2009, they probably didn't patch this flaw.

## Exploitation
We need to create a tar file with absolute paths. Let's tar up the developer key we used in the previous jailbreak. Create an absolute path tar with /etc/uks/pubdevkey01.pem and we're done. installHtmlViewer.sh runs as root and does a mntroot rw before extracting the file. Name it main-htmlviewer.tar.gz, have the user copy it to the user store, then instruct the user to run ;installHtml in the search bar.

Easy as that. Less than one hour to code execution. To be fair, my assumption is that factory images were never intended to be leaked. Running unsigned code on an OS probably designed to run unsigned code for test purposes isn't that impressive. The cool part was the discovery of the factory image and that it allowed for downgrades.

Thanks to NiLuJe and knc1.