## Bound service
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


## Worker manager 
``` kotlin 
    class BleScanWorker(
        context: Context,
        workerParams: WorkerParameters
    ) : CoroutineWorker(context, workerParams) {
    
        override suspend fun doWork(): Result {
            val scanner = BluetoothAdapter.getDefaultAdapter()?.bluetoothLeScanner
            if (scanner == null) return Result.failure()
    
            val scanFilter = ScanFilter.Builder().build()
            val scanSettings = ScanSettings.Builder()
                .setScanMode(ScanSettings.SCAN_MODE_LOW_POWER)
                .build()
    
            val callback = object : ScanCallback() {
                override fun onScanResult(callbackType: Int, result: ScanResult?) {
                    result?.let {
                        Log.d("BLE", "Device found: ${it.device.address}")
                    }
                }
    
                override fun onScanFailed(errorCode: Int) {
                    Log.e("BLE", "Scan failed with error: $errorCode")
                }
            }
    
            scanner.startScan(listOf(scanFilter), scanSettings, callback)
    
            delay(5000) // Scan for 5 seconds
    
            scanner.stopScan(callback)
    
            return Result.success()
        }
    }

val bleWorkRequest = PeriodicWorkRequestBuilder<BleScanWorker>(
    15, TimeUnit.MINUTES
).build()

WorkManager.getInstance(context).enqueueUniquePeriodicWork(
    "BleScanWork",
    ExistingPeriodicWorkPolicy.KEEP,
    bleWorkRequest
)

```

## jobscheduler
``` kotlin
class BleJobService : JobService() {

    private var isJobCancelled = false
    private var scanCallback: ScanCallback? = null

    override fun onStartJob(params: JobParameters?): Boolean {
        val scanner = BluetoothAdapter.getDefaultAdapter()?.bluetoothLeScanner ?: return false

        val filter = ScanFilter.Builder().build()
        val settings = ScanSettings.Builder()
            .setScanMode(ScanSettings.SCAN_MODE_LOW_POWER)
            .build()

        scanCallback = object : ScanCallback() {
            override fun onScanResult(callbackType: Int, result: ScanResult) {
                Log.d("BLE", "Device found: ${result.device.name}")
            }

            override fun onScanFailed(errorCode: Int) {
                Log.e("BLE", "Scan failed: $errorCode")
            }
        }

        scanner.startScan(listOf(filter), settings, scanCallback)

        // Stop scan after 5 seconds
        Handler(Looper.getMainLooper()).postDelayed({
            scanner.stopScan(scanCallback)
            jobFinished(params, false)
        }, 5000)

        return true
    }

    override fun onStopJob(params: JobParameters?): Boolean {
        BluetoothAdapter.getDefaultAdapter()?.bluetoothLeScanner?.stopScan(scanCallback)
        isJobCancelled = true
        return true // Reschedule if stopped prematurely
    }
}

val jobScheduler = getSystemService(Context.JOB_SCHEDULER_SERVICE) as JobScheduler

val component = ComponentName(this, BleJobService::class.java)
val jobInfo = JobInfo.Builder(1001, component)
    .setRequiredNetworkType(JobInfo.NETWORK_TYPE_ANY)
    .setRequiresCharging(false)
    .setRequiresDeviceIdle(false)
    .setPersisted(true) // Persist after reboot
    .setPeriodic(15 * 60 * 1000) // 15 minutes
    .build()

jobScheduler.schedule(jobInfo)

```

## live data 
``` java
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
```
## scan ble devices 

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

```

## gatt connection 

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

## gatt server code:
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

## find service via uuid 
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

```

### for ble 
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

### for ble with byte array
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
```

## batch scanning 
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

```

## ble advertisement as server 

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

```

## Bluetooth 

### server side code to broadcast the data
``` java
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

```

### client side communication 

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


```
### normal broadcast receive
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


```







