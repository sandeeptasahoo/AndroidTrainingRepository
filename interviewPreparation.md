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

What role does BluetoothDevice.getUuids() play in secure discovery?

What changes were introduced in Android 10 related to background location access?

How can you request Bluetooth and location permissions dynamically?

How do you revoke Bluetooth permissions once granted?

How would you explain Bluetooth security risks to a product manager?

🟨 BLE Advertising & Scanning (10 Questions)
How do you implement BLE advertising in Android?

What is AdvertiseSettings, and how does it affect battery consumption?

How do you configure AdvertiseData?

What are the limitations of BLE advertising payload size?

How do you detect when advertising has started successfully?

How do you manage advertising on devices with restricted background access?

How can you avoid conflicts with multiple advertisers?

What is the use of ScanFilter and ScanSettings?

How do you detect and filter nearby BLE beacons?

What strategies can optimize scanning frequency to save battery?

🟥 Data Transmission & GATT (15 Questions)
How do you read and write characteristics over BLE?

What is a BLE service and characteristic?

How do you handle notifications and indications in BLE?

How do you implement a GATT server on Android?

What are the limits of BLE throughput on Android?

How do you handle MTU size changes in BLE?

How do you handle BLE connection priority?

What is the role of BluetoothGattCallback?

What causes GATT operations to fail intermittently?

How do you debug BLE data transmission issues?

How do you implement acknowledgment for BLE writes?

How can you queue GATT operations reliably?

What are the typical causes of BLE disconnections?

How can you ensure reliable large data transfer over BLE?

Can Android GATT client connect to multiple servers at once?

🟦 Bluetooth Classic (10 Questions)
How do you connect to Bluetooth Classic devices in Android?

What is the role of BluetoothSocket and BluetoothServerSocket?

What are the limitations of Bluetooth Classic data rates?

How do you manage threading for BluetoothClassic communication?

What is SDP, and how does Android use it?

How do you handle a failed Bluetooth Classic connection?

What is RFCOMM, and how does it relate to Bluetooth Classic?

How do you perform Bluetooth discovery using startDiscovery()?

How can Classic and BLE be used in parallel?

What are the power implications of using Classic vs BLE?

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
How does Doze mode affect Bluetooth operations?

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
