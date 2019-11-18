---
title: 集成LeakCanary
date: 2019-11-15 11:00:52
category:
- Android
tags:
comment: true
---

# 集成
* 修改build.gradle

```gradle
dependencies {
  // debugImplementation because LeakCanary should only run in debug builds.
  debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.0-beta-3'
}
```

# 参考
* [https://square.github.io/leakcanary/](https://square.github.io/leakcanary/)
* [GitHub:square/leakcanary](https://github.com/square/leakcanary)