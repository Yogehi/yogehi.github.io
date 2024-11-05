---
layout: page
title: Faking a Positive COVID Test
permalink: /published-research/faking-a-positive-covid-test
---

#### Original post: [https://labs.f-secure.com/blog/faking-a-positive-covid-test](https://labs.f-secure.com/blog/faking-a-positive-covid-test)

## Introduction

F-Secure conducted research into the Ellume COVID-19 Home Test with the intention of finding methods to fake a COVID test result. This device was chosen specifically because of the Bluetooth device that is used as the analyzer for testing a nasal sample. As for the outcome of this research, F-Secure was successful in falsifying a COVID test result, and obtained a certificate verifying the COVID test result.

This article will go over the technical details of how this research was conducted.

<div align="center">
    <img src="/assets/published_research/faking-a-positive-covid-test_1.png">
</div>

## The Analyzer And Bluetooth Traffic

The analyzer itself was a custom board and a standard Lateral Flow test, with the custom board determining if the user was COVID positive or negative. This determination is based on what the "two lines" look like on the Later Flow test strip. The analyzer would then inform the companion mobile app if the user was COVID positive or negative.

On the board is some LEDs to light up the test strip and some lens to read the test strip result lines. A hole on the case itself can be used to put the nasal sample on the test strip.

<div align="center">
    <img src="/assets/published_research/faking-a-positive-covid-test_2.png">
</div>

The Android application contained an un-exported activity called `com.ellumehealth.homecovid.android/com.gsk.itreat.activities.BluetoothDebugActivity`. If you have root level access to your device, you can launch this activity to help interact with the analyzer over Bluetooth.

<div align="center">
    <img src="/assets/published_research/faking-a-positive-covid-test_3.png">
</div>

Using this activity, F-Secure deduced that there were two types of Bluetooth traffic that were most likely in charge of informing the mobile app if the user was COVID positive or negative:

* `STATUS`
* `MEASUREMENT_CONTROL_DATA`

Below is an example `STATUS` message:

```
64,-102,4,13,-119,85,0,11,4,1,0,1,0,-1,127,-19,13,103,-122
```

Breaking down the above traffic, the following information can be seen (the key to converting `STATUS` traffic to human readable data was found in the Android app, class `au.com.ellume.estick_sdk.messaging.comms.payloads.StatusPayload`):

```
Array[0-1]     64, -102              start of traffic
Array[2]       4                     the type of traffic (4 = STATUS)
Array[3]       13                    length of data in number of bytes
Array[4-7]     -119, 85, 0, 11       unique ID in unsigned byte values
Array[8]       4                     status of the test
Array[9]       1                     algorithm status
Array[10]      0                     test result
Array[11-12]   1, 0                  measurement count
Array[13-14]   -1, 127               time until result in unsigned byte values
Array[15-16]   -19, 13               time until the test is considered to be failed
Array[17-18]   103, -122             crc value in unsigned byte values
```

Below is an example `MEASUREMENT_CONTROL_DATA` message:

```
64,-102,12,24,-119,85,0,11,-98,0,-128,7,-81,36,0,4,80,24,0,0,62,2,0,19,25,-124,26,-119,-66,-43
```
Breaking down the above traffic, the following information can be seen (the key to converting `MEASUREMENT_CONTROL_DATA` traffic to human readable data was found in the Android app, class `au.com.ellume.estick_sdk.messaging.comms.payloads.MeasurementControlDataPayload`):

```
Array[0-1]     64, -102              start of traffic
Array[2]       12                    the type of traffic (12 = MEASUREMENT_CONTROL_DATA)
Array[3]       24                    length of data in number of bytes
Array[4-7]     -119, 85, 0, 11       unique ID in unsigned byte values
Array[8-9]     -98, 0                sequence number unsigned byte values
Array[10-11]   -128, 7               timestamp in unsigned byte values
Array[12-14]   -81, 36, 0            measurement of line 1
Array[15]      4                     algorithm status
Array[16-18]   80, 24, 0             measurement of line 2
Array[19]      0                     hardware status
Array[20-21]   62, 2                 dark frequency number in unsigned byte values
Array[22]      0                     test result
Array[23-24]   19, 25                c1 value in unsigned byte values
Array[25-26]   -124, 26              c2 value in unsigned byte values
Array[27]      -119                  checksum of measurement data in unsigned byte value
Array[28-19]   -66, -43              crc value in unsigned byte value
```

CRC values and checksums are calculated using two different algorithms. First an integer is calculated from the following example Java code, taken from the Android app, class `c` method `b(byte[])`:

```
// if request is MEASUREMENT_CONTROLDATA
//     checksum value calculated from bArr[4-27]
//     crc value calculated from bArr[0-(length-2)]
// if request is STATUS
//     crc value calculated from bArr[0-(length-2)]
public static int b(byte[] bArr) {
    int yayintyay = 0;
    for (byte b : bArr) {
        for (int i = 0; i < 8; i++) {
            boolean z = ((b >> (7 - i)) & 1) == 1;
            boolean z2 = ((yayintyay >> 15) & 1) == 1;
            yayintyay <<= 1;
            if (z ^ z2) {
                yayintyay ^= 4129;
            }
        }
     }
     yayintyay &= 65535;
     return yayintyay;
}
```

Then the integer value is converted to a byte array. The below snippet is taken from the Android app, class `au.com.ellume.estick_sdk.util.IntegerExtensions` method `toBytes(int)`:

```
public static byte[] toBytes(int i) {
    return ByteBuffer.allocate(4).order(ByteOrder.LITTLE_ENDIAN).putInt(i).array();
}
```

If a checksum value is being calculated, then the first byte array value is used. Otherwise, if a CRC value is being calculated, then the first and second byte array values are used.

## Modifying The Traffic

F-Secure determined that by changing only the byte value representing the "status of the test" in both `STATUS` and `MEASUREMENT_CONTROL_DATA` traffic, followed by calculating new CRC and checksum values, it was possible to alter the COVID test result before the Ellume app processes the data. There are multiple areas where you can hook into to modify BLE traffic. F-Secure created two PoC hooks for this research which hooks into the Android version of the Ellume app:

* A Frida script which hooks into class `EV` method `equals(Object)`
* A Xposed module which hooks into class `android.bluetooth.BluetoothGattCharacteristic` method `getValue()`

Below is output using the Frida script showing F-Secure successfully changing a negative test to positive:

```
[#] MEASUREMENT_CONTROL_DATA:
    [#] byte array: 64,-102,12,24,-119,85,0,11,88,2,-2,41,-105,48,0,7,-34,55,0,0,-66,1,1,93,34,58,35,-34,-5,-50
    [#] test result: 1 NEGATIVE
    [#] algorithm state: 7 RESULT_AVAILABLE
    [#] timestamp: 10750
    [#] unique id: 184571273 
    [#] line 1: 12439
    [#] line 2: 14302
    [#] hardware status: OK
    [#] sequence number: 600
    [#] darkfrequency: 446
    [#] given checksum: -34
    [-] Test was NEGATIVE, changing to POSITIVE
        [#] calculated new checksum and CRC value
        [#] NEW byte array: 64,-102,12,24,-119,85,0,11,88,2,-2,41,-105,48,0,7,-34,55,0,0,-66,1,2,93,34,58,35,68,-24,34
[#] STATUS: 
    [#] byte array: 64,-102,4,13,-119,85,0,11,5,7,1,88,2,0,0,-1,127,69,-83
    [#] result: 1 NEGATIVE
    [#] status: RESULT
    [#] algorithm state: 7 RESULT_AVAILABLE
    [#] unique id: 184571273
    [#] measure count: 600
    [#] time to result: 0
    [#] time to failure: 32767
    [-] Test was NEGATIVE, changing to POSITIVE
        [#] calculated new checksum and CRC value
        [#] NEW byte array: 64,-102,4,13,-119,85,0,11,5,7,2,88,2,0,0,-1,127,-57,117
```

Below is a screenshot of an email from Ellume confirming that the above COVID test was a positive test:

<div align="center">
    <img src="/assets/published_research/faking-a-positive-covid-test_4.png">
</div>

## Obtaining an Azova Certificate

At the time of this post's publication, the United States requires a negative COVID test, and the Ellume test is one option to provide proof of a negative test. If a customer chooses this option, they are required to have their test observed by the third party company Azova. To prove that F-Secure could fake a positive COVID test and obtain a certificate from Azova, the F-Secure Marketing Manager Alexandra Rinehimer took the COVID test while being supervised.

When it was time to take the test under the supervision of Azova, Alexandra used F-Secure’s Xposed Module PoC. After launching the Ellume app with the Xposed Module, Alex proceeded to take the test. Below is the output log from the test, showing that while analyzer 151131494 reported a negative result, the Xposed Module changed the result to positive:

```
2021-07-30 14:39:07.837 11090-11108/? I/EdXposed-Bridge: [#yay#] MEASUREMENT_CONTROL_DATA:
    [#yay#] byte array: [B@4899c09
    [#yay#] test result: 1 NEGATIVE
    [#yay#] algorithm state: 7 RESULT_AVAILABLE
    [#yay#] timestamp: 10490
    [#yay#] unique id: 151131494
    [#yay#] line 1: 12564
    [#yay#] line 2: 14354
    [#yay#] c1 value: 8570
    [#yay#] c2 value: 10169
    [#yay#] hardware status: OK
    [#yay#] sequence number: 573
    [#yay#] darkfrequency: 655
    [#yay#] given checksum: -99
    [#yay#] crc value: -72 -58
2021-07-30 14:39:07.837 11090-11108/? I/EdXposed-Bridge: [-yay-] Test was NEGATIVE, changing to POSITIVE
2021-07-30 14:39:07.837 11090-11108/? I/EdXposed-Bridge: [#yay#] calculated new checksum and CRC value
        [#yay#] NEW byte array: [B@4899c09
2021-07-30 14:39:07.912 11090-11108/? I/EdXposed-Bridge: [#yay#] STATUS: 
    [#yay#] byte array: [B@4185e0e
    [#yay#] result: 1 NEGATIVE
    [#yay#] status: RESULT
    [#yay#] algorithm state: 7 RESULT_AVAILABLE
    [#yay#] unique id: 151131494
    [#yay#] measure count: 573
    [#yay#] time to result: 0
    [#yay#] time to failure: 32767
    [#yay#] crc value: 111 17
2021-07-30 14:39:07.912 11090-11108/? I/EdXposed-Bridge: [-yay-] Test was NEGATIVE, changing to POSITIVE
2021-07-30 14:39:07.912 11090-11108/? I/EdXposed-Bridge: [#yay#] calculated new checksum and CRC value
    [#yay#] NEW byte array: [B@4185e0e
```

Below is the certificate that Alexandra was given from Azova:

<div align="center">
    <img src="/assets/published_research/faking-a-positive-covid-test_5.png">
</div>

Files for this research can be found here: [https://github.com/Yogehi/Ellume-COVID-Test_Research-Files](https://github.com/Yogehi/Ellume-COVID-Test_Research-Files)

## Contacting Ellume

F-Secure reached out to Ellume and presented the findings above. The following recommendations were made, which Ellume has stated have been implemented:

* Implement further analysis of results to flag spoofed data
* Implement additional obfuscation and OS checks in the Android app