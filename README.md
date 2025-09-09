[README.md](https://github.com/user-attachments/files/22231176/README.md)
### ConnectManager README

This library provides connection management for Classic Bluetooth, BLE, and Wi‑Fi, exposing friendly Java/Kotlin APIs (CompletableFuture and stream-like subscriptions).

- Package: `com.btBleTcp.connect`
- Core object: `ConnectManager` (Kotlin object; from Java call via `ConnectManager.INSTANCE`)
- Connection types: `ConnectType` (`WIFI(0) / BT(1) / BLE(2)`)

## Integration
- Put `connect-debug.aar` under `app/libs/`
- In `app/build.gradle` add:
```gradle
implementation files('libs/connect-debug.aar')
```

## Permissions
Android 12+ requires runtime permissions. Recommended manifest entries:
```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<uses-permission android:name="android.permission.BLUETOOTH_SCAN" tools:targetApi="31" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" tools:targetApi="31" />
```

Use `BluetoothPermissionManager` (provided by the library) on a `ComponentActivity` to request permissions:
```java
BluetoothPermissionManager pm = new BluetoothPermissionManager(this);
pm.checkAndRequestPermissionsWithCallback(
  () -> { /* granted */ },
  deniedList -> { /* denied */ }
);
```

## Initialization
```java
ConnectManager.INSTANCE.initManagerJava(ConnectType.BLE.getValue(), context)
  .thenAccept(ctx -> { /* ctx may be null */ })
  .exceptionally(e -> { /* handle error */ return null; });
```

- Re-initializing with a different type will auto close the previous port and create a new one.
- On success, `ConnectManager.globalContext` holds the current context and `port`.

## Scan and device stream
- Start scan (one-shot completion):
```java
ConnectManager.INSTANCE.startScanJava()
  .thenAccept(u -> { /* scan started */ });
```
- Subscribe device stream (continuous; returns `Set<BluetoothDevice>`):
```java
AutoCloseable sub = ConnectManager.INSTANCE.observeDeviceFlowJava(
  devices -> { /* update UI */ },
  error -> { /* handle error */ }
);
// later
sub.close();
```

## Connect / Reconnect / Close
- Connect by name or MAC address (MAC is recommended):
```java
ConnectManager.INSTANCE.connectJava(deviceAddress, context)
  .thenAccept(success -> { /* true when connected */ });
```
- Reconnect based on existing `globalContext.port`:
```java
ConnectManager.INSTANCE.reconnectJava()
  .thenAccept(success -> { /* true/false */ });
```
- Close connection:
```java
ConnectManager.INSTANCE.closeJava()
  .thenAccept(code -> { /* 0 = success, otherwise failure */ });
```

## Send and Receive
- Write (timeout in ms):
```java
ConnectManager.INSTANCE.writeJava(data, 3000L)
  .thenAccept(success -> { /* true when sent */ });
```
- Read once (Future of one response, timeout in ms):
```java
ConnectManager.INSTANCE.readJava(3000L)
  .thenAccept(bytes -> { /* may be null */ });
```
- Subscribe continuous receive stream (`ByteArray`):
```java
AutoCloseable recv = ConnectManager.INSTANCE.receiveDataFlowJava(
  bytes -> { /* show message */ },
  error -> { /* handle error */ }
);
// later
recv.close();
```

## BLE device info
- Valid in BLE mode only:
```java
ConnectManager.INSTANCE.getBleDeviceInfoJava()
  .thenAccept(info -> { /* may be null */ });
```

## Global callbacks (optional)
These callbacks are synced to the current `port` when set:
```kotlin
ConnectManager.onDeviceDisconnect = { /* device disconnected */ }
ConnectManager.onBluetoothDisconnect = { /* bluetooth disabled/disconnected */ }
ConnectManager.onHandleRssiUpdate = { rssi -> /* RSSI update */ }
```

## Error handling and threading
- All `*Java()` methods run work on IO and complete the `Future` on Main.
- Add `.exceptionally(...)` to handle errors; consider `.orTimeout(...)` to avoid hanging chains.

## Minimal Java walkthrough
```java
// 1) Permissions
BluetoothPermissionManager pm = new BluetoothPermissionManager(this);
pm.checkAndRequestPermissionsWithCallback(
  () -> {
    // 2) Init
    ConnectManager.INSTANCE.initManagerJava(ConnectType.BLE.getValue(), getApplicationContext())
      // 3) Start scan
      .thenCompose(ctx -> ConnectManager.INSTANCE.startScanJava())
      // 4) Subscribe devices
      .thenAccept(u -> {
        AutoCloseable deviceSub = ConnectManager.INSTANCE.observeDeviceFlowJava(
          devices -> {
            // update UI; on click -> 5) connect
          },
          error -> { /* show error */ }
        );
      });
  },
  denied -> { /* show denied */ }
);

// 5) Connect (prefer MAC address)
ConnectManager.INSTANCE.connectJava(deviceAddress, getApplicationContext())
  .thenAccept(success -> { /* go to communication page */ });

// 6) Send/Receive
ConnectManager.INSTANCE.writeJava("hello\r\n".getBytes(StandardCharsets.UTF_8), 3000L);
AutoCloseable recvSub = ConnectManager.INSTANCE.receiveDataFlowJava(
  bytes -> { /* show */ },
  error -> { /* show */ }
);

// 7) Reconnect/Close
ConnectManager.INSTANCE.reconnectJava();
ConnectManager.INSTANCE.closeJava();
```

## FAQ
- No callback in `thenCompose/thenAccept`: usually missing runtime permissions, Bluetooth off, or `initManager` returned null. Add `.orTimeout(...)` and logs to locate the stage.
- Reconnect failed with `Max reconnection attempts reached`: fall back to a full `connectJava(address, context)`.
- Device name may be null: prefer `device.getAddress()` for identity, guard `getName()` with null checks.

## Kotlin quick start
```kotlin
class MainActivity : ComponentActivity() {
  private lateinit var pm: BluetoothPermissionManager
  private var deviceSub: AutoCloseable? = null

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    pm = BluetoothPermissionManager(this)

    pm.checkAndRequestPermissionsWithCallback(
      Runnable {
        ConnectManager.initManager(ConnectType.BLE.value, applicationContext)
        ConnectManager.startScanJava()
          .thenAccept {
            deviceSub?.close()
            deviceSub = ConnectManager.observeDeviceFlowJava(
              { devices ->
                // update UI list
              },
              { error -> /* show error */ }
            )
          }
      },
      java.util.function.Consumer { denied -> /* show denied */ }
    )
  }

  override fun onDestroy() {
    deviceSub?.close()
    super.onDestroy()
  }
}
```

## Android 12/13 (API 31/33) notes
- Request runtime permissions at usage time:
  - Android 12+: `BLUETOOTH_SCAN`, `BLUETOOTH_CONNECT`
  - Android 6–11: `ACCESS_FINE_LOCATION` for BLE scan
- Declare bluetooth permissions in Manifest as shown above. For Android 12+, keep `tools:targetApi="31"` to avoid install-time warnings.
- Ensure Bluetooth and Location are enabled when scanning for BLE devices (system requirement for BLE scanning visibility).
- Consider adding a user-facing rationale before permission requests for better acceptance.


