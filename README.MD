## Changing the firmware for a Sierra MC8335/Gobi3000

Unfortunately this seems to require Windows, and I haven't been able to get it to work in a VM with passthrough.

From Windows run:

```
 cd C:\Program Files (x86)\Sierra Wireless Inc\Gobi\
 GobiQDLService
 GobiImgMgr -FWUPDATE AUTO -DEFAULTIMAGE 4
```

Image 4 is T-Mobile, 6 is Generic GSM
