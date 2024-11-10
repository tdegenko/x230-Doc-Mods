# Intro

Based on the [Skulls](//github.com/merge/skulls/tree/master/x230) & 
[Heads](//github.com/osresearch/heads-wiki/blob/master/Installing-Heads.md)

The x230 splits its 12 MB SPI Flash across two chips. A 4 MB Macronix chip (Top, near screen) 
containing the BIOS/UEFI and an 8 MB Winbond chip containing everything else.
Both chips have the same pin-out as follows.

```
Screen (furthest from you)
             ______
  MOSI  5 --|      |-- 4  GND
   CLK  6 --|      |-- 3  N/C
   N/C  7 --|      |-- 2  MISO
   VCC  8 --|______|-- 1  CS

Edge (closest to you)
```


# Flashing with a Pi

When flashing from a Raspberry Pi, that has the following pin-out:

```
  Edge of pi (furthest from you)
              (UART)
L           GND TX  RX                           CS
E            |   |   |                           |
F +---------------------------------------------------------------------------------+
T |  x   x   x   x   x   x   x   x   x   x   x   x   x   x   x   x   x   x   x   x  |
  |  x   x   x   x   x   x   x   x   x   x   x   x   x   x   x   x   x   x   x   x  |
E +----------------------------------^---^---^---^-------------------------------^--+
D                                    |   |   |   |                               |
G                                   3.3V MOSIMISO|                              GND
E                                 (VCC)         CLK
  Body of Pi (closest to you)

```

## Setup

Load drivers:
```
> modprobe spi_bcm2835
> modprobe spidev
```

## Flash Coreboot

### Splitting Coreboot

> dd if=coreboot.rom of=coreboot-top.rom bs=1M skip=8
> dd if=coreboot.rom of=coreboot-bottom.rom bs=1M count=8

### Flashing

Coreboot goes on the 4MB chip, which is the chip nearest screen. So, connect the 
test clip to the chip and do the following.

Check if chip detected:

```
> flashrom -p linux_spi:dev=/dev/spidev0.0
  Using clock_gettime for delay loops (clk_id: 1, resolution: 1ns).
  Found Macronix flash chip "MX25L3205(A)" (4096 kB, SPI) on linux_spi.
  Found Macronix flash chip "MX25L3205D/MX25L3208D" (4096 kB, SPI) on linux_spi.
  Found Macronix flash chip "MX25L3206E/MX25L3208E" (4096 kB, SPI) on linux_spi.
  Found Macronix flash chip "MX25L3273E" (4096 kB, SPI) on linux_spi.
  Multiple flash chip definitions match the detected chip(s): "MX25L3205(A)", "MX25L3205D/MX25L3208D", "MX25L3206E/MX25L3208E", "MX25L3273E"
  Please specify which chip definition to use with the -c  option.
```

Dump original bios:

    > flashrom -p linux_spi:dev=/dev/spidev0.0 -c "MX25L3206E/MX25L3208E" -r flash01.bin

Twice:

    > flashrom -p linux_spi:dev=/dev/spidev0.0 -c "MX25L3206E/MX25L3208E" -r flash02.bin

Compare checksums:

    > md5sum flash*.bin

Flash coreboot to chip:

    > flashrom -p linux_spi:dev=/dev/spidev0.0 -c "MX25L3206E/MX25L3208E" --write top.rom

## Disable Intel ME

Note that this will cause issues if you try to go back to the stock bios. 
You will not be able to change any settings.

You will need [me_cleaner](//github.com/corna/me_cleaner) for this.

Connect the test clip to the chip farther from the screen and do the following.

Check if chip detected:

```
> flashrom -p linux_spi:dev=/dev/spidev0.0
  flashrom v0.9.9-47-gaa91d5c on Linux 4.9.35-v7+ (armv7l)
  flashrom is free software, get the source code at https://flashrom.org
   
  Using clock_gettime for delay loops (clk_id: 1, resolution: 1ns).
  Found Winbond flash chip "W25Q64.V" (8192 kB, SPI) on linux_spi.
  No operations were specified.
```

Dump ME chip:

    > flashrom -p linux_spi:dev=/dev/spidev0.0 -r flash_me01.bin

Twice:

    > flashrom -p linux_spi:dev=/dev/spidev0.0 -r flash_me02.bin

Compare checksums:

    > md5sum flash_me*.bin

Combine into full bios image as me_cleaner needs a full bios file to work:

    > cat flash_me01.bin flash01.bin > full_bios.bin

Apply me_cleaner:

    > python me_cleaner full_bios.bin

Take bottom 8MB for bottom chip:

    > dd of=bottom.rom bs=1M if=full_bios.bin count=8

Flash bottom chip:

    > flashrom -p linux_spi:dev=/dev/spidev0.0 -w bottom.rom -c MX25L6406E/MX25L6408E


