### jinfo (取得jvm參數、flag、system properties)

* 用法: jinfo \[ option ] pid、jinfo \[ option ] executable core 、jinfo \[ option ] \[ servier-id ] remote-hostname-or-IP


| options | 描述                                  |
|-----|-------------------------------------|
| no-option    | 顯示command-line flag、system property |
| -flag name    | 顯示特定flag                            |
| -flags    | 顯示command-line flag                         |
| -sysprops    | 顯示system property                         |

<br><br>

| Dump Type | Use Case  | Contains  |
|-----------|-----------|-----------|
| Heap Dump | Diagnose memory issues like OutOfMemoryError  | Snapshot of objects in the Java heap  |
| Thread Dump | Troubleshoot performance issues, thread deadlocks, and infinite loops  | Snapshot of all thread states in the JVM  |
| Core Dump | Debug crashes caused by native libraries  | Process state when JVM crashes  |