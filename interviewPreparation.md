## context:
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
        1. Configuration change happens for orientation change, locale language changes, keyboard availability, screen size changes
        2. When configuration changes, onDestroy is called for the  current activity. A new instance is created with onCreate.
        3. The state can be restored manually using SaveInstanceState in onCreate bundle
        4. @Override
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

Intents and Broadcast Receivers:

What is the difference between explicit and implicit intents?

How do you use BroadcastReceiver for system events (e.g., Bluetooth state changes)?

Services:

What are the types of Android Services?

How do foreground services differ, especially in context of Bluetooth scanning?

Permissions:

Which permissions are required for Bluetooth, Wi-Fi Direct, or nearby device access?

How do you handle runtime permission requests?

WorkManager/JobScheduler:

Which is suitable for periodic background Bluetooth sync tasks?

Threads and Async Tasks:

Alternatives to AsyncTask in modern Android (e.g., Kotlin Coroutines).

Why is multithreading important in Bluetooth/D2D comms?

Foreground Service Notification:

Why is a foreground notification mandatory for Bluetooth scanning in Android 10+?

Data Storage:

How to store scanned device data locally?

What’s the best approach to store sensor or telemetry data?

Dependency Injection:

How do you inject BluetoothAdapter using Dagger/Hilt?

Jetpack Components:

How can LiveData help in observing device state changes?

Core Android + Bluetooth/D2D Fundamentals (15 Questions)
1.

What are the primary Bluetooth profiles supported in Android?

How do Bluetooth Classic and Bluetooth Low Energy (BLE) differ in Android?

What permissions are required for Bluetooth operations in Android 12+?

What is the purpose of BluetoothAdapter in Android?

How do you check if Bluetooth is enabled on the device?

What is the role of BluetoothManager?

What is GATT in BLE, and how does Android handle it?

What is the difference between GATT server and GATT client?

How do you scan for BLE devices in Android?

What is the ScanCallback used for in BLE?

How can Android apps connect to paired Bluetooth Classic devices?

What are the implications of running Bluetooth operations in the background on Android 10+?

How do you manage Bluetooth connection states?

Explain bonding vs pairing in Bluetooth.

How do you handle multiple BLE connections simultaneously?

🟩 Permissions, Security & Privacy (10 Questions)
What are the implications of ACCESS_FINE_LOCATION in Bluetooth scanning?

How has Bluetooth permission handling changed from Android 11 to Android 12/13?

What runtime permissions are mandatory for BLE advertising?

How does Android handle MAC address randomization for BLE scanning?

How do you ensure secure communication over Bluetooth?

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
