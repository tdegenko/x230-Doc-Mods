## Coreboot

[Flashing Coreboot is covered here.](/FlashingCoreboot.md)

## bootsplash.jpg

This is a custom bootsplash for coreboot.
TODO: Coreboot documentation.

## Changing the firmware for a Sierra MC8335/Gobi3000

Unfortunately this seems to require Windows, and I haven't been able to get it to work in a VM with passthrough.

From Windows run:

```
 cd C:\Program Files (x86)\Sierra Wireless Inc\Gobi\
 GobiQDLService
 GobiImgMgr -FWUPDATE AUTO -DEFAULTIMAGE 4
```

Image 4 is T-Mobile, 6 is Generic GSM


## x220 Keyboard

Keyboard swapped out for the x220 and embedec controller flashed using tools from [here](//github.com/hamishcoleman/thinkpad-ec).
