/## context:
### AOSP low level android frameworks
### GATT and GAP protocol
### Scanning and bonding
### SPP profile
### socket Communication
### Nearby API
### NFC
### Bluetooth background mode 
#### Doze Mode
#### Scanfilter
### Bluetooth permission 
### power efficiency: AlarmManager Vs JobScheduler VS WorkManager

### other important questions
1. Android Application Lifecycle:
     1. Explain the activity and fragment lifecycle.
          1. Activity is the main content that the user can interact with
          2. It has several life cycle callbacks
               1. onCreate: UI initialise
               2. onStart: visible to the user but not foreground
               3. onResume: The user can interact
               4. onPause: another activity is on foreground, save ui changes 
               5. onStop: resources released
               6. onRestart : 
               7. onDestroy: cleaning up resources 
          3. A fragment is a modular piece of an activity's UI whose lifecycle is tied to the host activity.
          4. It has lifecycle callbacks:
               1. onAttach: The fragment attaches to the host activity
               2. onCreate: initialise fragment UI state
               3. onCreateView: inflate the view
               4. onViewCreated: view created, now the listener setup can be done
               5. onStart: fragment is visible
               6. onResume: The user can interact
               7. onPause: fragment no longer foreground
               8. onStop: no longer visible
               9. onDestroyView: Clean up UI reference
               10. onDestroy: fragment instance destroy
               11. onDetach: fragment detaches from activity  
     2. How do you manage configuration changes?
        1. Configuration change happens for orientation change, locale language changes, keyboard availability, and screen size changes
        2. When configuration changes, onDestroy is called for the  current activity. A new instance is created with onCreate.
        3. The state can be restored manually using the SaveInstanceState in the onCreate bundle
        4.
     ``` java
          @Override
          protected void onSaveInstanceState(Bundle outState) {
              outState.putString("inputText", editText.getText().toString());
              super.onSaveInstanceState(outState);
           }
          @Override
          protected void onCreate(Bundle savedInstanceState) {
              super.onCreate(savedInstanceState);
              if (savedInstanceState != null) {
                  String text = savedInstanceState.getString("inputText");
                  editText.setText(text);
              }
          }
2. Intents and Broadcast Receivers:
   1. What is the difference between explicit and implicit intents?
      1. To navigate within the app activity, services are started where the target service or activity is specified. That's an explicit intent.
      2. In implicit intent, instead of specifying a specific activity, a general action to perform is declared, and Android finds the app that can handle it.
   2. How do you use BroadcastReceiver for system events (e.g., Bluetooth state changes)?
      ``` xml
      <uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
      <receiver android:name=".BluetoothStateReceiver">
              <intent-filter>
                  <action android:name="android.bluetooth.adapter.action.STATE_CHANGED" />
              </intent-filter>
      </receiver>
3. Services:
   1. What are the types of Android Services?
      1. Foreground service:
         1. The service will keep running under memory pressure.
         2. eg: music player, fitness tracker, download/upload
         3. Call startForeground() within 5 seconds of starting.
      2. Background service:
         1. Run without any user interaction or notification, but the app needs to be in foreground or using JobIntentService, WorkManager
         2. Available after Android 8+
         3. The tasks should be short  
      3. Bound Service
         1. Allows components to bind and interact via an interface
         2. The service will run as long as a client is bound to it
         
   2. Service lifecycle
      normal service
      1. onCreate:
         When the service is created first.
      2. onStartCommand:
         called whenever startService is called. The return type will decide how the next start service will use the intent.
         START_STICKY: Restart service with null intent.
         START_NOT_STICKY: Don’t restart.
         START_REDELIVER_INTENT: Restart and redeliver the last intent.
      3. onDestroy:
      
      bound service
      1. onBind(Intent intent): Called when a client binds using bindService(). Returns an IBinder for interaction.
      2. onUnbind(Intent intent): Called when all clients have unbound.
      3. onRebind(Intent intent): Called when new clients bind after onUnbind().
      4. onDestroy(): Called when the service is no longer used.

   3. How do foreground services differ, especially in the context of Bluetooth scanning?
      1. If the app is in the background and tries to use Bluetooth scanning, the service will be killed silently, but if it runs with a foreground service (like a notification), it will not. It will be executed.
``` java
         public class BluetoothScanService extends Service {
              @Override
              public int onStartCommand(Intent intent, int flags, int startId) {
                  startForeground(1, buildNotification());
                  startBleScan();
                  return START_STICKY;
              }
          
              private void startBleScan() {
                  BluetoothLeScanner scanner = BluetoothAdapter.getDefaultAdapter().getBluetoothLeScanner();
                  scanner.startScan(scanCallback);
              }
          
              private Notification buildNotification() {
                  return new NotificationCompat.Builder(this, "scan_channel")
                          .setContentTitle("Scanning for devices")
                          .setSmallIcon(R.drawable.ic_bluetooth)
                          .setOngoing(true)
                          .build();
              }
          
              @Override
              public IBinder onBind(Intent intent) {
                  return null;
              }
          }
```
4. Permissions:
   1. Which permissions are required for Bluetooth, Wi-Fi Direct, or nearby device access?
      1. Bluetooth:
      Android 12+
      - BLUETOOTH_CONNECT (for connect/bond/pair)
      - BLUETOOTH_SCAN (for scanning)
      - ACCESS_FINE_LOCATION (still needed for location-based scan filtering on some devices)
      2. Wi-Fi Direct:
      - ACCESS_FINE_LOCATION (for discovering peers)
      - CHANGE_WIFI_STATE
      - ACCESS_WIFI_STATE
      3. Nearby connections
      - NEARBY_WIFI_DEVICES (for nearby over Wi-Fi)
      - NEARBY_DEVICES (for BLE, Wi-Fi, or UWB)
      - BLUETOOTH_SCAN and ACCESS_FINE_LOCATION may still apply, depending on the implementation
   2. How do you handle runtime permission requests?
      ``` java
      if (hasAllPermissions(this, permissions)) {
              startBluetoothScan();
          } else {
              ActivityCompat.requestPermissions(this, permissions, BLUETOOTH_REQUEST_CODE);
          }
5. WorkManager/JobScheduler:
   1. Which is suitable for periodic background Bluetooth sync tasks?
      WorkManager is suitable for periodic background work
      reasons:
      life cycle aware
      runs even if the app crashes
      long-running task without load
   3. Threads and Async Tasks:
      1. Thread is a java concept
      ``` java
         Thread thread = new Thread(new Runnable() {
             @Override
             public void run() {
                 // Background task
             }
         });
         thread.start();
      ```
      3. Async tasks has been deprecated
         ``` java
         private class MyAsyncTask extends AsyncTask<Void, Integer, String> {

              @Override
              protected void onPreExecute() {
                  // Runs on UI thread before task starts
              }
          
              @Override
              protected String doInBackground(Void... params) {
                  // Runs on background thread
                  return "Result";
              }
          
              @Override
              protected void onProgressUpdate(Integer... values) {
                  // UI updates during progress
              }
          
              @Override
              protected void onPostExecute(String result) {
                  // Runs on UI thread with result
              }
          }

6. Alternatives to AsyncTask in modern Android (e.g., Kotlin Coroutines).
     1. Why is multithreading important in Bluetooth/D2D comms?
        In a background thread, after receiving any broadcast, we can run a long task on the same thread, as we can lose some responses
        So, a worker handler with a coroutine is the best way to manage this. 

7. Foreground Service Notification:
   1. Why is a foreground notification mandatory for Bluetooth scanning in Android 10+?
      Battery optimisation was introduced by Google in 10+ with strictness on privacy policy.
      

8. Data Storage:
   1. How to store scanned device data locally?
      using live data observer,
      each data point should have a time field to handle heavy flow
      
   3. What’s the best approach to store sensor or telemetry data?
      Speed and memory efficiency (since sensors can generate a lot of data),
      Persistence (to avoid data loss)
      Battery-awareness
      Upload/sync capability (to cloud or server)

      Use Handler, Coroutine, or ExecutorService to offload writes

9. Dependency Injection:
   1. How do you inject BluetoothAdapter using Dagger/Hilt?
``` kotlin
     @Module
     @InstallIn(SingletonComponent::class)
     object BluetoothModule {
     
         @Provides
         @Singleton
         fun provideBluetoothAdapter(): BluetoothAdapter? {
             val bluetoothManager = 
                 ApplicationProvider.getApplicationContext<Context>()
                     .getSystemService(Context.BLUETOOTH_SERVICE) as BluetoothManager
             return bluetoothManager.adapter
         }


     @AndroidEntryPoint
     class MainActivity : AppCompatActivity() {
     
         @Inject
         lateinit var bluetoothAdapter: BluetoothAdapter
     
         override fun onCreate(savedInstanceState: Bundle?) {
             super.onCreate(savedInstanceState)
     
             if (bluetoothAdapter.isEnabled) {
                 // Use the adapter
             }
         }
     }
   
```
11. Jetpack Components:
    1. How can LiveData help in observing device state changes?
    ``` kotlin
        class BluetoothStateLiveData(context: Context) : LiveData<Boolean>() {

              private val appContext = context.applicationContext
              private val bluetoothReceiver = object : BroadcastReceiver() {
                  override fun onReceive(context: Context?, intent: Intent?) {
                      if (BluetoothAdapter.ACTION_STATE_CHANGED == intent?.action) {
                          val state = intent.getIntExtra(BluetoothAdapter.EXTRA_STATE, -1)
                          value = state == BluetoothAdapter.STATE_ON
                      }
                  }
              }
          
              override fun onActive() {
                  super.onActive()
                  val filter = IntentFilter(BluetoothAdapter.ACTION_STATE_CHANGED)
                  appContext.registerReceiver(bluetoothReceiver, filter)
                  // Emit current state
                  val adapter = BluetoothAdapter.getDefaultAdapter()
                  value = adapter?.isEnabled == true
              }
          
              override fun onInactive() {
                  super.onInactive()
                  appContext.unregisterReceiver(bluetoothReceiver)
              }
          }

### Core Android + Bluetooth/D2D Fundamentals (15 Questions)
1. What are the primary Bluetooth profiles supported in Android?
   | Profile                                      | Description                                                             | Android API
   MEDIA PROFILES                         
   1. A2DP (Advanced Audio Distribution Profile)    | Streams high-quality audio from device to Bluetooth headphones/speakers | [`BluetoothA2dp`](https://developer.android.com/reference/android/bluetooth/BluetoothA2dp) |
   2. AVRCP (Audio/Video Remote Control Profile)    | Controls media playback (play, pause, skip) from headset/car controls   | Integrated with system, no direct API                                                 
   3. HFP (Hands-Free Profile)                      | Used for voice calls in cars/headsets                                   | System-managed, no direct app-level API

Input Device Profiles
   4. HID (Human Interface Device)                  | Supports input devices like keyboards, mice, game controllers           | `BluetoothHidDevice`, `BluetoothHidHost`
   5. HOGP (HID over GATT)                          | BLE-based HID devices (e.g., fitness trackers with button input)        | Limited support via GATT

Telephony & Messaging Profiles
   6. PBAP (Phone Book Access Profile)              | Used in cars to access contacts                                         | System-level only
   7. MAP (Message Access Profile)                  | Access and push SMS/MMS between phone and car                           | System-level only 

Communication & Networking Profiles
   8. SPP (Serial Port Profile)                     | Emulates serial communication over Bluetooth (classic)                  | `BluetoothSocket`, `BluetoothServerSocket` 
   9. PAN (Personal Area Network)                   | Internet sharing (tethering over Bluetooth)                             | Limited support; no public API   

Low Energy (BLE) Profiles
   10. GATT (Generic Attribute Profile)	             BLE-based data communication, services, characteristics	                  BluetoothGatt, BluetoothGattServer, BluetoothGattCallback
   11. TIP (Time Profile)	                       Sync time from smartphone to BLE device	                                 Via GATT, no dedicated Android API
   12. HRP (Heart Rate Profile)	                  Used for fitness devices (e.g., chest straps, smartwatches)                Via GATT with standardized UUIDs

Obsolete or Limited Support Profiles

   13. OPP (Object Push Profile)                    | Push files (e.g., images, contacts) between devices                     | Deprecated in modern Android versions   |
   14. FTP (File Transfer Profile)                  | Browsing remote file systems                                            | Not supported in standard Android stack |

2. How do Bluetooth Classic and Bluetooth Low Energy (BLE) differ in Android?
   1. Comparison
     | Feature                      | **Bluetooth Classic**             | **Bluetooth Low Energy (BLE)**                  |
     | ---------------------------- | --------------------------------- | ----------------------------------------------- |
     | **API Start**                | Android API 1                     | Android 4.3+ (API 18)                           |
     | **Use Case**                 | Streaming, continuous connections | Sensor data, infrequent small data transfers    |
     | **Power Consumption**        | Higher                            | Ultra-low power                                 |
     | **Connection Time**          | Slower (100–200 ms)               | Faster (3–10 ms)                                |
     | **Data Throughput**          | Higher (\~2–3 Mbps)               | Lower (\~0.27 Mbps)                             |
     | **Device Types**             | Headsets, speakers, printers      | Beacons, wearables, heart rate monitors         |
     | **Communication Mode**       | Continuous stream                 | GATT-based request-response                     |
     | **Simultaneous Connections** | Few (1:1 or limited multi-point)  | Many-to-one (central with multiple peripherals) |
     | **Pairing Requirement**      | Often required                    | Optional (can operate in advertising mode)      |
   2. Android API Differences
     | Operation         | Bluetooth Classic                                             | BLE                                |
     | ----------------- | ------------------------------------------------------------- | ---------------------------------- |
     | **Discovery**     | `startDiscovery()` (scan all nearby)                          | `startScan()` (fine-tuned filters) |
     | **Connect**       | `BluetoothSocket.connect()`                                   | `connectGatt()`                    |
     | **Data Transfer** | Streams via `InputStream`/`OutputStream`                      | Read/write GATT characteristics    |
     | **Permissions**   | `BLUETOOTH`, `BLUETOOTH_ADMIN`, `BLUETOOTH_CONNECT` (API 31+) | Same + location for scanning       |
     | **Pairing**       | Often via system UI                                           | GATT pairing optional              |

   3.  Code-level Differences
       Classic example
       ``` java
       BluetoothDevice device = bluetoothAdapter.getRemoteDevice(address);
       BluetoothSocket socket = device.createRfcommSocketToServiceRecord(MY_UUID);
       socket.connect();
       ```

       BLE Example
       ``` java
       device.connectGatt(context, false, new BluetoothGattCallback() {
           @Override
           public void onConnectionStateChange(...) { ... }
     
           @Override
           public void onServicesDiscovered(...) { ... }
       });

3. What permissions are required for Bluetooth operations in Android 12+?
   nearby device permission, location permission, and bluetooth permission 
4. What is the purpose of BluetoothAdapter in Android?
   1. The BluetoothAdapter in Android is the entry point for all Bluetooth interactions on the device. It represents the device’s Bluetooth radio and provides APIs to manage and control Bluetooth operations.
   2. It’s typically a singleton and is accessed using BluetoothAdapter.getDefaultAdapter().
5. How do you check if Bluetooth is enabled on the device?
   bluetoothAdapter.isEnabled()
   Intent enableBtIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
   startActivityForResult(enableBtIntent, REQUEST_ENABLE_BT);
6. What is the role of BluetoothManager?
   1. Multiple active GATT connections
   2. Connection state changes (connected/disconnected)
   3. Resource cleanup on disconnect

7. What is GATT in BLE, and how does Android handle it?
   1. Scan for BLE devices (BluetoothLeScanner)
   2. Connect using connectGatt()
   3. Discover services
   4. Read/Write characteristics or descriptors
   5. Enable notifications or indications
   6. Disconnect and close
  
   important
   GATT operations are asynchronous – you must wait for callbacks.
   Only one operation (read/write/discover) should be active at a time.
   BLE scanning and connection require runtime permissions (e.g., BLUETOOTH_CONNECT, NEARBY_DEVICES).
``` kotlin
   class MainActivity : AppCompatActivity() {

         private lateinit var bluetoothAdapter: BluetoothAdapter
         private lateinit var scanner: BluetoothLeScanner
         private var bluetoothGatt: BluetoothGatt? = null
     
         private val targetDeviceName = "MyBLEDevice" // Replace with your BLE device name
         private val targetServiceUUID = UUID.fromString("0000180d-0000-1000-8000-00805f9b34fb") // e.g., Heart Rate
         private val targetCharUUID = UUID.fromString("00002a37-0000-1000-8000-00805f9b34fb")    // e.g., HR Measurement
     
         override fun onCreate(savedInstanceState: Bundle?) {
             super.onCreate(savedInstanceState)
     
             val bluetoothManager = getSystemService(Context.BLUETOOTH_SERVICE) as BluetoothManager
             bluetoothAdapter = bluetoothManager.adapter
             scanner = bluetoothAdapter.bluetoothLeScanner
     
             checkPermissionsAndStartScan()
         }
     
         private fun checkPermissionsAndStartScan() {
             val perms = arrayOf(
                 Manifest.permission.BLUETOOTH_SCAN,
                 Manifest.permission.BLUETOOTH_CONNECT,
                 Manifest.permission.ACCESS_FINE_LOCATION
             )
     
             if (perms.any { ContextCompat.checkSelfPermission(this, it) != PackageManager.PERMISSION_GRANTED }) {
                 ActivityCompat.requestPermissions(this, perms, 1001)
             } else {
                 startScan()
             }
         }
     
         private fun startScan() {
             scanner.startScan(scanCallback)
             Toast.makeText(this, "Scanning...", Toast.LENGTH_SHORT).show()
         }
     
         private val scanCallback = object : ScanCallback() {
             override fun onScanResult(callbackType: Int, result: ScanResult) {
                 val device = result.device
                 if (device.name == targetDeviceName) {
                     scanner.stopScan(this)
                     bluetoothGatt = device.connectGatt(this@MainActivity, false, gattCallback)
                     Log.d("BLE", "Connecting to ${device.address}")
                 }
             }
         }
     
         private val gattCallback = object : BluetoothGattCallback() {
     
             override fun onConnectionStateChange(gatt: BluetoothGatt, status: Int, newState: Int) {
                 if (newState == BluetoothProfile.STATE_CONNECTED) {
                     Log.d("BLE", "Connected! Discovering services...")
                     gatt.discoverServices()
                 } else if (newState == BluetoothProfile.STATE_DISCONNECTED) {
                     Log.d("BLE", "Disconnected")
                     bluetoothGatt?.close()
                     bluetoothGatt = null
                 }
             }
     
             override fun onServicesDiscovered(gatt: BluetoothGatt, status: Int) {
                 val service = gatt.getService(targetServiceUUID)
                 val characteristic = service?.getCharacteristic(targetCharUUID)
     
                 if (characteristic != null) {
                     gatt.readCharacteristic(characteristic)
                 } else {
                     Log.e("BLE", "Characteristic not found")
                 }
             }
     
             override fun onCharacteristicRead(gatt: BluetoothGatt, characteristic: BluetoothGattCharacteristic, status: Int) {
                 if (status == BluetoothGatt.GATT_SUCCESS) {
                     val data = characteristic.value
                     Log.d("BLE", "Characteristic Read: ${data?.joinToString()}")
                 } else {
                     Log.e("BLE", "Failed to read characteristic")
                 }
             }
         }
     
         override fun onDestroy() {
             bluetoothGatt?.close()
             super.onDestroy()
         }
     }

```
8. What is the difference between GATT server and GATT client?
   GATT client: The device that requests data
   GATT server: The device that provides data

   gatt server code:
   ``` kotlin 
   class GattServerService : Service() {

         private lateinit var bluetoothManager: BluetoothManager
         private lateinit var bluetoothAdapter: BluetoothAdapter
         private var gattServer: BluetoothGattServer? = null
     
         private val SERVICE_UUID = UUID.fromString("00001810-0000-1000-8000-00805f9b34fb")
         private val CHAR_UUID = UUID.fromString("00002a35-0000-1000-8000-00805f9b34fb")
     
         override fun onCreate() {
             super.onCreate()
     
             bluetoothManager = getSystemService(Context.BLUETOOTH_SERVICE) as BluetoothManager
             bluetoothAdapter = bluetoothManager.adapter
     
             startGattServer()
         }
     
         private fun startGattServer() {
             val service = BluetoothGattService(SERVICE_UUID, BluetoothGattService.SERVICE_TYPE_PRIMARY)
     
             val characteristic = BluetoothGattCharacteristic(
                 CHAR_UUID,
                 BluetoothGattCharacteristic.PROPERTY_READ or BluetoothGattCharacteristic.PROPERTY_NOTIFY,
                 BluetoothGattCharacteristic.PERMISSION_READ
             )
     
             service.addCharacteristic(characteristic)
     
             gattServer = bluetoothManager.openGattServer(this, gattServerCallback)
             gattServer?.addService(service)
         }
     
         private val gattServerCallback = object : BluetoothGattServerCallback() {
     
             override fun onConnectionStateChange(device: BluetoothDevice?, status: Int, newState: Int) {
                 Log.d("GATT_SERVER", "Device connection state changed: $newState")
             }
     
             override fun onCharacteristicReadRequest(
                 device: BluetoothDevice, requestId: Int,
                 offset: Int, characteristic: BluetoothGattCharacteristic
             ) {
                 if (characteristic.uuid == CHAR_UUID) {
                     val responseValue = byteArrayOf(0x42) // some dummy data
                     gattServer?.sendResponse(device, requestId, BluetoothGatt.GATT_SUCCESS, 0, responseValue)
                     Log.d("GATT_SERVER", "Read request served")
                 }
             }
         }
     
         override fun onDestroy() {
             gattServer?.close()
             super.onDestroy()
         }
     
         override fun onBind(intent: Intent?): IBinder? = null
     }

Start BLE advertisement
``` kotlin
     val advertiser = bluetoothAdapter.bluetoothLeAdvertiser
     val settings = AdvertiseSettings.Builder()
         .setAdvertiseMode(AdvertiseSettings.ADVERTISE_MODE_LOW_LATENCY)
         .setConnectable(true)
         .build()
     
     val data = AdvertiseData.Builder()
         .setIncludeDeviceName(true)
         .addServiceUuid(ParcelUuid(SERVICE_UUID))
         .build()
     
     advertiser.startAdvertising(settings, data, advertiseCallback)
```
9. How do you scan for BLE devices in Android? What is the ScanCallback used for in BLE?
    ``` kotlin
   val scanner = bluetoothAdapter.bluetoothLeScanner
   val scanCallback = object : ScanCallback() {
        override fun onScanResult(callbackType: Int, result: ScanResult) {
             val device = result.device
             Log.d("BLE", "Found device: ${device.name}, address: ${device.address}")
         }
     
         override fun onScanFailed(errorCode: Int) {
             Log.e("BLE", "Scan failed with error: $errorCode")
         }
     }
   val filters = listOf(
         ScanFilter.Builder()
             .setDeviceName("MyBLEDevice") // or .setServiceUuid(ParcelUuid(UUID))
             .build()
     )

     val settings = ScanSettings.Builder()
         .setScanMode(ScanSettings.SCAN_MODE_LOW_LATENCY)
         .build()
     
     scanner.startScan(filters, settings, scanCallback)

11. Explain bonding vs pairing in Bluetooth.
    Pairing: Establish a trusted connection for the first time.
    Bonding: Save the trusted relationship for future connections. 
12. How do you handle multiple BLE connections simultaneously?
    BLE peripherals (servers) can usually only connect to one client.
    Android phones acting as clients can connect to ~4–7 BLE devices (varies by hardware).
     ``` kotlin 
    class MultiBleActivity : AppCompatActivity() {

         private lateinit var bluetoothAdapter: BluetoothAdapter
         private lateinit var bleScanner: BluetoothLeScanner
         private val gattMap = mutableMapOf<BluetoothDevice, BluetoothGatt>()
         private val scanResults = mutableSetOf<BluetoothDevice>()
     
         override fun onCreate(savedInstanceState: Bundle?) {
             super.onCreate(savedInstanceState)
     
             val manager = getSystemService(Context.BLUETOOTH_SERVICE) as BluetoothManager
             bluetoothAdapter = manager.adapter
             bleScanner = bluetoothAdapter.bluetoothLeScanner
     
             startScan()
         }
     
         private fun startScan() {
             val scanCallback = object : ScanCallback() {
                 override fun onScanResult(callbackType: Int, result: ScanResult) {
                     val device = result.device
                     if (device.name != null && scanResults.add(device)) {
                         Log.d("BLE_SCAN", "Discovered: ${device.name} - ${device.address}")
                         connectToDevice(device)
                     }
                 }
     
                 override fun onScanFailed(errorCode: Int) {
                     Log.e("BLE_SCAN", "Scan failed: $errorCode")
                 }
             }
     
             bleScanner.startScan(scanCallback)
     
             Handler(Looper.getMainLooper()).postDelayed({
                 bleScanner.stopScan(scanCallback)
                 Log.d("BLE_SCAN", "Scan stopped")
             }, 10000) // Stop after 10 seconds
         }
     
         private fun connectToDevice(device: BluetoothDevice) {
             Log.d("BLE_CONNECT", "Connecting to ${device.address}")
             val gatt = device.connectGatt(this, false, gattCallback)
             gattMap[device] = gatt
         }
     
         private val gattCallback = object : BluetoothGattCallback() {
     
             override fun onConnectionStateChange(gatt: BluetoothGatt, status: Int, newState: Int) {
                 val device = gatt.device
                 if (newState == BluetoothProfile.STATE_CONNECTED) {
                     Log.d("BLE_GATT", "Connected to ${device.address}")
                     gatt.discoverServices()
                 } else if (newState == BluetoothProfile.STATE_DISCONNECTED) {
                     Log.d("BLE_GATT", "Disconnected from ${device.address}")
                     gatt.close()
                     gattMap.remove(device)
                 }
             }
     
             override fun onServicesDiscovered(gatt: BluetoothGatt, status: Int) {
                 val device = gatt.device
                 val services = gatt.services
                 Log.d("BLE_GATT", "Services for ${device.address}:")
                 for (service in services) {
                     Log.d("BLE_GATT", " - Service: ${service.uuid}")
                 }
             }
         }
     
         override fun onDestroy() {
             super.onDestroy()
             gattMap.values.forEach { it.close() }
             gattMap.clear()
         }
     }

🟩 Permissions, Security & Privacy (10 Questions)
1. What are the implications of ACCESS_FINE_LOCATION in Bluetooth scanning?
   Starting from Android 6.0 (API level 23):
   1. Bluetooth scanning (especially BLE scanning) can reveal the user’s location indirectly, because many BLE beacons are deployed in fixed physical places (like shops, malls, etc.).
   2. Therefore, Android treats BLE scanning as location-sensitive, and requires location permissions to proceed.
   3. Even with permission granted, if location services (GPS) are turned off, BLE scanning may return nothing.

2. How has Bluetooth permission handling changed from Android 11 to Android 12/13?
   1. Android 11 has
      1. BLUETOOTH, ACCESS_FINE_LOCATION : Ble scan
      2. BLUETOOTH, BLUETOOTH_ADMIN : Connect to device
      3. BLUETOOTH_ADMIN : Advertise (be discoverable)
      4. ACCESS_FINE_LOCATION : always needed
   2. android 12+ has
      1. BLUETOOTH_SCAN (+ optional ACCESS_FINE_LOCATION) : Ble scan
      2. BLUETOOTH_CONNECT : connect device
      3. BLUETOOTH_ADVERTISE : Advertise (be discoverable)
      4. ACCESS_FINE_LOCATION : can be avoided with android:usesPermissionFlags="neverForLocation"

3. How does Android handle MAC address randomization for BLE scanning?
   1. Advertisers (BLE peripherals) can also randomize their MAC using non-resolvable or resolvable private addresses.
   2. If the advertiser is a paired (bonded) device, Android may use the static address for identification.
   3. You cannot rely on MAC addresses for BLE device identification. Instead, use device name, service UUIDs, or manufacturer data.
   4. Don't store MAC addresses to reconnect — use BluetoothDevice.getAddress(), but understand it might be randomized and change over time.
   5. Bonded (paired) devices: Android may use the real MAC address.

4. How do you ensure secure communication over Bluetooth?
   1. for transmission using secure socket with Serial port Profile (SPP) (UUID) secures the communication
   ```
     BluetoothSocket socket = device.createRfcommSocketToServiceRecord(MY_UUID);
   ```
   2. Authenticated pairing with encryption and Authenticated LE Secure Connections can secure ble connection.
   3. Even with BLE encryption, consider end-to-end application-layer encryption, such as:
      1. AES/GCM
         ```
         fun encryptMessage(msg: String, secretKey: SecretKeySpec): ByteArray {
              val cipher = Cipher.getInstance("AES/GCM/NoPadding")
              cipher.init(Cipher.ENCRYPT_MODE, secretKey)
              return cipher.doFinal(msg.toByteArray())
          }
      3. Public-key cryptography
      4. HMAC for message integrity
         HMAC uses a hash function (e.g., SHA-256) + a secret key.
         It ensures:
         ✅ Message hasn't been tampered with
         ✅ Message came from a trusted source (with the key)
         ``` kotlin 
         import javax.crypto.Mac
         import javax.crypto.spec.SecretKeySpec
          
          fun generateHmacSHA256(message: String, secret: String): String {
              val keySpec = SecretKeySpec(secret.toByteArray(), "HmacSHA256")
              val mac = Mac.getInstance("HmacSHA256")
              mac.init(keySpec)
              val hmacBytes = mac.doFinal(message.toByteArray())
              return hmacBytes.joinToString("") { "%02x".format(it) }
          }
         
          val message = "temperature=26.5"
          val secretKey = "my_ble_secret_key_123"
          
          val hmac = generateHmacSHA256(message, secretKey)
          println("HMAC = $hmac")

5. What role does BluetoothDevice.getUuids() play in secure discovery?
   1. Returns an array of ParcelUuid objects.
   2. Each UUID represents a Bluetooth service that the device supports (e.g., A2DP, HFP, custom SPP services).
   3. Helps the initiating device understand what functionality the remote device provides.
       ``` java
      ParcelUuid[] uuids = bluetoothDevice.getUuids();
   4. for BLE users
       ``` java
      bluetoothGatt.discoverServices();
      BluetoothGattService.getUuid().
   5. demo code
      ``` java
          IntentFilter filter1 = new IntentFilter(BluetoothDevice.ACTION_FOUND);
          IntentFilter filter2 = new IntentFilter(BluetoothDevice.ACTION_UUID);
          registerReceiver(receiver, filter1);
          registerReceiver(receiver, filter2);

          BluetoothAdapter bluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
          if (bluetoothAdapter != null && bluetoothAdapter.isEnabled()) {
              bluetoothAdapter.startDiscovery();
          }

           private final BroadcastReceiver receiver = new BroadcastReceiver() {
              public void onReceive(Context context, Intent intent) {
                  String action = intent.getAction();
                  if (BluetoothDevice.ACTION_FOUND.equals(action)) {
                      BluetoothDevice device = intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);
                      if (device != null) {
                          Log.d("Device Found", device.getName() + " - " + device.getAddress());
          
                          // Optional: Wait for UUIDs to be fetched
                          device.fetchUuidsWithSdp(); // Triggers ACTION_UUID
                      }
                  } else if (BluetoothDevice.ACTION_UUID.equals(action)) {
                      BluetoothDevice device = intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);
                      Parcelable[] uuidExtra = intent.getParcelableArrayExtra(BluetoothDevice.EXTRA_UUID);
                      if (uuidExtra != null) {
                          for (Parcelable p : uuidExtra) {
                              ParcelUuid uuid = (ParcelUuid) p;
                              if (uuid.getUuid().toString().equals("00001101-0000-1000-8000-00805F9B34FB")) {
                                  Log.d("Target Device", "Found matching UUID device: " + device.getName());
                                  // You can now initiate secure connection
                              }
                          }
                      }
                  }
              }
          };
   6. for ble
       ``` java
          BluetoothLeScanner scanner = BluetoothAdapter.getDefaultAdapter().getBluetoothLeScanner();
          UUID targetUuid = UUID.fromString("0000180D-0000-1000-8000-00805F9B34FB"); // Heart Rate
          ScanFilter filter = new ScanFilter.Builder()
                  .setServiceUuid(new ParcelUuid(targetUuid))
                  .build();
          
          List<ScanFilter> filters = new ArrayList<>();
          filters.add(filter);
          
          ScanSettings settings = new ScanSettings.Builder()
                  .setScanMode(ScanSettings.SCAN_MODE_LOW_LATENCY)
                  .build();
          
          scanner.startScan(filters, settings, scanCallback);

           private final ScanCallback scanCallback = new ScanCallback() {
              @Override
              public void onScanResult(int callbackType, ScanResult result) {
                  BluetoothDevice device = result.getDevice();
                  Log.d("BLE Device", "Found device: " + device.getName());
          
                  // Optional: Connect and verify services
                  connectToDevice(device);
              }
          };

           private void connectToDevice(BluetoothDevice device) {
              device.connectGatt(context, false, gattCallback);
          }
          
          private final BluetoothGattCallback gattCallback = new BluetoothGattCallback() {
              @Override
              public void onConnectionStateChange(BluetoothGatt gatt, int status, int newState) {
                  if (newState == BluetoothProfile.STATE_CONNECTED) {
                      gatt.discoverServices();
                  }
              }
          
              @Override
              public void onServicesDiscovered(BluetoothGatt gatt, int status) {
                  for (BluetoothGattService service : gatt.getServices()) {
                      if (service.getUuid().equals(UUID.fromString("0000180D-0000-1000-8000-00805F9B34FB"))) {
                          Log.d("Service Found", "Heart Rate service available!");
                          // Now you can interact with the characteristics
                      }
                  }
              }
          };
   7. for ble with byte array
       ``` java
       ScanCallback scanCallback = new ScanCallback() {
              @Override
              public void onScanResult(int callbackType, ScanResult result) {
                  ScanRecord scanRecord = result.getScanRecord();
                  if (scanRecord != null) {
                      byte[] rawBytes = scanRecord.getBytes();  // ← This is your raw advertisement packet
          
                      // Example: print in hex
                      StringBuilder sb = new StringBuilder();
                      for (byte b : rawBytes) {
                          sb.append(String.format("%02X ", b));
                      }
                      Log.d("BLE-ADV", "Advertisement Bytes: " + sb.toString());
                  }
              }
          };
     
6. What changes were introduced in Android 10 related to background location access?
   If the application is running in the background, the app should request ACCESS_BACKGROUND_LOCATION.

7. How do you revoke Bluetooth permissions once granted?
   If Bluetooth is turned off, and I want to open the settings page for the user
   I can send an intent to the settings page and launch it
   ``` java
     Intent intent = new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
     intent.setData(Uri.parse("package:" + context.getPackageName()));
     context.startActivity(intent);

🟨 BLE Advertising & Scanning (10 Questions)
1. How do you implement BLE advertising in Android?
   ``` kotlin
     val advertiser = BluetoothAdapter.getDefaultAdapter().bluetoothLeAdvertiser
     val settings = AdvertiseSettings.Builder()
         .setAdvertiseMode(AdvertiseSettings.ADVERTISE_MODE_LOW_LATENCY)
         .setTxPowerLevel(AdvertiseSettings.ADVERTISE_TX_POWER_HIGH)
         .setConnectable(false)
         .build()
     
     val data = AdvertiseData.Builder()
         .setIncludeDeviceName(true)
         .addServiceUuid(ParcelUuid(UUID.fromString("0000180D-0000-1000-8000-00805f9b34fb"))) // Heart Rate UUID
         .addManufacturerData(0x1234, byteArrayOf(0x01, 0x02)) // Optional custom data
         .build()
     
     advertiser.startAdvertising(settings, data, object : AdvertiseCallback() {
         override fun onStartSuccess(settingsInEffect: AdvertiseSettings) {
             Log.d("BLE", "Advertising started")
         }
     
         override fun onStartFailure(errorCode: Int) {
             Log.e("BLE", "Advertising failed: $errorCode")
         }
     })

        val notification = NotificationCompat.Builder(this, "ble_channel")
         .setContentTitle("BLE Advertising")
         .setSmallIcon(R.drawable.ic_ble)
         .build()
     
          startForegroundService(Intent(this, BleAdvertiserService::class.java))

2. What is AdvertiseSettings, and how does it affect battery consumption?
   following parameter can affect power decipitation
   1. setAdvertiseMode(): Sets the frequency of advertisement packets
   2. setTxPowerLevel(): Sets transmission power (i.e., how far the signal can reach)
   3. setConnectable(): Whether your device allows connections
   4. setTimeout(): Duration of advertising

3. What are the limitations of BLE advertising payload size?
   1. Each packet size can be max upto 31 bytes
   2. there is 2 segments of pay load
      1. Advertisement packet: which is sent initially in each advertisement (31 bytes)
      2. Scan Response packet: which is sent upon request (31 bytes)
      ``` kotlin
          val advertiseData = AdvertiseData.Builder()
              .setIncludeDeviceName(true)
              .addServiceUuid(ParcelUuid(YOUR_UUID))
              .build()
          
          val scanResponse = AdvertiseData.Builder()
              .addManufacturerData(0x1234, byteArrayOf(0x01, 0x02))
              .build()
     3. Android’s BLE stack handles:
        1. Receiving initial advertisement
        2. Sending scan request (if peripheral allows it)
        3. Receiving scan response
        4. Merging both into ScanResult

4. How can you avoid conflicts with multiple advertisers?
   1. Some devices allow 2-3 advertisers, not more than that
   2. It's better to check available advertisers before advertising
``` java 
      BluetoothLeAdvertiser advertiser = BluetoothAdapter.getDefaultAdapter().getBluetoothLeAdvertiser();
      if (!BluetoothAdapter.getDefaultAdapter().isMultipleAdvertisementSupported()) {
         Log.e("BLE", "Multiple advertisement not supported");
      }
```
   4. Before starting, we need to stop the old advertiser
      advertiser.stopAdvertising(advertiseCallback);
   5. 

5. What is the use of ScanFilter and ScanSettings?
   ``` kotlin
   private fun startBleScan() {
         val serviceUuid = ParcelUuid.fromString("0000180D-0000-1000-8000-00805F9B34FB") // Heart Rate UUID
     
         // 1. Set ScanFilter for specific service UUID
         val filters = listOf(
             ScanFilter.Builder()
                 .setServiceUuid(serviceUuid)
                 .build()
         )
     
         // 2. Configure ScanSettings with:
         // - SCAN_MODE_LOW_POWER (or BALANCED/LOW_LATENCY)
         // - Report results every 5 seconds
         // - Callback when first match is found
         // - Up to 3 matches reported per filter
         val settings = ScanSettings.Builder()
             .setScanMode(ScanSettings.SCAN_MODE_LOW_POWER)          // Use LOW_POWER for battery saving
             .setReportDelay(5000)                                   // 5 seconds delay, batching results
             .setCallbackType(ScanSettings.CALLBACK_TYPE_FIRST_MATCH) // Callback when first match is found
             .setNumOfMatches(ScanSettings.MATCH_NUM_FEW_ADVERTISEMENT) // Up to 3 matches per filter
             .build()
     
         bluetoothLeScanner.startScan(filters, settings, scanCallback)
     }
   ```
   1. setReportDelay(5000): Results will be batched and delivered every 5 seconds.
   2. setCallbackType(CALLBACK_TYPE_FIRST_MATCH): Fires callback only for first match (reduces redundant processing).
   3. setNumOfMatches(MATCH_NUM_FEW_ADVERTISEMENT): System reports up to 3 matches per filter.

6. How do you detect and filter nearby BLE beacons?
   ``` java
   new ScanFilter.Builder()
    .setManufacturerData(manufacturerId, manufacturerData, manufacturerDataMask)
    .build();
7. What strategies can optimize scanning frequency to save battery?
   1. ``` java
        new Handler().postDelayed(() -> {
         scanner.stopScan(callback);
          }, 10000); 
   2. ``` java
      new ScanSettings.Builder()
         .setDeviceAddress("XX:XX:XX:XX:XX:XX") 
         .setScanMode(ScanSettings.SCAN_MODE_LOW_POWER)
         .setReportDelay(5000) // Deliver results every 5 seconds
         .build();
   3. example of using batch scanning
      ``` java
      public class BleScannerActivity extends AppCompatActivity {

              private BluetoothLeScanner bleScanner;
              private ScanCallback scanCallback;
          
              @Override
              protected void onCreate(Bundle savedInstanceState) {
                  super.onCreate(savedInstanceState);
          
                  BluetoothAdapter bluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
                  if (bluetoothAdapter != null && bluetoothAdapter.isEnabled()) {
                      bleScanner = bluetoothAdapter.getBluetoothLeScanner();
                      startBatchScan();
                  } else {
                      Toast.makeText(this, "Bluetooth not available or enabled", Toast.LENGTH_SHORT).show();
                  }
              }
          
              private void startBatchScan() {
                  // Configure Scan Settings
                  ScanSettings settings = new ScanSettings.Builder()
                          .setScanMode(ScanSettings.SCAN_MODE_LOW_POWER)
                          .setReportDelay(5000) // Report results every 5 seconds
                          .build();
          
                  // Optionally configure filters
                  List<ScanFilter> filters = new ArrayList<>();
                  // e.g., ScanFilter for specific UUID/device
          
                  // Define ScanCallback
                  scanCallback = new ScanCallback() {
                      @Override
                      public void onBatchScanResults(List<ScanResult> results) {
                          for (ScanResult result : results) {
                              BluetoothDevice device = result.getDevice();
                              int rssi = result.getRssi();
                              Log.d("BLE", "Device: " + device.getAddress() + ", RSSI: " + rssi);
                              // You can also access advertisement data: result.getScanRecord()
                          }
                      }
          
                      @Override
                      public void onScanFailed(int errorCode) {
                          Log.e("BLE", "Scan failed with error: " + errorCode);
                      }
                  };
          
                  bleScanner.startScan(filters, settings, scanCallback);
          
                  // Optional: Stop scan after a timeout
                  new Handler(Looper.getMainLooper()).postDelayed(() -> stopScan(), 15000);
              }
          
              private void stopScan() {
                  if (bleScanner != null && scanCallback != null) {
                      bleScanner.stopScan(scanCallback);
                      Log.d("BLE", "Scan stopped");
                  }
              }
          }


🟥 Data Transmission & GATT (15 Questions)
1. What is a BLE service and characteristic?
     ``` java
     private final UUID BATTERY_SERVICE_UUID = UUID.fromString("0000180F-0000-1000-8000-00805f9b34fb");
     private final UUID BATTERY_LEVEL_UUID = UUID.fromString("00002A19-0000-1000-8000-00805f9b34fb");
     
     private BluetoothGatt bluetoothGatt;
     
     // Connect to GATT server
     private final BluetoothGattCallback gattCallback = new BluetoothGattCallback() {
         @Override
         public void onConnectionStateChange(BluetoothGatt gatt, int status, int newState) {
             if (newState == BluetoothProfile.STATE_CONNECTED) {
                 gatt.discoverServices();
             }
         }
     
         @Override
         public void onServicesDiscovered(BluetoothGatt gatt, int status) {
             BluetoothGattService batteryService = gatt.getService(BATTERY_SERVICE_UUID);
             if (batteryService != null) {
                 BluetoothGattCharacteristic batteryLevelChar = batteryService.getCharacteristic(BATTERY_LEVEL_UUID);
     
                 // Read battery level once
                 gatt.readCharacteristic(batteryLevelChar);
     
                 // Enable notifications
                 gatt.setCharacteristicNotification(batteryLevelChar, true);
     
                 BluetoothGattDescriptor descriptor = batteryLevelChar.getDescriptor(
                         UUID.fromString("00002902-0000-1000-8000-00805f9b34fb")); // Client Characteristic Configuration
                 if (descriptor != null) {
                     descriptor.setValue(BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE);
                     gatt.writeDescriptor(descriptor);
                 }
             }
         }
     
         @Override
         public void onCharacteristicRead(BluetoothGatt gatt, BluetoothGattCharacteristic characteristic, int status) {
             if (BATTERY_LEVEL_UUID.equals(characteristic.getUuid())) {
                 int batteryLevel = characteristic.getIntValue(BluetoothGattCharacteristic.FORMAT_UINT8, 0);
                 Log.d("BLE", "Battery Level: " + batteryLevel + "%");
             }
         }
     
         @Override
         public void onCharacteristicChanged(BluetoothGatt gatt, BluetoothGattCharacteristic characteristic) {
             if (BATTERY_LEVEL_UUID.equals(characteristic.getUuid())) {
                 int batteryLevel = characteristic.getIntValue(BluetoothGattCharacteristic.FORMAT_UINT8, 0);
                 Log.d("BLE", "Battery Level (notified): " + batteryLevel + "%");
             }
         }
     };
     ```
     1. through gatt connection service is discovered first
     2. through service read or write access is asked and used 


2. How do you handle notifications and indications in BLE?
   notification or indicator is a feature in which notification will come to the service subscriber and in indiaction the client who is service surscriber have to return a confirmation
   ``` java
     private static final UUID BATTERY_SERVICE_UUID = UUID.fromString("0000180F-0000-1000-8000-00805f9b34fb");
     private static final UUID BATTERY_LEVEL_UUID = UUID.fromString("00002A19-0000-1000-8000-00805f9b34fb");
     private static final UUID CCCD_UUID = UUID.fromString("00002902-0000-1000-8000-00805f9b34fb");
     
     @Override
     public void onServicesDiscovered(BluetoothGatt gatt, int status) {
         BluetoothGattService service = gatt.getService(BATTERY_SERVICE_UUID);
         BluetoothGattCharacteristic characteristic = service.getCharacteristic(BATTERY_LEVEL_UUID);
     
         // Step 1: Enable local notifications
         gatt.setCharacteristicNotification(characteristic, true);
     
         // Step 2: Write to CCCD to enable notifications or indications
         BluetoothGattDescriptor descriptor = characteristic.getDescriptor(CCCD_UUID);
         descriptor.setValue(BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE); // or ENABLE_INDICATION_VALUE
         gatt.writeDescriptor(descriptor);
     }
      @Override
     public void onCharacteristicChanged(BluetoothGatt gatt, BluetoothGattCharacteristic characteristic) {
         if (BATTERY_LEVEL_UUID.equals(characteristic.getUuid())) {
             int level = characteristic.getIntValue(BluetoothGattCharacteristic.FORMAT_UINT8, 0);
             Log.d("BLE", "Battery Level changed: " + level + "%");
         }
     }


3. How do you implement a GATT server on Android?
   1. it need to have 4 component
      1. BluetoothGattServer
      2. BluetoothGattService
      3. BluetoothGattCharacteristic
      4. BluetoothGattDescriptor
   ``` java
     BluetoothManager bluetoothManager = (BluetoothManager) getSystemService(Context.BLUETOOTH_SERVICE);
     BluetoothAdapter bluetoothAdapter = bluetoothManager.getAdapter();
     BluetoothGattServer gattServer = bluetoothManager.openGattServer(context, gattServerCallback);
     UUID SERVICE_UUID = UUID.fromString("0000180F-0000-1000-8000-00805f9b34fb"); // Battery
     UUID CHAR_UUID = UUID.fromString("00002A19-0000-1000-8000-00805f9b34fb");  // Battery level
     
     BluetoothGattService service = new BluetoothGattService(SERVICE_UUID, BluetoothGattService.SERVICE_TYPE_PRIMARY);
     
     BluetoothGattCharacteristic batteryLevelChar = new BluetoothGattCharacteristic(
         CHAR_UUID,
         BluetoothGattCharacteristic.PROPERTY_READ | BluetoothGattCharacteristic.PROPERTY_NOTIFY,
         BluetoothGattCharacteristic.PERMISSION_READ
     );
     
     // Optional: Add CCCD descriptor
     UUID CCCD_UUID = UUID.fromString("00002902-0000-1000-8000-00805f9b34fb");
     BluetoothGattDescriptor cccd = new BluetoothGattDescriptor(CCCD_UUID, BluetoothGattDescriptor.PERMISSION_READ | BluetoothGattDescriptor.PERMISSION_WRITE);
     batteryLevelChar.addDescriptor(cccd);
     
     service.addCharacteristic(batteryLevelChar);
     gattServer.addService(service);

     BluetoothLeAdvertiser advertiser = bluetoothAdapter.getBluetoothLeAdvertiser();

     AdvertiseSettings settings = new AdvertiseSettings.Builder()
         .setAdvertiseMode(AdvertiseSettings.ADVERTISE_MODE_LOW_LATENCY)
         .setConnectable(true)
         .setTimeout(0)
         .build();
     
     AdvertiseData data = new AdvertiseData.Builder()
         .setIncludeDeviceName(true)
         .addServiceUuid(new ParcelUuid(SERVICE_UUID))
         .build();
     
     advertiser.startAdvertising(settings, data, advertiseCallback);

     BluetoothGattServerCallback gattServerCallback = new BluetoothGattServerCallback() {
         @Override
         public void onConnectionStateChange(BluetoothDevice device, int status, int newState) {
             Log.d("GATT", "Device connected: " + device.getAddress());
         }
     
         @Override
         public void onCharacteristicReadRequest(BluetoothDevice device, int requestId, int offset,
                                                 BluetoothGattCharacteristic characteristic) {
             if (CHAR_UUID.equals(characteristic.getUuid())) {
                 byte[] value = new byte[]{50}; // 50% battery
                 gattServer.sendResponse(device, requestId, BluetoothGatt.GATT_SUCCESS, offset, value);
             }
         }
     
         @Override
         public void onDescriptorWriteRequest(BluetoothDevice device, int requestId,
                                              BluetoothGattDescriptor descriptor,
                                              boolean preparedWrite, boolean responseNeeded,
                                              int offset, byte[] value) {
             // Enable notifications
             if (CCCD_UUID.equals(descriptor.getUuid())) {
                 descriptor.setValue(value);
                 if (responseNeeded) {
                     gattServer.sendResponse(device, requestId, BluetoothGatt.GATT_SUCCESS, offset, value);
                 }
             }
         }
     };

     batteryLevelChar.setValue(new byte[]{80}); // new battery level
     gattServer.notifyCharacteristicChanged(connectedDevice, batteryLevelChar, false);

4. What are the limits of BLE throughput on Android?
   1. | Factor                                | Value                                         |
   2. | ------------------------------------- | --------------------------------------------- |
   3. | Max ATT MTU size                      | 517 bytes (Android 5.0+ via request)          |
   4. | Max data per notification             | \~244 bytes (ATT\_MTU - 3)                    |
   5. | Connection interval (min)             | \~7.5 ms (ideal)                              |
   6. | Notifications per connection interval | 4–6 (theoretically)                           |
   7. | Theoretical Max Throughput            | \~1 Mbps (BLE 4.2 with Data Length Extension) |
   8. | Realistic Android Throughput          | **80–150 kbps** (varies by hardware + OS)     |

5. How do you handle MTU (Maximum Transmission Unit) size changes in BLE?
   1. MTU defines the maximum size (in bytes) of a single ATT (Attribute Protocol) packet.
   2. Default MTU is 23 bytes (20 bytes payload).
   3. Can be increased up to 517 bytes (since Android 5.0, API 21).
   4. Both client and server must agree on the negotiated MTU.
     ``` java
     bluetoothGatt.getMtu()
     bluetoothGatt.requestMtu(517);  // Request max supported MTU

     @Override
     public void onMtuChanged(BluetoothGatt gatt, int mtu, int status) {
         super.onMtuChanged(gatt, mtu, status);
         if (status == BluetoothGatt.GATT_SUCCESS) {
             Log.d("BLE", "MTU changed to: " + mtu);
             // Now you can send larger packets based on new MTU
         } else {
             Log.d("BLE", "MTU change failed");
         }
     }

     ```
6. How do you handle BLE connection priority?
   ``` java
   BluetoothGatt gatt = device.connectGatt(context, false, new BluetoothGattCallback() {
         @Override
         public void onConnectionStateChange(BluetoothGatt gatt, int status, int newState) {
             if (newState == BluetoothProfile.STATE_CONNECTED) {
                 Log.d("BLE", "Connected. Requesting high priority...");
                 gatt.requestConnectionPriority(BluetoothGatt.CONNECTION_PRIORITY_HIGH);
                 gatt.discoverServices(); // You can still discover services afterward
             }
         }
     });
   ```

   1. | Priority Type                   | Purpose                                 | Use Case Example                                      |
   2. | ------------------------------- | --------------------------------------- | ----------------------------------------------------- |
   3. | `CONNECTION_PRIORITY_HIGH`      | Low latency, high frequency updates     | Real-time apps (heart rate monitor, game controllers) |
   4. | `CONNECTION_PRIORITY_BALANCED`  | Default setting, balanced performance   | Most general use cases                                |
   5. | `CONNECTION_PRIORITY_LOW_POWER` | Infrequent communication, saves battery | Background sensors, data logging                      |

6. What causes GATT operations to fail intermittently?
   1. GATT can fail due to simultaneous GATT operations, solution: 
      1. Queue operations manually.
      2. Start the next GATT operation only after the previous one completes.
   2. poor signal, solution:
      1. Monitor RSSI (readRemoteRssi()).
      2. Reduce distance or interference.
   3. OS-Level Bluetooth Stack Issues
      1. Call BluetoothGatt.disconnect() and close() before reconnecting.
      2. Wait ~1 second before retrying a connection.
   4. MTU Negotiation Delays or Mismatch
      1. Use requestMtu() and handle onMtuChanged() before large transfers.
   5. BLE allows only one operation at a time. Failing to wait for a response causes drops or silent failures.
      ``` java
      int props = characteristic.getProperties();
      boolean canWrite = (props & BluetoothGattCharacteristic.PROPERTY_WRITE) > 0;

7. How do you debug BLE data transmission issues?
   Use Nordic’s nRF Connect to validate BLE operations.

8. How do you implement acknowledgment for BLE writes?
   peripheral device can send an acknowledgment via characteristics change
   ``` java

     BluetoothGattCharacteristic characteristic = ...;
     characteristic.setValue(data);
     characteristic.setWriteType(BluetoothGattCharacteristic.WRITE_TYPE_DEFAULT); // <-- ACK
     bluetoothGatt.writeCharacteristic(characteristic);
      
   @Override
     public void onCharacteristicChanged(BluetoothGatt gatt,
                                          BluetoothGattCharacteristic characteristic) {
         byte[] value = characteristic.getValue();
         if (Arrays.equals(value, "ACK".getBytes())) {
             Log.d("BLE", "ACK received from peripheral");
         }
     }


9. How can you queue GATT operations reliably?
   Queuing GATT operations is critical in Android BLE development because GATT operations must be executed sequentially — only one operation (read/write/descriptor request/etc.) can be in progress at a time. If you issue multiple calls back-to-back, many will fail silently or return GATT_BUSY.
   

10. What are the typical causes of BLE disconnections?
    can be caused by
    1. low battery
    2. interference
    3. out of range
    4. physical obstacle
    5. GATT over load
    6. ANR
    7. missing permission
    8. bluetooth stack bugs
    9. bluetooth restart

11. How can you ensure reliable large data transfer over BLE?
    1. checking the max mtu limit
    2. based on that splitting the data
    3. writing the data in small fragments
    4. the smaller the fragment lesser the battery drainage and higher the time for transmission. 

12. Can Android GATT client connect to multiple servers at once?
    1. 10+ server can be connected at once
    2. but on each server, read and write operations can be done sequentially, not simultaneously.
    3. Each connection has its own BluetoothGatt instance.

🟦 Bluetooth Classic (10 Questions)
1. How do you connect to Bluetooth Classic devices in Android?
   1. After discovering the Bluetooth device through broadcasting
      ``` java
      // server side code to broadcast the data
      
           BluetoothServerSocket serverSocket = bluetoothAdapter.listenUsingRfcommWithServiceRecord("MyApp", SPP_UUID);
           //listenUsingInsecureRfcommWithServiceRecord() — no pairing required.

      BluetoothSocket socket = serverSocket.accept();  // Blocking call
           InputStream in = socket.getInputStream();
          OutputStream out = socket.getOutputStream();
           outputStream.write("Hello Device".getBytes());
              // Read response (blocking)
              int data = inputStream.read();

      serverSocket.close();


      // client side code to get the data 
           private static final UUID SPP_UUID = UUID.fromString("00001101-0000-1000-8000-00805F9B34FB");
           Set<BluetoothDevice> pairedDevices = bluetoothAdapter.getBondedDevices();
          for (BluetoothDevice device: pairedDevices) {
              Log.d("BT", "Paired Device: " + device.getName() + ", " + device.getAddress());
          }

           BluetoothDevice device = bluetoothAdapter.getRemoteDevice("XX:XX:XX:XX:XX:XX"); // MAC Address
          BluetoothSocket socket = null;
          
          try {
              socket = device.createRfcommSocketToServiceRecord(SPP_UUID);
              socket.connect();  // Blocking call
              OutputStream outputStream = socket.getOutputStream();
              InputStream inputStream = socket.getInputStream();
          
              outputStream.write("Hello Device".getBytes());
              
              // Read response (blocking)
              int data = inputStream.read();
              
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              try {
                  if (socket != null) socket.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }


2. What is the role of BluetoothSocket and BluetoothServerSocket?
   bluetoothSocket act as server and BluetoothServerSocket act as client 

3. What are the limitations of Bluetooth Classic data rates?
   1. it has 3 rates
      1. basic rate: 721 knps
      2. Enhanced Data Rate 2: 1.3 Mbps
      3. Enhanced Data Rate 3: 2.1 Mbps
   2. Overhead and Latency
   3. Full-Duplex Constraints
   4. Cannot use Classic to broadcast or efficiently communicate with multiple devices simultaneously.
   5. Android limits simultaneous Classic + BLE scanning or connections.

4. How do you manage threading for BluetoothClassic communication?
   ``` kotlin
        class BluetoothClient(
         private val bluetoothDevice: BluetoothDevice,
         private val uuid: UUID
     ) {
         private var bluetoothSocket: BluetoothSocket? = null
         private var inputStream: InputStream? = null
         private var outputStream: OutputStream? = null
     
         private val coroutineScope = CoroutineScope(Dispatchers.IO + SupervisorJob())
     
         fun connect(onConnected: () -> Unit, onFailure: (Exception) -> Unit) {
             coroutineScope.launch {
                 try {
                     bluetoothSocket = bluetoothDevice.createRfcommSocketToServiceRecord(uuid)
                     bluetoothSocket?.connect()
     
                     inputStream = bluetoothSocket?.inputStream
                     outputStream = bluetoothSocket?.outputStream
     
                     withContext(Dispatchers.Main) { onConnected() }
     
                     listenForData()
                 } catch (e: Exception) {
                     withContext(Dispatchers.Main) { onFailure(e) }
                 }
             }
         }
     
         private suspend fun listenForData() {
             val buffer = ByteArray(1024)
             try {
                 while (true) {
                     val bytesRead = inputStream?.read(buffer) ?: break
                     if (bytesRead > 0) {
                         val received = buffer.copyOf(bytesRead)
                         // handle incoming data
                         Log.d("Bluetooth", "Received: ${received.decodeToString()}")
                     }
                 }
             } catch (e: IOException) {
                 Log.e("Bluetooth", "Read failed: ${e.message}")
             }
         }
     
         fun write(data: ByteArray) {
             coroutineScope.launch {
                 try {
                     outputStream?.write(data)
                 } catch (e: IOException) {
                     Log.e("Bluetooth", "Write failed: ${e.message}")
                 }
             }
         }
     
         fun disconnect() {
             coroutineScope.cancel()
             try {
                 inputStream?.close()
                 outputStream?.close()
                 bluetoothSocket?.close()
             } catch (e: IOException) {
                 Log.e("Bluetooth", "Disconnect failed: ${e.message}")
             }
         }
     }


     val client = BluetoothClient(device, MY_UUID)

     client.connect(
         onConnected = { Log.d("BT", "Connected!") },
         onFailure = { error -> Log.e("BT", "Failed: ${error.message}") }
     )
     
     // To send:
     client.write("Hello!".toByteArray())
     
     // To close:
     client.disconnect()

5. What is SDP, and how does Android use it?
   1. SDP (Service Discovery Protocol) is a key protocol in Bluetooth that allows devices to discover the services offered by other devices, along with the characteristics of those services.
   2. SDP is part of the Bluetooth Classic stack (not BLE), and it operates before a Bluetooth connection is fully established.
   3. When acting as a client, Android uses SDP implicitly when you call:
      ``` kotlin
          val device: BluetoothDevice = // already bonded device
          device.fetchUuidsWithSdp()
          
          val receiver = object : BroadcastReceiver() {
              override fun onReceive(context: Context?, intent: Intent?) {
                  val action = intent?.action
                  if (BluetoothDevice.ACTION_UUID == action) {
                      val uuids = intent.getParcelableArrayExtra(BluetoothDevice.EXTRA_UUID)
                      uuids?.forEach { uuid ->
                          Log.d("Bluetooth", "Discovered UUID: ${(uuid as ParcelUuid).uuid}")
                      }
                  }
              }
          }

6. How do you handle a failed Bluetooth Classic connection?
   1. Device not in range or turned off
   2. UUID mismatch or not exposed via SDP
   3. Device not paired or bonded
   4. Socket already in use
   5. Timeout or connection rejected
   6. Bluetooth permission or adapter issue
     ``` kotlin
     fun retryConnect(device: BluetoothDevice, uuid: UUID, retries: Int = 3): BluetoothSocket? {
         repeat(retries) { attempt ->
             val socket = safeConnect(device, uuid)
             if (socket != null) return socket
             Thread.sleep((attempt + 1) * 1000L)
         }
         return null
     }
7. What is RFCOMM, and how does it relate to Bluetooth Classic?
   RFCOMM (Radio Frequency Communication) is a protocol that emulates serial port communication over the Bluetooth Classic stack. It allows Bluetooth devices to communicate in a way similar to RS-232 serial cables, making it especially useful for legacy systems and serial-based peripherals like GPS modules, barcode scanners, or Arduino devices.

8. How do you perform Bluetooth discovery using startDiscovery()?
   ``` kotlin
   class DeviceDiscoveryReceiver : BroadcastReceiver() {
         override fun onReceive(context: Context, intent: Intent) {
             when (intent.action) {
                 BluetoothDevice.ACTION_FOUND -> {
                     val device: BluetoothDevice =
                         intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE)!!
                     Log.d("BT_DISCOVERY", "Found: ${device.name} - ${device.address}")
                 }
                 BluetoothAdapter.ACTION_DISCOVERY_FINISHED -> {
                     Log.d("BT_DISCOVERY", "Discovery finished")
                 }
             }
         }
     }
     
     // Usage in Activity or ViewModel
     val bluetoothAdapter: BluetoothAdapter? = BluetoothAdapter.getDefaultAdapter()
     
     // Register receiver
     val filter = IntentFilter().apply {
         addAction(BluetoothDevice.ACTION_FOUND)
         addAction(BluetoothAdapter.ACTION_DISCOVERY_FINISHED)
     }
     val receiver = DeviceDiscoveryReceiver()
     context.registerReceiver(receiver, filter)
     
     // Start discovery
     if (bluetoothAdapter?.isDiscovering == true) {
         bluetoothAdapter.cancelDiscovery()
     }
     bluetoothAdapter?.startDiscovery()

9. How can Classic and BLE be used in parallel?
   Yes — many Android devices support both Bluetooth Classic and BLE and can use both in parallel, but:
   1. Some devices may prioritize Classic over BLE or vice versa.
   2. Bluetooth hardware has a shared radio, so simultaneous usage is multiplexed, which may degrade performance.

🟧 Nearby, Wi-Fi Direct, and Other D2D Protocols (15 Questions)
What is the Nearby Connections API and how is it used?

How does Wi-Fi Direct differ from standard Wi-Fi communication?

What are the phases of Wi-Fi Direct connection in Android?

How do you use WifiP2pManager to discover peers?

How do you send files using Nearby or Wi-Fi Direct?

What limitations does Wi-Fi Direct have on Android?

How do you decide between BLE, Bluetooth Classic, Nearby, and Wi-Fi Direct?

What are the energy trade-offs of various D2D technologies?

How does device discovery work with Google's Nearby API?

How do you handle group formation in Wi-Fi Direct?

🟪 System Behavior, Lifecycle, and Power (10 Questions)
1. How does Doze mode affect Bluetooth operations?
   Doze mode, introduced in Android 6.0 (API 23), is designed to reduce battery usage by restricting background activity when the device is idle. However, it has specific implications for Bluetooth operations, particularly those running in the background.
What is the impact of battery optimization on BLE scanning?

How do you ensure reliable Bluetooth operations across configuration changes?

How do you handle reconnecting to devices after a reboot?

How can you make your BLE service resilient to process death?

How do you maintain Bluetooth connections in the background?

How do foreground services interact with Bluetooth APIs?

What is the best practice for starting Bluetooth services on boot?

What are common causes of Bluetooth-related ANRs?

How does airplane mode impact Bluetooth D2D features?

🟫 Architecture & Best Practices (10 Questions)
How would you architect an app that continuously syncs data over BLE?

How would you decouple your UI layer from Bluetooth communication logic?

How do you architect multi-device BLE communication?

How do you handle device compatibility issues across different manufacturers?

How do you write unit tests for Bluetooth-related code?

What are common pitfalls in Android BLE app architecture?

How would you handle BLE communication in a Jetpack Compose app?

How do you design for interoperability with non-Android BLE devices?

How do you structure your code to handle both Classic and BLE devices?

How can you improve UX for pairing and device discovery?

🟥 Debugging, Logs, and Tools (5 Questions)
What tools are available for sniffing BLE packets?

How do you use logcat effectively for debugging BLE issues?

How do you debug connection issues between Android and BLE peripherals?

How do you use nRF Connect for testing Android Bluetooth?

What developer settings can help with Bluetooth debugging on Android?

🔵 Scenario-Based & Behavioral (5 Questions)
Describe a time when you faced a challenge implementing BLE on Android. How did you overcome it?

How would you build a secure proximity-based unlocking system?

How would you debug inconsistent BLE performance across multiple Android devices?

What would you do if your BLE app worked on Pixel devices but failed on Samsung ones?

How would you design a device-to-device chat app using Bluetooth and Nearby?

### Development tips 
#### bluetooth system  
1. Prefer dynamic registration in modern apps due to Android background limitations.
2. Always unregister receivers to avoid memory leaks.
3. For long-running tasks in response to broadcasts, start a foreground service or use WorkManager.
