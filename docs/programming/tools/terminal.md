# Terminal comments

Display running tasks (stacks of activities)
`adb shell dumpsys activity activities | sed -En -e '/Stack #/p' -e '/Running activities/,/Run #0/p'`
returns:
```
  Stack #124: type=standard mode=fullscreen
    Running activities (most recent first):
      TaskRecord{111dd6e #577 A=com.lyst.lystapp.debug U=0 StackId=124 sz=2}
        Run #1: ActivityRecord{40d22c8 u0 com.lyst.lystapp.debug/com.lyst.account.activity.AccountActivity t577}
        Run #0: ActivityRecord{7b11a74 u0 com.lyst.lystapp.debug/com.lyst.home.view.HomeActivity t577}
  Stack #0: type=home mode=fullscreen
    Running activities (most recent first):
      TaskRecord{e4998d #2 I=com.android.launcher3/com.android.a1launcher.AndroidOneLauncher U=0 StackId=0 sz=1}
        Run #0: ActivityRecord{755e254 u0 com.android.launcher3/com.android.a1launcher.AndroidOneLauncher t2}
```

<br><br>
`adb shell dumpsys activity activities | grep 'Hist #' | grep 'YOUR_PACKAGE_NAME'`
returns:
```
* Hist #1: ActivityRecord{b9b3bd3 u0 nimdokai.androidplayground/.MainActivity t73}
* Hist #0: ActivityRecord{586b9e4 u0 nimdokai.androidplayground/.MainActivity t73}
```
