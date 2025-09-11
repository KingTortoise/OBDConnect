# ConnectManager 管理类 - Java & Kotlin 调用示例

ConnectManager 是一个用于管理蓝牙经典和BLE连接的核心管理类，提供友好的Java和Kotlin API，支持CompletableFuture和流式订阅。

## 核心特性

- **多协议支持**: 蓝牙经典和BLE连接
- **现代API**: 基于CompletableFuture的异步操作
- **流式订阅**: 实时数据流和响应式编程
- **权限管理**: 自动运行时权限处理
- **跨平台**: 同时支持Java和Kotlin
- **线程安全**: 所有操作都正确处理线程

## 包信息

- **包名**: `com.btBleTcp.connect`
- **核心类**: 
  - `ConnectManager` (Kotlin object; Java中通过 `ConnectManager.INSTANCE` 访问)
  - `BluetoothPermissionManager` (Kotlin类)
- **连接类型**: `ConnectType` 枚举 (`WIFI(0)`, `BT(1)`, `BLE(2)`)

## 集成方式

### 1. 添加库

将 `connect-debug.aar` 文件放在 `app/libs/` 目录下，并在 `app/build.gradle` 中添加：

```gradle
dependencies {
    implementation files('libs/connect-debug.aar')
}
```

## ConnectManager API 详解

### 初始化

#### Java 调用示例

```java
// 初始化连接管理器
ConnectManager.INSTANCE.initManagerJava(ConnectType.BLE.getValue(), getApplicationContext())
    .thenAccept(context -> {
        // 初始化成功
        // context 可能为 null - 这是正常的
        Log.d("ConnectManager", "初始化成功");
    })
    .exceptionally(throwable -> {
        // 处理初始化错误
        Log.e("ConnectManager", "初始化失败", throwable);
        return null;
    });
```

#### Kotlin 调用示例

```kotlin
// 初始化连接管理器
ConnectManager.initManager(ConnectType.BLE.value, applicationContext)
    .onSuccess { context ->
        // 初始化成功
        Log.d("ConnectManager", "初始化成功")
    }
    .onFailure { throwable ->
        // 处理初始化错误
        Log.e("ConnectManager", "初始化失败", throwable)
    }
```

### 设备发现

#### 开始扫描

**Java 调用示例:**

```java
// 开始设备发现
ConnectManager.INSTANCE.startScanJava()
    .thenAccept(unused -> {
        // 扫描开始成功
        Toast.makeText(this, "扫描已开始", Toast.LENGTH_SHORT).show();
    })
    .exceptionally(throwable -> {
        // 处理扫描开始错误
        Log.e("Scan", "扫描开始失败", throwable);
        return null;
    });
```

**Kotlin 调用示例:**

```kotlin
// 开始设备发现
ConnectManager.startScan()
    .onSuccess {
        // 扫描开始成功
        Toast.makeText(this, "扫描已开始", Toast.LENGTH_SHORT).show()
    }
    .onFailure { throwable ->
        // 处理扫描开始错误
        Log.e("Scan", "扫描开始失败", throwable)
    }
```

#### 订阅设备流

**Java 调用示例:**

```java
// 订阅连续设备更新
AutoCloseable deviceSubscription = ConnectManager.INSTANCE.getDeviceFlowJava(
    devices -> {
        // 使用发现的设备更新UI
        Set<BluetoothDevice> deviceSet = (Set<BluetoothDevice>) devices;
        for (BluetoothDevice device : deviceSet) {
            String deviceInfo = device.getName() + " (" + device.getAddress() + ")";
            // 添加到设备列表
        }
    },
    error -> {
        // 处理设备流错误
        Log.e("DeviceStream", "设备流错误", error);
    }
);

// 不要忘记关闭订阅
deviceSubscription.close();
```

**Kotlin 调用示例:**

```kotlin
// 订阅连续设备更新
val deviceFlow = ConnectManager.getDeviceFlow()
deviceFlow.collect { devices ->
    // 使用发现的设备更新UI
    devices.forEach { device ->
        val deviceInfo = "${device.name} (${device.address})"
        // 添加到设备列表
    }
}
```

### 连接管理

#### 连接设备

**Java 调用示例:**

```java
// 使用设备地址（推荐）或名称连接
String deviceAddress = "AA:BB:CC:DD:EE:FF"; // 推荐使用MAC地址
ConnectManager.INSTANCE.connectJava(deviceAddress, getApplicationContext())
    .thenAccept(success -> {
        if (Boolean.TRUE.equals(success)) {
            // 连接成功
            Toast.makeText(this, "连接成功", Toast.LENGTH_SHORT).show();
        } else {
            // 连接失败
            Toast.makeText(this, "连接失败", Toast.LENGTH_SHORT).show();
        }
    })
    .exceptionally(throwable -> {
        // 处理连接错误
        Log.e("Connection", "连接错误", throwable);
        return null;
    });
```

**Kotlin 调用示例:**

```kotlin
// 使用设备地址（推荐）或名称连接
val deviceAddress = "AA:BB:CC:DD:EE:FF" // 推荐使用MAC地址
ConnectManager.connect(deviceAddress, applicationContext)
    .onSuccess { success ->
        if (success) {
            // 连接成功
            Toast.makeText(this, "连接成功", Toast.LENGTH_SHORT).show()
        } else {
            // 连接失败
            Toast.makeText(this, "连接失败", Toast.LENGTH_SHORT).show()
        }
    }
    .onFailure { throwable ->
        // 处理连接错误
        Log.e("Connection", "连接错误", throwable)
    }
```

#### 重连

**Java 调用示例:**

```java
// 使用现有连接上下文重连
ConnectManager.INSTANCE.reconnectJava()
    .thenAccept(success -> {
        if (Boolean.TRUE.equals(success)) {
            Toast.makeText(this, "重连成功", Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(this, "重连失败", Toast.LENGTH_SHORT).show();
        }
    });
```

**Kotlin 调用示例:**

```kotlin
// 使用现有连接上下文重连
ConnectManager.reconnect()
    .onSuccess { success ->
        if (success) {
            Toast.makeText(this, "重连成功", Toast.LENGTH_SHORT).show()
        } else {
            Toast.makeText(this, "重连失败", Toast.LENGTH_SHORT).show()
        }
    }
    .onFailure { throwable ->
        Log.e("Reconnect", "重连错误", throwable)
    }
```

#### 关闭连接

**Java 调用示例:**

```java
// 关闭当前连接
ConnectManager.INSTANCE.closeJava()
    .thenAccept(unused  -> {
        Toast.makeText(this, "连接已关闭", Toast.LENGTH_SHORT).show();
    });
```

**Kotlin 调用示例:**

```kotlin
// 关闭当前连接
ConnectManager.close()
```

### 数据通信

#### 发送数据

**Java 调用示例:**

```java
// 发送数据（带超时）
String message = "Hello, Device!\r\n";
byte[] data = message.getBytes(StandardCharsets.UTF_8);

ConnectManager.INSTANCE.writeJava(data, 3000L) // 3秒超时
    .thenAccept(success -> {
        if (Boolean.TRUE.equals(success)) {
            Toast.makeText(this, "数据发送成功", Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(this, "数据发送失败", Toast.LENGTH_SHORT).show();
        }
    })
    .exceptionally(throwable -> {
        Log.e("SendData", "发送错误", throwable);
        return null;
    });
```

**Kotlin 调用示例:**

```kotlin
// 发送数据（带超时）
val message = "Hello, Device!\r\n"
val data = message.toByteArray(StandardCharsets.UTF_8)

ConnectManager.write(data, 3000L) // 3秒超时
    .onSuccess { success ->
        if (success) {
            Toast.makeText(this, "数据发送成功", Toast.LENGTH_SHORT).show()
        } else {
            Toast.makeText(this, "数据发送失败", Toast.LENGTH_SHORT).show()
        }
    }
    .onFailure { throwable ->
        Log.e("SendData", "发送错误", throwable)
    }
```

#### 订阅连续数据流

**Java 调用示例:**

```java
// 订阅连续数据接收
AutoCloseable dataSubscription = ConnectManager.INSTANCE.receiveDataFlowJava(
    receivedData -> {
        if (receivedData != null && receivedData.length > 0) {
            String message = new String(receivedData, StandardCharsets.UTF_8);
            // 处理接收到的数据
            runOnUiThread(() -> {
                // 使用接收到的数据更新UI
                textView.append("接收: " + message + "\n");
            });
        }
    },
    error -> {
        Log.e("DataStream", "数据流错误", error);
        runOnUiThread(() -> {
            Toast.makeText(this, "数据流错误: " + error.getMessage(), Toast.LENGTH_LONG).show();
        });
    }
);

// 完成时关闭订阅
dataSubscription.close();
```

**Kotlin 调用示例:**

```kotlin
// 订阅连续数据接收
val dataFlow = ConnectManager.receiveDataFlow()
dataFlow.collect { receivedData ->
    if (receivedData.isNotEmpty()) {
        val message = String(receivedData, StandardCharsets.UTF_8)
        // 处理接收到的数据
        runOnUiThread {
            // 使用接收到的数据更新UI
            textView.append("接收: $message\n")
        }
    }
}
```

### BLE设备信息

**Java 调用示例:**

```java
// 获取BLE设备信息（仅BLE模式）
ConnectManager.INSTANCE.getBleDeviceInfoJava()
    .thenAccept(deviceInfo -> {
        if (deviceInfo != null) {
            // 处理设备信息
            String info = "BLE设备信息: " + deviceInfo.toString();
            Toast.makeText(this, "设备信息获取成功", Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(this, "无设备信息可用", Toast.LENGTH_SHORT).show();
        }
    });
```

**Kotlin 调用示例:**

```kotlin
// 获取BLE设备信息（仅BLE模式）
ConnectManager.getBleDeviceInfo()?.let { deviceInfo ->
    // 处理设备信息
    val info = "BLE设备信息: $deviceInfo"
    Toast.makeText(this, "设备信息获取成功", Toast.LENGTH_SHORT).show()
} ?: run {
    Toast.makeText(this, "无设备信息可用", Toast.LENGTH_SHORT).show()
}
```
### BLE特征值操作

BLE特征值操作是BLE连接中的重要功能，用于配置设备的读写特征值和通知描述符。

#### CharacteristicProperty 枚举类

`CharacteristicProperty` 枚举类定义了常见的BLE特征属性，提供了类型安全的属性管理方式。

```
CharacteristicProperty.READ              // 读属性 (0x02)
CharacteristicProperty.WRITE             // 写属性 (0x08)  
CharacteristicProperty.NOTIFY            // 通知属性 (0x10)
CharacteristicProperty.INDICATE          // 指示属性 (0x20)
CharacteristicProperty.WRITE_WITHOUT_RESPONSE // 无响应写属性 (0x04)
```
**实用方法:**

```java
// Java 使用示例
// 根据属性值获取枚举列表
List<CharacteristicProperty> properties = CharacteristicProperty.fromPropertyValue(characteristic.getProperties());

// 根据显示名称获取枚举
CharacteristicProperty property = CharacteristicProperty.fromDisplayName("WRITE");

// 获取显示名称
String displayName = CharacteristicProperty.WRITE.getDisplayName(); // "WRITE"

// 转换为显示名称列表
List<String> names = CharacteristicProperty.toDisplayNames(properties);
```

```kotlin
// Kotlin 使用示例
// 根据属性值获取枚举列表
val properties = CharacteristicProperty.fromPropertyValue(characteristic.properties)

// 根据显示名称获取枚举
val property = CharacteristicProperty.fromDisplayName("WRITE")

// 获取显示名称
val displayName = CharacteristicProperty.WRITE.displayName // "WRITE"

// 转换为显示名称列表
val names = CharacteristicProperty.toDisplayNames(properties)
```

#### 修改BLE写入特征值

用于配置BLE设备的写入特征值，控制数据发送功能。

**Java 调用示例:**

```java
// 修改BLE写入特征值（仅BLE模式）
String characteristicUuid = "0000FFE1-0000-1000-8000-00805F9B34FB"; // 特征值UUID
String typeName = CharacteristicProperty.WRITE.getDisplayName(); // 类型名称
boolean enable = true; // 启用或禁用

ConnectManager.INSTANCE.onChangeBleWriteInfoJava(characteristicUuid, typeName, enable)
    .thenAccept(unused -> {
        Toast.makeText(this, "BLE写入特征值修改成功", Toast.LENGTH_SHORT).show();
    })
    .exceptionally(throwable -> {
        Log.e("BLE", "修改写入特征值失败", throwable);
        runOnUiThread(() -> {
            Toast.makeText(this, "修改失败: " + throwable.getMessage(), Toast.LENGTH_LONG).show();
        });
        return null;
    });
```

**Kotlin 调用示例:**

```kotlin
// 修改BLE写入特征值（仅BLE模式）
val characteristicUuid = "0000FFE1-0000-1000-8000-00805F9B34FB" // 特征值UUID
val typeName = CharacteristicProperty.WRITE.displayName // 类型名称
val enable = true // 启用或禁用

ConnectManager.onChangeBleWriteInfo(characteristicUuid, typeName, enable)
// 注意：Kotlin版本直接调用，没有返回值
```

#### 修改BLE描述符

用于配置BLE设备的通知描述符，控制数据接收功能。

**Java 调用示例:**

```java
// 修改BLE描述符（仅BLE模式）
String descriptorUuid = "00002902-0000-1000-8000-00805F9B34FB"; // 描述符UUID
String typeName = CharacteristicProperty.NOTIFY.getDisplayName(); // 类型名称
boolean enable = true; // 启用或禁用

ConnectManager.INSTANCE.onChangeBleDescriptorInfoJava(descriptorUuid, typeName, enable)
    .thenAccept(unused -> {
        Toast.makeText(this, "BLE描述符修改成功", Toast.LENGTH_SHORT).show();
    })
    .exceptionally(throwable -> {
        Log.e("BLE", "修改描述符失败", throwable);
        runOnUiThread(() -> {
            Toast.makeText(this, "修改失败: " + throwable.getMessage(), Toast.LENGTH_LONG).show();
        });
        return null;
    });
```

**Kotlin 调用示例:**

```kotlin
// 修改BLE描述符（仅BLE模式）
val descriptorUuid = "00002902-0000-1000-8000-00805F9B34FB" // 描述符UUID
val typeName = CharacteristicProperty.NOTIFY.displayName // 类型名称
val enable = true // 启用或禁用

ConnectManager.onChangeBleDescriptorInfo(descriptorUuid, typeName, enable)
// 注意：Kotlin版本直接调用，没有返回值
```

#### BLE特征值操作注意事项

1. **仅BLE模式有效**: 这些方法只在BLE连接模式下有效，其他连接类型调用会被忽略
2. **UUID格式**: 确保使用正确的UUID格式，通常是128位标准格式
3. **调用时机**: 建议在连接成功后立即配置特征值，确保后续数据通信正常
4. **错误处理**: Java版本需要处理可能的异常，Kotlin版本直接调用
5. **类型安全**: 推荐使用`CharacteristicProperty`枚举而不是硬编码字符串，提供类型安全和IDE支持

### 全局回调

**Java 调用示例:**

```java
// 设置设备断开连接回调
ConnectManager.INSTANCE.setOnDeviceDisconnect(() -> {
    runOnUiThread(() -> {
        Toast.makeText(this, "设备已断开连接", Toast.LENGTH_SHORT).show();
        // 处理设备断开逻辑
    });
});

// 设置蓝牙断开连接回调
ConnectManager.INSTANCE.setOnBluetoothDisconnect(() -> {
    runOnUiThread(() -> {
        Toast.makeText(this, "蓝牙已断开", Toast.LENGTH_SHORT).show();
        // 处理蓝牙断开逻辑
    });
});

// 设置RSSI更新回调
ConnectManager.INSTANCE.setOnHandleRssiUpdate(rssi -> {
    runOnUiThread(() -> {
        // 更新RSSI显示
        textView.setText("信号强度: " + rssi + " dBm");
    });
});
```

**Kotlin 调用示例:**

```kotlin
// 设置设备断开连接回调
ConnectManager.onDeviceDisconnect = {
    runOnUiThread {
        Toast.makeText(this, "设备已断开连接", Toast.LENGTH_SHORT).show()
        // 处理设备断开逻辑
    }
}

// 设置蓝牙断开连接回调
ConnectManager.onBluetoothDisconnect = {
    runOnUiThread {
        Toast.makeText(this, "蓝牙已断开", Toast.LENGTH_SHORT).show()
        // 处理蓝牙断开逻辑
    }
}

// 设置RSSI更新回调
ConnectManager.onHandleRssiUpdate = { rssi ->
    runOnUiThread {
        // 更新RSSI显示
        textView.text = "信号强度: $rssi dBm"
    }
}
```
## 最佳实践

1. **始终在 `onDestroy()` 中关闭订阅**或不再需要时
2. **使用MAC地址**而不是设备名称进行连接
3. **处理设备信息方法中的null值**
4. **添加超时**以防止挂起操作
5. **在不同Android版本上测试权限流程**
6. **对所有异步操作使用适当的错误处理**

## 支持

如有问题和疑问，请参考库文档或联系开发团队。
