[README.md](https://github.com/user-attachments/files/22269875/README.md)
# Bluetooth & BLE Connection Manager

A comprehensive Android library for managing Bluetooth Classic and BLE connections with friendly Java/Kotlin APIs using CompletableFuture and stream-like subscriptions.

## Features

- **Multi-Protocol Support**: Bluetooth Classic and BLE connections
- **Modern APIs**: CompletableFuture-based asynchronous operations
- **Stream Subscriptions**: Real-time data flow with reactive programming
- **Permission Management**: Automated runtime permission handling
- **Cross-Platform**: Works with both Java and Kotlin
- **Thread-Safe**: All operations properly handle threading

## Package Information

- **Package**: `com.btBleTcp.connect`
- **Core Classes**: 
  - `ConnectManager` (Kotlin object; access via `ConnectManager.INSTANCE` from Java)
  - `BluetoothPermissionManager` (Kotlin class)
- **Connection Types**: `ConnectType` enum (`WIFI(0)`, `BT(1)`, `BLE(2)`)

## Integration

### 1. Add the Library

Place the `connect-debug.aar` file under `app/libs/` and add to your `app/build.gradle`:

```gradle
dependencies {
    implementation files('libs/connect-debug.aar')
}
```

### 2. Required Permissions

Add these permissions to your `AndroidManifest.xml`:

```xml
<!-- Basic Bluetooth permissions -->
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />

<!-- Location permissions for BLE scanning (Android 6.0+) -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

<!-- New Bluetooth permissions (Android 12+) -->
<uses-permission android:name="android.permission.BLUETOOTH_SCAN" 
    tools:targetApi="31" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" 
    tools:targetApi="31" />
```

## BluetoothPermissionManager

The `BluetoothPermissionManager` class handles all Bluetooth-related runtime permissions automatically based on the Android version.

### Basic Usage

```java
// Initialize the permission manager
BluetoothPermissionManager permissionManager = new BluetoothPermissionManager(this);

// Request permissions with callbacks
permissionManager.checkAndRequestPermissionsWithCallback(
    () -> {
        // All permissions granted - proceed with Bluetooth operations
        Toast.makeText(this, "Permissions granted", Toast.LENGTH_SHORT).show();
    },
    deniedPermissions -> {
        // Some permissions denied - show user message
        Toast.makeText(this, "Permissions denied: " + deniedPermissions, Toast.LENGTH_LONG).show();
    }
);
```

### Advanced Methods

```java
// Check if all required permissions are already granted (without requesting)
boolean hasAllPermissions = permissionManager.hasAllRequiredPermissions();

// Get list of required permissions for current Android version
List<String> requiredPermissions = permissionManager.getRequiredPermissions();

// Check Android 8+ specific permissions
boolean hasAndroid8Permissions = permissionManager.checkAndroid8Permissions();
```

### Permission Requirements by Android Version

- **Android 12+ (API 31+)**: `BLUETOOTH_SCAN`, `BLUETOOTH_CONNECT`
- **Android 8+ (API 26+)**: `ACCESS_FINE_LOCATION`, `ACCESS_COARSE_LOCATION`
- **Android 6+ (API 23+)**: `ACCESS_FINE_LOCATION`
- **Android 5 and below**: No runtime permissions required

## ConnectManager

The `ConnectManager` is the core class for all connection operations. It's implemented as a Kotlin object, so from Java you access it via `ConnectManager.INSTANCE`.

### Initialization

```java
// Initialize with connection type and context
ConnectManager.INSTANCE.initManagerJava(ConnectType.BLE.getValue(), getApplicationContext())
    .thenAccept(context -> {
        // Initialization successful
        // context may be null - this is normal
    })
    .exceptionally(throwable -> {
        // Handle initialization error
        Log.e("ConnectManager", "Initialization failed", throwable);
        return null;
    });
```

**Note**: Re-initializing with a different connection type will automatically close the previous connection and create a new one.

### Device Discovery

#### Start Scanning

```java
// Start device discovery
ConnectManager.INSTANCE.startScanJava()
    .thenAccept(unused -> {
        // Scan started successfully
        Toast.makeText(this, "Scan started", Toast.LENGTH_SHORT).show();
    })
    .exceptionally(throwable -> {
        // Handle scan start error
        return null;
    });
```

#### Subscribe to Device Stream

```java
// Subscribe to continuous device updates
AutoCloseable deviceSubscription = ConnectManager.INSTANCE.observeDeviceFlowJava(
    devices -> {
        // Update UI with discovered devices
        Set<BluetoothDevice> deviceSet = (Set<BluetoothDevice>) devices;
        for (BluetoothDevice device : deviceSet) {
            String deviceInfo = device.getName() + " (" + device.getAddress() + ")";
            // Add to your device list
        }
    },
    error -> {
        // Handle device stream error
        Log.e("DeviceStream", "Error in device stream", error);
    }
);

// Don't forget to close the subscription
deviceSubscription.close();
```

### Connection Management

#### Connect to Device

```java
// Connect using device address (recommended) or name
String deviceAddress = "AA:BB:CC:DD:EE:FF"; // MAC address preferred
ConnectManager.INSTANCE.connectJava(deviceAddress, getApplicationContext())
    .thenAccept(success -> {
        if (Boolean.TRUE.equals(success)) {
            // Connection successful
            Toast.makeText(this, "Connected successfully", Toast.LENGTH_SHORT).show();
        } else {
            // Connection failed
            Toast.makeText(this, "Connection failed", Toast.LENGTH_SHORT).show();
        }
    })
    .exceptionally(throwable -> {
        // Handle connection error
        Log.e("Connection", "Connection error", throwable);
        return null;
    });
```

#### Reconnect

```java
// Reconnect using existing connection context
ConnectManager.INSTANCE.reconnectJava()
    .thenAccept(success -> {
        if (Boolean.TRUE.equals(success)) {
            Toast.makeText(this, "Reconnected successfully", Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(this, "Reconnection failed", Toast.LENGTH_SHORT).show();
        }
    });
```

#### Close Connection

```java
// Close the current connection
ConnectManager.INSTANCE.closeJava()
    .thenAccept(resultCode -> {
        if (resultCode == 0) {
            Toast.makeText(this, "Connection closed", Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(this, "Close failed with code: " + resultCode, Toast.LENGTH_SHORT).show();
        }
    });
```

### Data Communication

#### Send Data

```java
// Send data with timeout
String message = "Hello, Device!\r\n";
byte[] data = message.getBytes(StandardCharsets.UTF_8);

ConnectManager.INSTANCE.writeJava(data, 3000L) // 3 second timeout
    .thenAccept(success -> {
        if (Boolean.TRUE.equals(success)) {
            Toast.makeText(this, "Data sent successfully", Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(this, "Failed to send data", Toast.LENGTH_SHORT).show();
        }
    })
    .exceptionally(throwable -> {
        Log.e("SendData", "Send error", throwable);
        return null;
    });
```

#### Receive Data (One-time)

```java
// Read data once with timeout
ConnectManager.INSTANCE.readJava(5000L) // 5 second timeout
    .thenAccept(receivedData -> {
        if (receivedData != null && receivedData.length > 0) {
            String message = new String(receivedData, StandardCharsets.UTF_8);
            Toast.makeText(this, "Received: " + message, Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(this, "No data received", Toast.LENGTH_SHORT).show();
        }
    });
```

#### Subscribe to Continuous Data Stream

```java
// Subscribe to continuous data reception
AutoCloseable dataSubscription = ConnectManager.INSTANCE.receiveDataFlowJava(
    receivedData -> {
        if (receivedData != null && receivedData.length > 0) {
            String message = new String(receivedData, StandardCharsets.UTF_8);
            // Process received data
            runOnUiThread(() -> {
                // Update UI with received data
                textView.append("Received: " + message + "\n");
            });
        }
    },
    error -> {
        Log.e("DataStream", "Error in data stream", error);
        runOnUiThread(() -> {
            Toast.makeText(this, "Data stream error: " + error.getMessage(), Toast.LENGTH_LONG).show();
        });
    }
);

// Close subscription when done
dataSubscription.close();
```

### BLE Device Information

```java
// Get BLE device information (BLE mode only)
ConnectManager.INSTANCE.getBleDeviceInfoJava()
    .thenAccept(deviceInfo -> {
        if (deviceInfo != null) {
            // Process device information
            String info = "BLE Device Info: " + deviceInfo.toString();
            Toast.makeText(this, "Device info retrieved", Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(this, "No device info available", Toast.LENGTH_SHORT).show();
        }
    });
```

### Global Callbacks (Optional)

Set global callbacks for connection events:

```kotlin
// In Kotlin
ConnectManager.onDeviceDisconnect = { 
    // Device disconnected
}

ConnectManager.onBluetoothDisconnect = { 
    // Bluetooth disabled/disconnected
}

ConnectManager.onHandleRssiUpdate = { rssi -> 
    // RSSI update received
}
```

## Complete Example

Here's a complete example showing the typical workflow:

```java
public class MainActivity extends AppCompatActivity {
    
    private BluetoothPermissionManager permissionManager;
    private AutoCloseable deviceSubscription;
    private AutoCloseable dataSubscription;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        // Initialize permission manager
        permissionManager = new BluetoothPermissionManager(this);
        
        // Request permissions and start Bluetooth operations
        permissionManager.checkAndRequestPermissionsWithCallback(
            this::startBluetoothOperations,
            this::handlePermissionDenied
        );
    }
    
    private void startBluetoothOperations() {
        // 1. Initialize connection manager
        ConnectManager.INSTANCE.initManagerJava(ConnectType.BLE.getValue(), getApplicationContext())
            // 2. Start scanning
            .thenCompose(ctx -> ConnectManager.INSTANCE.startScanJava())
            // 3. Subscribe to device stream
            .thenAccept(u -> {
                deviceSubscription = ConnectManager.INSTANCE.observeDeviceFlowJava(
                    this::updateDeviceList,
                    this::handleDeviceStreamError
                );
            })
            .exceptionally(this::handleInitializationError);
    }
    
    private void connectToDevice(String deviceAddress) {
        ConnectManager.INSTANCE.connectJava(deviceAddress, getApplicationContext())
            .thenAccept(success -> {
                if (Boolean.TRUE.equals(success)) {
                    // Start data reception
                    startDataReception();
                    // Navigate to communication activity
                    startActivity(new Intent(this, CommunicationActivity.class));
                }
            });
    }
    
    private void startDataReception() {
        dataSubscription = ConnectManager.INSTANCE.receiveDataFlowJava(
            this::processReceivedData,
            this::handleDataStreamError
        );
    }
    
    private void processReceivedData(byte[] data) {
        if (data != null && data.length > 0) {
            String message = new String(data, StandardCharsets.UTF_8);
            runOnUiThread(() -> {
                // Update UI with received data
            });
        }
    }
    
    private void sendData(String message) {
        byte[] data = (message + "\r\n").getBytes(StandardCharsets.UTF_8);
        ConnectManager.INSTANCE.writeJava(data, 3000L)
            .thenAccept(success -> {
                if (Boolean.TRUE.equals(success)) {
                    Toast.makeText(this, "Data sent", Toast.LENGTH_SHORT).show();
                }
            });
    }
    
    @Override
    protected void onDestroy() {
        // Clean up subscriptions
        if (deviceSubscription != null) {
            try { deviceSubscription.close(); } catch (Exception ignored) {}
        }
        if (dataSubscription != null) {
            try { dataSubscription.close(); } catch (Exception ignored) {}
        }
        super.onDestroy();
    }
    
    // Error handling methods
    private void handlePermissionDenied(List<String> deniedPermissions) {
        Toast.makeText(this, "Permissions denied: " + deniedPermissions, Toast.LENGTH_LONG).show();
    }
    
    private Void handleInitializationError(Throwable throwable) {
        runOnUiThread(() -> {
            Toast.makeText(this, "Initialization failed: " + throwable.getMessage(), Toast.LENGTH_LONG).show();
        });
        return null;
    }
    
    private void handleDeviceStreamError(Throwable error) {
        runOnUiThread(() -> {
            Toast.makeText(this, "Device stream error: " + error.getMessage(), Toast.LENGTH_LONG).show();
        });
    }
    
    private void handleDataStreamError(Throwable error) {
        runOnUiThread(() -> {
            Toast.makeText(this, "Data stream error: " + error.getMessage(), Toast.LENGTH_LONG).show();
        });
    }
}
```

## Threading and Error Handling

### Threading Model

- All `*Java()` methods execute work on background threads (IO)
- CompletableFuture callbacks complete on the main thread
- Always use `runOnUiThread()` when updating UI from callbacks

### Error Handling Best Practices

```java
// Always add error handling
ConnectManager.INSTANCE.someMethod()
    .thenAccept(result -> {
        // Handle success
    })
    .exceptionally(throwable -> {
        // Handle error
        Log.e("TAG", "Operation failed", throwable);
        return null;
    })
    .orTimeout(10, TimeUnit.SECONDS); // Add timeout to prevent hanging
```

## Troubleshooting

### Common Issues

1. **No callbacks in `thenCompose/thenAccept`**:
   - Check if runtime permissions are granted
   - Verify Bluetooth is enabled
   - Ensure `initManager` didn't return null
   - Add `.orTimeout(...)` and logs to identify the issue

2. **Reconnect fails with "Max reconnection attempts reached"**:
   - Fall back to a full `connectJava(address, context)` call
   - Check if the device is still available

3. **Device name is null**:
   - Prefer `device.getAddress()` for device identification
   - Always guard `device.getName()` with null checks

4. **Permission issues on Android 12+**:
   - Ensure `BLUETOOTH_SCAN` and `BLUETOOTH_CONNECT` permissions are declared
   - Use `BluetoothPermissionManager` for proper permission handling

### Debug Tips

- Enable Bluetooth logs in Android Studio
- Use `adb logcat` to monitor system Bluetooth logs
- Test on different Android versions
- Verify device compatibility

## Best Practices

1. **Always close subscriptions** in `onDestroy()` or when no longer needed
2. **Use MAC addresses** instead of device names for connections
3. **Handle null values** from device information methods
4. **Add timeouts** to prevent hanging operations
5. **Test permission flows** on different Android versions
6. **Use proper error handling** for all async operations

## License

This library is provided as-is for development and testing purposes.

## Support

For issues and questions, please refer to the library documentation or contact the development team.
