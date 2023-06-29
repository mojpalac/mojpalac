Hilt add complexity to UI testing when using Fragments
(additionally when using multi-module project),
in some cases it might be event impossible to implement (vide Lyst)
as it would require a refactor of whole architecture of the app.

# Running Test Instrumented Test via ADV

```
 adb shell am instrument -w -e package com.justeat.ordersui -e class com.justeat.ordersui.OrderListWithSidePanelScreenshotTest#badgeIsShownOnTabletsButNotOnPhonesForCancelledOrder -e annotation com.justeat.uitestutils.IsScreenshotTest com.justeat.ordersui.test/com.justeat.TestRunner
```

[more commands](https://developer.android.com/studio/test/command-line#am-instrument-options)