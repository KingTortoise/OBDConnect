# ConnectManager Management Class - Java & Kotlin Usage Examples

ConnectManager is a core management class for managing Bluetooth Classic and BLE connections, providing friendly Java and Kotlin APIs with CompletableFuture and stream-like subscriptions.

## Core Features

- **Multi-Protocol Support**: Bluetooth Classic and BLE connections
- **Modern APIs**: CompletableFuture-based asynchronous operations
- **Stream Subscriptions**: Real-time data flow and reactive programming
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

## ConnectManager API Details

### Initialization

#### Java Usage Example

```java
// Initialize connection manager
ConnectManager.INSTANCE.initManagerJava(ConnectType.BLE.getValue(), getApplicationContext())
    .thenAccept(context -> {
        // Initialization successful
        // context may be null - this is normal
        Log.d("ConnectManager", "Initialization successful");
    })
    .exceptionally(throwable -> {
        // Handle initialization error
        Log.e("ConnectManager", "Initialization failed", throwable);
        return null;
    });
```

#### Kotlin Usage Example

```kotlin
// Initialize connection manager
ConnectManager.initManager(ConnectType.BLE.value, applicationContext)
    .onSuccess { context ->
        // Initialization successful
        Log.d("ConnectManager", "Initialization successful")
    }
    .onFailure { throwable ->
        // Handle initialization error
        Log.e("ConnectManager", "Initialization failed", throwable)
    }
```

### Device Discovery

#### Start Scanning

**Java Usage Example:**

```java
// Start device discovery
ConnectManager.INSTANCE.startScanJava()
    .thenAccept(unused -> {
        // Scan started successfully
        Toast.makeText(this, "Scan started", Toast.LENGTH_SHORT).show();
    })
    .exceptionally(throwable -> {
        // Handle scan start error
        Log.e("Scan", "Scan start failed", throwable);
        return null;
    });
```

**Kotlin Usage Example:**

```kotlin
// Start device discovery
ConnectManager.startScan()
    .onSuccess {
        // Scan started successfully
        Toast.makeText(this, "Scan started", Toast.LENGTH_SHORT).show()
    }
    .onFailure { throwable ->
        // Handle scan start error
        Log.e("Scan", "Scan start failed", throwable)
    }
```

#### Subscribe to Device Stream

**Java Usage Example:**

```java
// Subscribe to continuous device updates
AutoCloseable deviceSubscription = ConnectManager.INSTANCE.getDeviceFlowJava(
    devices -> {
        // Update UI with discovered devices
        Set<BluetoothDevice> deviceSet = (Set<BluetoothDevice>) devices;
        for (BluetoothDevice device : deviceSet) {
            String deviceInfo = device.getName() + " (" + device.getAddress() + ")";
            // Add to device list
        }
    },
    error -> {
        // Handle device stream error
        Log.e("DeviceStream", "Device stream error", error);
    }
);

// Don't forget to close subscription
deviceSubscription.close();
```

**Kotlin Usage Example:**

```kotlin
// Subscribe to continuous device updates
val deviceFlow = ConnectManager.getDeviceFlow()
deviceFlow.collect { devices ->
    // Update UI with discovered devices
    devices.forEach { device ->
        val deviceInfo = "${device.name} (${device.address})"
        // Add to device list
    }
}
```

### Connection Management

#### Connect to Device

**Java Usage Example:**

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

**Kotlin Usage Example:**

```kotlin
// Connect using device address (recommended) or name
val deviceAddress = "AA:BB:CC:DD:EE:FF" // MAC address preferred
ConnectManager.connect(deviceAddress, applicationContext)
    .onSuccess { success ->
        if (success) {
            // Connection successful
            Toast.makeText(this, "Connected successfully", Toast.LENGTH_SHORT).show()
        } else {
            // Connection failed
            Toast.makeText(this, "Connection failed", Toast.LENGTH_SHORT).show()
        }
    }
    .onFailure { throwable ->
        // Handle connection error
        Log.e("Connection", "Connection error", throwable)
    }
```

#### Reconnect

**Java Usage Example:**

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

**Kotlin Usage Example:**

```kotlin
// Reconnect using existing connection context
ConnectManager.reconnect()
    .onSuccess { success ->
        if (success) {
            Toast.makeText(this, "Reconnected successfully", Toast.LENGTH_SHORT).show()
        } else {
            Toast.makeText(this, "Reconnection failed", Toast.LENGTH_SHORT).show()
        }
    }
    .onFailure { throwable ->
        Log.e("Reconnect", "Reconnection error", throwable)
    }
```

#### Close Connection

**Java Usage Example:**

```java
// Close current connection
ConnectManager.INSTANCE.closeJava()
    .thenAccept(unused -> {
        Toast.makeText(this, "Connection closed", Toast.LENGTH_SHORT).show();
    });
```

**Kotlin Usage Example:**

```kotlin
// Close current connection
ConnectManager.close()
```

### Data Communication

#### Send Data

**Java Usage Example:**

```java
// Send data with timeout
String message = "Hello, Device!\r\n";
byte[] data = message.getBytes(StandardCharsets.UTF_8);

ConnectManager.INSTANCE.writeJava(data, 3000L) // 3 second timeout
    .thenAccept(success -> {
        if (Boolean.TRUE.equals(success)) {
            Toast.makeText(this, "Data sent successfully", Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(this, "Data send failed", Toast.LENGTH_SHORT).show();
        }
    })
    .exceptionally(throwable -> {
        Log.e("SendData", "Send error", throwable);
        return null;
    });
```

**Kotlin Usage Example:**

```kotlin
// Send data with timeout
val message = "Hello, Device!\r\n"
val data = message.toByteArray(StandardCharsets.UTF_8)

ConnectManager.write(data, 3000L) // 3 second timeout
    .onSuccess { success ->
        if (success) {
            Toast.makeText(this, "Data sent successfully", Toast.LENGTH_SHORT).show()
        } else {
            Toast.makeText(this, "Data send failed", Toast.LENGTH_SHORT).show()
        }
    }
    .onFailure { throwable ->
        Log.e("SendData", "Send error", throwable)
    }
```

#### Subscribe to Continuous Data Stream

**Java Usage Example:**

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
        Log.e("DataStream", "Data stream error", error);
        runOnUiThread(() -> {
            Toast.makeText(this, "Data stream error: " + error.getMessage(), Toast.LENGTH_LONG).show();
        });
    }
);

// Close subscription when done
dataSubscription.close();
```

**Kotlin Usage Example:**

```kotlin
// Subscribe to continuous data reception
val dataFlow = ConnectManager.receiveDataFlow()
dataFlow.collect { receivedData ->
    if (receivedData.isNotEmpty()) {
        val message = String(receivedData, StandardCharsets.UTF_8)
        // Process received data
        runOnUiThread {
            // Update UI with received data
            textView.append("Received: $message\n")
        }
    }
}
```

### BLE Device Information

**Java Usage Example:**

```java
// Get BLE device information (BLE mode only)
ConnectManager.INSTANCE.getBleDeviceInfoJava()
    .thenAccept(deviceInfo -> {
        if (deviceInfo != null) {
            // Process device information
            String info = "BLE Device Info: " + deviceInfo.toString();
            Toast.makeText(this, "Device info retrieved successfully", Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(this, "No device info available", Toast.LENGTH_SHORT).show();
        }
    });
```

**Kotlin Usage Example:**

```kotlin
// Get BLE device information (BLE mode only)
ConnectManager.getBleDeviceInfo()?.let { deviceInfo ->
    // Process device information
    val info = "BLE Device Info: $deviceInfo"
    Toast.makeText(this, "Device info retrieved successfully", Toast.LENGTH_SHORT).show()
} ?: run {
    Toast.makeText(this, "No device info available", Toast.LENGTH_SHORT).show()
}
```

### BLE Characteristic Operations

BLE characteristic operations are important functions in BLE connections, used to configure device read/write characteristics and notification descriptors.

#### CharacteristicProperty Enum Class

The `CharacteristicProperty` enum class defines common BLE characteristic properties, providing type-safe property management.

```
CharacteristicProperty.READ              // Read property (0x02)
CharacteristicProperty.WRITE             // Write property (0x08)  
CharacteristicProperty.NOTIFY            // Notify property (0x10)
CharacteristicProperty.INDICATE          // Indicate property (0x20)
CharacteristicProperty.WRITE_WITHOUT_RESPONSE // Write without response property (0x04)
```

**Utility Methods:**

```java
// Java usage example
// Get enum list from property value
List<CharacteristicProperty> properties = CharacteristicProperty.fromPropertyValue(characteristic.getProperties());

// Get enum from display name
CharacteristicProperty property = CharacteristicProperty.fromDisplayName("WRITE");

// Get display name
String displayName = CharacteristicProperty.WRITE.getDisplayName(); // "WRITE"

// Convert to display name list
List<String> names = CharacteristicProperty.toDisplayNames(properties);
```

```kotlin
// Kotlin usage example
// Get enum list from property value
val properties = CharacteristicProperty.fromPropertyValue(characteristic.properties)

// Get enum from display name
val property = CharacteristicProperty.fromDisplayName("WRITE")

// Get display name
val displayName = CharacteristicProperty.WRITE.displayName // "WRITE"

// Convert to display name list
val names = CharacteristicProperty.toDisplayNames(properties)
```

#### Modify BLE Write Characteristic

Used to configure BLE device write characteristics, controlling data transmission functionality.

**Java Usage Example:**

```java
// Modify BLE write characteristic (BLE mode only)
String characteristicUuid = "0000FFE1-0000-1000-8000-00805F9B34FB"; // Characteristic UUID
String typeName = CharacteristicProperty.WRITE.getDisplayName(); // Type name
boolean enable = true; // Enable or disable

ConnectManager.INSTANCE.onChangeBleWriteInfoJava(characteristicUuid, typeName, enable)
    .thenAccept(unused -> {
        Toast.makeText(this, "BLE write characteristic modified successfully", Toast.LENGTH_SHORT).show();
    })
    .exceptionally(throwable -> {
        Log.e("BLE", "Failed to modify write characteristic", throwable);
        runOnUiThread(() -> {
            Toast.makeText(this, "Modification failed: " + throwable.getMessage(), Toast.LENGTH_LONG).show();
        });
        return null;
    });
```

**Kotlin Usage Example:**

```kotlin
// Modify BLE write characteristic (BLE mode only)
val characteristicUuid = "0000FFE1-0000-1000-8000-00805F9B34FB" // Characteristic UUID
val typeName = CharacteristicProperty.WRITE.displayName // Type name
val enable = true // Enable or disable

ConnectManager.onChangeBleWriteInfo(characteristicUuid, typeName, enable)
// Note: Kotlin version calls directly, no return value
```

#### Modify BLE Descriptor

Used to configure BLE device notification descriptors, controlling data reception functionality.

**Java Usage Example:**

```java
// Modify BLE descriptor (BLE mode only)
String descriptorUuid = "00002902-0000-1000-8000-00805F9B34FB"; // Descriptor UUID
String typeName = CharacteristicProperty.NOTIFY.getDisplayName(); // Type name
boolean enable = true; // Enable or disable

ConnectManager.INSTANCE.onChangeBleDescriptorInfoJava(descriptorUuid, typeName, enable)
    .thenAccept(unused -> {
        Toast.makeText(this, "BLE descriptor modified successfully", Toast.LENGTH_SHORT).show();
    })
    .exceptionally(throwable -> {
        Log.e("BLE", "Failed to modify descriptor", throwable);
        runOnUiThread(() -> {
            Toast.makeText(this, "Modification failed: " + throwable.getMessage(), Toast.LENGTH_LONG).show();
        });
        return null;
    });
```

**Kotlin Usage Example:**

```kotlin
// Modify BLE descriptor (BLE mode only)
val descriptorUuid = "00002902-0000-1000-8000-00805F9B34FB" // Descriptor UUID
val typeName = CharacteristicProperty.NOTIFY.displayName // Type name
val enable = true // Enable or disable

ConnectManager.onChangeBleDescriptorInfo(descriptorUuid, typeName, enable)
// Note: Kotlin version calls directly, no return value
```

#### BLE Characteristic Operation Notes

1. **BLE mode only**: These methods are only effective in BLE connection mode, other connection types will be ignored
2. **UUID format**: Ensure correct UUID format
3. **Call timing**: Recommend configuring characteristics immediately after successful connection to ensure normal subsequent data communication
4. **Error handling**: Java version needs to handle possible exceptions, Kotlin version calls directly
5. **Type safety**: Recommend using `CharacteristicProperty` enum instead of hardcoded strings, providing type safety and IDE support

### Global Callbacks

**Java Usage Example:**

```java
// Set device disconnect callback
ConnectManager.INSTANCE.setOnDeviceDisconnect(() -> {
    runOnUiThread(() -> {
        Toast.makeText(this, "Device disconnected", Toast.LENGTH_SHORT).show();
        // Handle device disconnect logic
    });
});

// Set Bluetooth disconnect callback
ConnectManager.INSTANCE.setOnBluetoothDisconnect(() -> {
    runOnUiThread(() -> {
        Toast.makeText(this, "Bluetooth disconnected", Toast.LENGTH_SHORT).show();
        // Handle Bluetooth disconnect logic
    });
});

// Set RSSI update callback
ConnectManager.INSTANCE.setOnHandleRssiUpdate(rssi -> {
    runOnUiThread(() -> {
        // Update RSSI display
        textView.setText("Signal strength: " + rssi + " dBm");
    });
});
```

**Kotlin Usage Example:**

```kotlin
// Set device disconnect callback
ConnectManager.onDeviceDisconnect = {
    runOnUiThread {
        Toast.makeText(this, "Device disconnected", Toast.LENGTH_SHORT).show()
        // Handle device disconnect logic
    }
}

// Set Bluetooth disconnect callback
ConnectManager.onBluetoothDisconnect = {
    runOnUiThread {
        Toast.makeText(this, "Bluetooth disconnected", Toast.LENGTH_SHORT).show()
        // Handle Bluetooth disconnect logic
    }
}

// Set RSSI update callback
ConnectManager.onHandleRssiUpdate = { rssi ->
    runOnUiThread {
        // Update RSSI display
        textView.text = "Signal strength: $rssi dBm"
    }
}
```

## Best Practices

1. **Always close subscriptions** in `onDestroy()` or when no longer needed
2. **Use MAC addresses** instead of device names for connections
3. **Handle null values** from device information methods
4. **Add timeouts** to prevent hanging operations
5. **Test permission flows** on different Android versions
6. **Use proper error handling** for all async operations

## Support

For issues and questions, please refer to the library documentation or contact the development team.
