NetworkEvents
===============================

Android library listening network events.

Currently supported events:
* ConnectivityStatus change:
 * WIFI_CONNECTED
 * WIFI_CONNECTED_HAS_INTERNET
 * WIFI_CONNECTED_HAS_NO_INTERNET
 * MOBILE_CONNECTED
 * OFFLINE
* Wifi Access Points Signal Strength Change

## Overview
[ConnectivityManager](http://developer.android.com/reference/android/net/ConnectivityManager.html) avaiable in Android API allows us to check, whether we are connected to WiFi network or mobile network. Despite this fact, we can be connected to WiFi network, but such network can be also disconnected from the Internet. This project shows, how can we create additional so called _InternetConnectionChangeReceiver_ which allows us to determine, if we really have access to the Internet besides being connected to WiFi network. We can do that by pinging sample remote host (e.g. www.google.com).

## API

### Initialize and register NetworkEvents

In your activity create `NetworkEvents` field.

```java
private NetworkEvents networkEvents;
```

In `onCreate()` method initialize object and register listener.
Pass `Context` and instance of the `Bus` to the constructor of `NetworkEvents` class.

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  setContentView(R.layout.activity_main);
  
  networkEvents = new NetworkEvents(this, BusProvider.getInstance());
  networkEvents.register();
}
```

### Setup event bus and resume NetworkEvents

Add [Otto Event Bus](http://square.github.io/otto/) to your project. Register and unregister bus properly in `onResume()` and `onPause()` methods. You can take a look on [sample project](https://github.com/pwittchen/NetworkEvents/tree/master/example) and [BusProvider](https://github.com/pwittchen/NetworkEvents/blob/master/example/src/main/java/pwittchen/com/networkevents/provider/BusProvider.java) class.

```java
@Override
protected void onResume() {
  super.onResume();
  BusProvider.getInstance().register(this);
  networkEvents.resume();
}

@Override
protected void onPause() {
  super.onPause();
  BusProvider.getInstance().unregister(this);
}
```

### Unregister NetworkEvents

In `onDestroy()` method unregister listener.

```java
@Override
protected void onDestroy() {
  super.onDestroy();
  networkEvents.unregister();
}
```

### Subscribe for ConnectivityStatusChangedEvent

Create method with `@Subscribe` annotation and listen `ConnectivityStatusChangedEvent`.

```java
@Subscribe
public void connectivityStatusChanged(ConnectivityStatusChangedEvent event) {
  Toast.makeText(this, event.getConnectivityStatus().toString(), Toast.LENGTH_SHORT).show();
  Toast.makeText(this, event.getWifiInfo().toString(), Toast.LENGTH_SHORT).show();
}
```

See the file: [ConnectivityStatusActivity.java](https://github.com/pwittchen/NetworkEvents/blob/master/example/src/main/java/pwittchen/com/networkevents/activity/ConnectivityStatusActivity.java) for more details.

### Subscribe for WifiAccessPointsSignalStrengthChangedEvent (optionally)

Optionally we can listen WifiAccessPointsSignalStrengthChangedEvent and perform desired action, when list of Access Points was refreshed. It occurs, when RSSI (signal strength) has changed. In the code snippet below, we retrieve list of all access points.

```java
@Subscribe
public void wifiAccessPointsRefreshed(WifiAccessPointsSignalStrengthChangedEvent event) {
  WifiManager wifiManager = (WifiManager) getSystemService(Context.WIFI_SERVICE);
  List<ScanResult> accessPoints = wifiManager.getScanResults();
}
```

Besides ConnectivityStatus, we can optionally retrieve WiFi info with `getWifiInfo()` method available in `ConnectivityStatusChangedEvent` class.

See the file: [AccessPointsScanActivity.java](https://github.com/pwittchen/NetworkEvents/blob/master/example/src/main/java/pwittchen/com/networkevents/activity/AccessPointsScanActivity.java) for more details.

### Sample application with library module dependency
In [example](https://github.com/pwittchen/NetworkEvents/tree/master/example) directory you can find sample application using InternetConnectionStateListener library via module dependency in Android Studio.

The following activities exists in the sample project:
* [BaseActivity.java](https://github.com/pwittchen/NetworkEvents/blob/master/example/src/main/java/pwittchen/com/networkevents/activity/BaseActivity.java) contains abstract `BaseActivity` class with all operations, which needs to be performed in derived classes.
* [ConnectivityStatusActivity.java](https://github.com/pwittchen/NetworkEvents/blob/master/example/src/main/java/pwittchen/com/networkevents/activity/ConnectivityStatusActivity.java) - contains basic usage of this library, which is listening ConnectivityStatus
* [AccessPointsScanActivity.java](https://github.com/pwittchen/NetworkEvents/blob/master/example/src/main/java/pwittchen/com/networkevents/activity/AccessPointsScanActivity.java) - contains example with subscribing WifiAccessPointsRefreshedEvent, so we can perform an action when list of access points was refreshed and strength of the signals was changed
* [RoomLocationActivity.java](https://github.com/pwittchen/NetworkEvents/blob/master/example/src/main/java/pwittchen/com/networkevents/activity/RoomLocationActivity.java) - experimental location inside the building based on MAC addresses (BSSID) of the WiFi Access Points
