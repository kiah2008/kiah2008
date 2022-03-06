# query features
```java
PackageManager pm = getPackageManager();
boolean claimsLowLatencyFeature = pm.hasSystemFeature(PackageManager.FEATURE_AUDIO_LOW_LATENCY);
```

# query device info 
```java
String deviceInfo = "";
deviceInfo += "API " + Build.VERSION.SDK_INT + "\n";
deviceInfo += "Build " + Build.ID + " " +
        Build.VERSION.CODENAME + " " +
        Build.VERSION.RELEASE + " " +
        Build.VERSION.INCREMENTAL + "\n";
```
