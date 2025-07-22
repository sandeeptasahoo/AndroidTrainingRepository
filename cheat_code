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
