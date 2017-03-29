---
layout: post
title:  "MapFragment With LocationListener"
date:   2017-03-29 17:00:00 +0800
categories: android
tags: [android]
image:
  background: triangular.png
---

本文簡介使用SupportMapFragment與LocationListener於地圖定位目前位置

## 介紹:
- [定位權限](#定位權限檢查) 
- [MapFragment地圖設定](#mapfragment地圖設定)
- [監聽取得GPS經緯度](#監聽取得gps經緯度)
- [地圖顯示目前位置](#地圖顯示目前位置)

### 定位權限檢查
----
App首先於Manifest必須定義定位權限 
``<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />``，在android 6.0以上版本 (API level 23)，對於危險權限的存取需要經過使用者同意。

參考：[Normal and Dangerous Permissions](https://developer.android.com/guide/topics/security/permissions.html#normal-dangerous)

```java
if (ActivityCompat.checkSelfPermission(mContext, android.Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
    ActivityCompat.requestPermissions(mContext, LOCATION_FINE_PERMS, PERMISSIONS_REQUEST_ACCESS_FINE_LOCATION);
    Log.i(TAG, "requestPermissions !!!");
}else{ // permission Granted
    // do some thing
}
```


### MapFragment地圖設定
----
在Manifest定義啟用Google Maps Android API服務的金鑰
``` xml
<meta-data
            android:name="com.google.android.geo.API_KEY"
            android:value="@string/api_key" />
```

配置MapFragmentLayout時必須傳入``OnMapReadyCallback``實體給地圖準備完成時Callback通知

``` java
public class LocationMapActivity extends AppCompatActivity implements OnMapReadyCallback{
public SupportMapFragment mMapFragment ;
    private void setView(){
        mMapFragment = (SupportMapFragment) getSupportFragmentManager().findFragmentById(R.id.map);
        mMapFragment.getMapAsync(LocationMapActivity.this);
    }
    @Override
    public void onMapReady(GoogleMap map) {
        ...
    }
}
```
``` xml
    <fragment
        android:id="@+id/map"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        class="com.google.android.gms.maps.SupportMapFragment" />
```


### 監聽取得gps經緯度
----
#### 存取經緯度必須實作三種介面

1 .GoogleApiClient.ConnectionCallbacks

GoogleApiClient告知已連結or暫停，呼叫``onConnected(@Nullable Bundle bundle)``or``onConnectionSuspended(int i)``

2 .GoogleApiClient.OnConnectionFailedListener

GoogleApiClient告知連結失敗，呼叫``onConnectionFailed(@NonNull ConnectionResult connectionResult)``

3 .GoogleApiClient.OnConnectionFailedListener

LocationServices.FusedLocationApi告知取得經緯度資訊時，呼叫``onLocationChanged(Location location)``

``` java
public class LocationUpdate implements GoogleApiClient.ConnectionCallbacks
, GoogleApiClient.OnConnectionFailedListener
, LocationListener
{
    private GoogleApiClient mGoogleApiClient;
    private LocationRequest mLocationRequest;

    public LocationUpdate(Context context) {
        // Create a new global location parameters object
        mLocationRequest = LocationRequest.create();
        /*
         * Set the update interval
         */
        mLocationRequest.setInterval(INTERVAL);
        // Use high accuracy
        mLocationRequest.setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY);

        mGoogleApiClient = new GoogleApiClient.Builder(context)
        .addApi(LocationServices.API)
        .addConnectionCallbacks(this)
        .addOnConnectionFailedListener(this)
        .build();
     ...

    public void startPeriodicUpdates() {
        ...
        LocationServices.FusedLocationApi.requestLocationUpdates(mGoogleApiClient, mLocationRequest, this);
    }
}
``` 
#### 各介面Callback如下
``` java

    // GoogleApiClient.ConnectionCallbacks
    @Override
    public void onConnected(@Nullable Bundle bundle) {
    }

    // GoogleApiClient.ConnectionCallbacks
    @Override
    public void onConnectionSuspended(int i) {
    }

    // GooglePlayServicesClient.OnConnectionFailedListener
    @Override
    public void onConnectionFailed(@NonNull ConnectionResult connectionResult){
    }

    // LocationListener
    @Override
    public void onLocationChanged(Location location) {
    }
``` 

#### 取得Location資訊發出廣播
在取得經緯度資訊後，發出廣播通知讓畫面呈現目前定位座標.

```java
private synchronized void setFuseLocation(Location currentLocation){
    Intent intent = new Intent(ACTION);
    Bundle bundle = new Bundle();
    bundle.putSerializable("LocationInfo",mLocationInfo);
    intent.putExtras(bundle);
    mContext.sendBroadcast(intent);
}
```


### 地圖顯示目前位置
----

畫面利用MarkerOptions物件，將title 片段描述snippet 經緯度LatLng存入，並於mMap呈現.
```java
public class LocationMapActivity extends AppCompatActivity{
...
    @Override
    public void onResultLocation(LocationUpdate.LocationInfo mLocationInfo) {
    ...
        double lat = mLocationInfo.lat;
        double lng = mLocationInfo.lng;
        String title = Position_Title_Default;
        String snippet = ""+mLocationInfo.xLng+"_"+mLocationInfo.yLat;
        LatLng location = new LatLng(lat, lng);
        MarkerOptions markerOpt = new MarkerOptions();
        markerOpt.title(title);
        markerOpt.snippet(snippet);
        markerOpt.position(location);
        currentPositionMarker = mMap.addMarker(markerOpt);
        currentPositionMarker.showInfoWindow();
    }
}
```
