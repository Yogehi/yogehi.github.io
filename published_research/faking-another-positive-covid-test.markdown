---
layout: page
title: Faking a Positive COVID Test
permalink: /published-research/faking-another-positive-covid-test
---

#### Original post: [https://labs.f-secure.com/blog/faking-another-positive-covid-test/](https://labs.f-secure.com/blog/faking-another-positive-covid-test/)

## Introduction

WithSecure conducted research into the Cue Health Home COVID-19 Test with the intention of finding methods to create fraudulent COVID-19 test results. This device was chosen because the reader unit used Bluetooth to send test results to the patient's phone. As for the outcome of this research, WithSecure was successful in falsifying a COVID-19 test result and obtaining a certificate verifying that this COVID-19 test result was valid.

This article will go over the technical details of how this research was conducted against the following application and firmware versions:

* Cue Health Android application com.cuehealth.healthapp version 1.4.3.134
* Cue Reader firmware version 0.17.6

![yay](/assets/published_research/faking-another-positive-covid-test_1.jpg)

## Overview: Reader and Cartridge

Before diving into the technical details, we will give a brief overview of the test components and how it works. Two components are required to take a COVID-19 test by Cue Health, the Cue Reader and the Test Cartridge.

The Cue Reader unit contains a heater, gyroscope, and the necessary hardware to communicate with a user's phone via Bluetooth. When taking a test, the Test Cartridge is inserted into the Cue Reader and the test itself is conducted within the Test Cartridge. The data is then passed to the reader, which is then transmitted to the user's phone.

![yay](/assets/published_research/faking-another-positive-covid-test_2.jpg)

Each COVID-19 Test Cartridge comes with a nasal swab which is inserted into the cartridge. The Test Cartridge contains liquid which is released when the nasal swab is inserted, and gathers in one area in the cartridge where the test is conducted.

![yay](/assets/published_research/faking-another-positive-covid-test_3.jpg)

## Technical Details: Bluetooth Traffic and Protobuf

Now that we have established a brief overview of the device, now we can dig into how the Bluetooth traffic is structured. The Cue Reader communicated with the Android app via the Protobuf protocol over Bluetooth. This has the following general structure packet:

```
08 XX XX XX YY ZZ ZZ ZZ ZZ ZZ ZZ ZZ
```

* XX – packet count in Protobuf
* YY – the packet type (Cartridge Status, Test Result, etc)
* ZZ – the data itself

Cue Health implemented a Protobuf parser in class `b.l.h.o$b` method `y()`. This implementation can be seen below:

```
public int y() throws java.io.IOException {
            int var3_bytePosition = this.i;
            int var1_payloadLength = this.g;
            if (var1_payloadLength != var3_bytePosition) {
                byte[] var6 = this.e;
                int var2_bytePositionPlus1 = var3_bytePosition + 1;
                byte var7 = var6[var3_bytePosition];
                if (var7 >= 0) {
                    this.i = var2_bytePositionPlus1;
                    return var7;
                }

                if (var1_payloadLength - var2_bytePositionPlus1 >= 9) {
                    var1_payloadLength = var2_bytePositionPlus1 + 1;
                    var3_bytePosition = var7 ^ var6[var2_bytePositionPlus1] << 7;
                    if (var3_bytePosition < 0) {
                        var2_bytePositionPlus1 = var3_bytePosition ^ -128;
                    } else {
                        var2_bytePositionPlus1 = var1_payloadLength + 1;
                        var3_bytePosition ^= var6[var1_payloadLength] << 14;
                        if (var3_bytePosition >= 0) {
                            var3_bytePosition ^= 16256;
                            var1_payloadLength = var2_bytePositionPlus1;
                            var2_bytePositionPlus1 = var3_bytePosition;
                        } else {
                            var1_payloadLength = var2_bytePositionPlus1 + 1;
                            var2_bytePositionPlus1 = var3_bytePosition ^ var6[var2_bytePositionPlus1] << 21;
                            if (var2_bytePositionPlus1 < 0) {
                                var2_bytePositionPlus1 ^= -2080896;
                            } else {
                                int var4 = var1_payloadLength + 1;
                                byte var5 = var6[var1_payloadLength];
                                var3_bytePosition = var2_bytePositionPlus1 ^ var5 << 28 ^ 266354560;
                                var2_bytePositionPlus1 = var3_bytePosition;
                                var1_payloadLength = var4;
                                if (var5 < 0) {
                                    int var8 = var4 + 1;
                                    var2_bytePositionPlus1 = var3_bytePosition;
                                    var1_payloadLength = var8;
                                    if (var6[var4] < 0) {
                                        var4 = var8 + 1;
                                        var2_bytePositionPlus1 = var3_bytePosition;
                                        var1_payloadLength = var4;
                                        if (var6[var8] < 0) {
                                            var8 = var4 + 1;
                                            var2_bytePositionPlus1 = var3_bytePosition;
                                            var1_payloadLength = var8;
                                            if (var6[var4] < 0) {
                                                var4 = var8 + 1;
                                                var2_bytePositionPlus1 = var3_bytePosition;
                                                var1_payloadLength = var4;
                                                if (var6[var8] < 0) {
                                                    var1_payloadLength = var4 + 1;
                                                    var2_bytePositionPlus1 = var3_bytePosition;
                                                    if (var6[var4] < 0) {
                                                        return (int) this.O();
                                                    }
                                                }
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }

                    this.i = var1_payloadLength;
                    return var2_bytePositionPlus1;
                }
            }

            return (int) this.O();
        }
```

Each received Bluetooth message is processed through the above parser, and the class `com.cuehealth.protobuf.reader.CueMessage` method `CueMessage()` contained the types of Bluetooth messages the Cue Health app recognized in decimal format. As an example, per the below code snippet, packet types in decimal `170` (`AA` in HEX) are Test Result type `data`:

```
private CueMessage(Abstracto oVar, g0 g0Var) throws a1 {
...
  case 170:
    builder3 = this.msgCase_ == 21 ? ((TestResults) this.msg_).toBuilder() : builder3;
    Abstractv1 w12 = oVar.w(TestResults.parser(), g0Var);
    this.msg_ = w12;
    if (builder3 != null) {
      builder3.mergeFrom((TestResults) w12);
      this.msg_ = builder3.buildPartial();
    }
    this.msgCase_ = 21;
    continue;
```

By then referencing class `com.cuehealth.protobuf.reader.TestResults` method `TestResults()`, it was found that the data type value in decimal `34` (`22` in HEX) represented the "Assay Result":

```
private TestResults(final o o, final g0 g0) throws a1 {
...
  case 34: {
    int n7 = n;
    if ((n & 0x8) != 0x8) {
      n2 = n;
      n2 = n;
      final ArrayList<AssayResult> assayResults_ = new ArrayList<AssayResult>();
      n2 = n;
      this.assayResults_ = assayResults_;
      n7 = (n | 0x8);
    }
    n2 = n7;
    this.assayResults_.add((AssayResult)o.w(AssayResult.parser(), g0));
    n = n7;
    continue;
  }
```

Following this to class `com.cuehealth.protobuf.reader.AssayResult` method `AssayResult()`, it was found that two types of data were used:

* `type` is represented by decimal value `8` (`8` in HEX)
* `code` is represented by decimal value `16` (`10` in HEX)

```
private AssayResult(Abstracto oVar, g0 g0Var) throws a1 {
...
  if (G == 8) {
    this.type_ = oVar.p();
  } else if (G == 16) {
    this.code_ = oVar.p();
```

The class `com.cuehealth.protobuf.reader.AssayResult` method `Code()` contained information regarding what `code` values were expected. `Negative` was represented by decimal value `3` (`3` in HEX):

```
public enum Code implements z0.Abstractc {
  INVALID(0),
  CONCENTRATION_LEVEL(1),
  POSITIVE(2),
  NEGATIVE(3),
  PASS(4),
  FAIL(5),
  UNRECOGNIZED(-1);
```

Within the same class, method `Type()` contained information on what `type` values were expected. The value for `CORONAVIRUS` was decimal value `19` (`13` in HEX):

```
public enum Type implements z0.Abstractc {
  NONE(0),
  INFLUENZA(1),
  INFLAMMATION_CRP(2),
  VITAMIN_D(3),
  TESTOSTERONE(4),
  FERTILITY(5),
  HIV_1_VIRAL_LOAD(6),
  ANC_PLUS_WBC(7),
  ZIKA_IG_M(9),
  ZIKA_VIRAL(10),
  PREGNANCY(11),
  CORTISOL(12),
  CHOLESTEROL_LDL(13),
  CHOLESTEROL_HDL(14),
  HEMOGLOBIN_A1C(15),
  RSV(17),
  CORONAVIRUS(19),
  RVC(20),
  UNRECOGNIZED(-1);
```

Putting all of this together, a negative Coronavirus test result most likely looked like the following in HEX:

```
08 XX XX XX AA YY YY YY YY 22 04 08 13 10 03 YY YY
```

* `08` is the start of the packet
* `XX` is the packet count in protobuf
* `AA` is a "Test Result" packet type
* `YY` is other "Test Result" data
* `22` is AssayResult data
* `04` is that the proceeding data is 4 bytes
* `08 13` is type: coronavirus
* `10 03` is code: negative

## Obtaining a Modified COVID-19 Test Result

WithSecure developed a Frida script which hooked into the Java class class `b.a.a.h.v3.k0$a` method `onCharacteristicChanged`. This method was chosen to be hooked into due to it being one of the first methods called when processing Bluetooth traffic within the application. The script intercepted each Bluetooth packet sent to the application and performed the following steps:

* The Bluetooth traffic is put through the Cue Health Protobuf parser to help determine what type of packet was received by the Cue Reader
* If the data type is "Test Result", then search for either HEX `220408131002` or `220408131003`
* Determine if the test result is positive or negative
* Switch the result (negative to positive, positive to negative)
* Put the modified data back into the app

Using this script, it was possible to change the test result within the Bluetooth data while being proctored by a Cue Health representative. Below is the output of the Frida script, showing that it had detected a negative test result and changed it to a positive result:

```
[#] TestResults
    hex payload: 08fefb02aa0188010a120a1000748a7ba3e34142bc4bd2f2a7494d9510021a2e0a1908918fbf30101320a382013080cff395063a0631383830334612119a010e080215008088c520012d0000c8c32204081310033a1097c89ec5abe5a7c40ee118c42f1291c240bcdeb190e22f48b6a8fb90e22f52120a104cd2f87ebf
    raw bytes length: 125
    bluetooth packet count: 48638
    packet type: 170 TestResults
    packet size: 136
      
    [#] COVID-19 NEGATIVE test found
       [#] changed COVID-19 NEGATIVE to POSITIVE
       [#] new hex payload: 08fefb02aa0188010a120a1000748a7ba3e34142bc4bd2f2a7494d9510021a2e0a1908918fbf30101320a382013080cff395063a0631383830334612119a010e080215008088c520012d0000c8c32204081310023a1097c89ec5abe5a7c40ee118c42f1291c240bcdeb190e22f48b6a8fb90e22f52120a104cd2f87ebf  
 
[#] TestResults
    hex payload: 08fffb02aa0188010a120a1000748a7ba3e34142bc4bd2f2a7494d9510021a2e0a1908918fbf30101320a382013080cff395063a0631383830334612119a010e080215008088c520012d0000c8c32204081310033a1097c89ec5abe5a7c40ee118c42f1291c240bcdeb190e22f48b6a8fb90e22f52120a104cd2f87ebf
    raw bytes length: 125
    bluetooth packet count: 48639
    packet type: 170 TestResults
    packet size: 136
      
    [#] COVID-19 NEGATIVE test found
       [#] changed COVID-19 NEGATIVE to POSITIVE
       [#] new hex payload: 08fffb02aa0188010a120a1000748a7ba3e34142bc4bd2f2a7494d9510021a2e0a1908918fbf30101320a382013080cff395063a0631383830334612119a010e080215008088c520012d0000c8c32204081310023a1097c89ec5abe5a7c40ee118c42f1291c240bcdeb190e22f48b6a8fb90e22f52120a104cd2f87ebf
```

Below is a picture of the certificate obtained by WithSecure after the test was taken:

![yay](/assets/published_research/faking-another-positive-covid-test_4.png)

Files for this research can be found here: [https://github.com/Yogehi/Cue-COVID-Test_Research-Files](https://github.com/Yogehi/Cue-COVID-Test_Research-Files)

## Contacting Cue Health and Remediation

WithSecure reached out to Cue Health and presented the findings above. Cue Health stated they have added checks server side which should detect manipulated test results.

Users should update their Cue Health mobile application at least the following versions:

* Android - 1.7.2
* iOS - 1.7.1

When using the latest versions of the mobile application, the user will be prompted to update their Cue Reader if it is not running at least the following firmware version: 0.17.8 (5)