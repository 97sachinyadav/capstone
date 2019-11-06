  
<?xml version="1.0" encoding="utf-8"?>

<manifest

package="org.physical_web.physicalweb" xmlns:android="http://schemas.android.com/apk/res/android">

<uses-feature
android:name="android.hardware.bluetooth_le" android:required="true"/>

<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:required="true"
android:name="android.permission.ACCESS_WIFI_STATE"/>
<uses-permission android:required="true"
android:name="android.permission.CHANGE_WIFI_STATE"/>
<!-- This is required for the scan library. -->
<uses-permission android:name="android.permission.WAKE_LOCK"/>

<application
android:allowBackup="true" android:fullBackupContent="true" android:icon="@drawable/ic_launcher" android:label="@string/app_name" android:supportsRtl="true" android:theme="@style/CustomAppTheme">
<activity
android:name=".MainActivity" android:label="@string/app_name" android:launchMode="singleTask" android:windowSoftInputMode="adjustPan">
<intent-filter>
<action android:name="android.intent.action.MAIN"/>

<category android:name="android.intent.category.LAUNCHER"/>
</intent-filter>
</activity>
<activity
android:name=".BroadcastActivity" android:label="@string/app_name" android:launchMode="singleInstance" android:theme="@android:style/Theme.Translucent.NoTitleBar" android:excludeFromRecents="true">
<intent-filter>
<action android:name="android.intent.action.SEND" />
<category android:name="android.intent.category.DEFAULT" />
<data android:mimeType="text/plain" />
</intent-filter>
<intent-filter>
<action android:name="android.intent.action.SEND" />
<category android:name="android.intent.category.DEFAULT" />
<data android:mimeType="image/*" />
</intent-filter>
<intent-filter>
<action android:name="android.intent.action.SEND" />
<category android:name="android.intent.category.DEFAULT" />
<data android:mimeType="text/html" />
</intent-filter>
<intent-filter>
<action android:name="android.intent.action.SEND" />
<category android:name="android.intent.category.DEFAULT" />
<data android:mimeType="video/*" />
</intent-filter>
<intent-filter>
<action android:name="android.intent.action.SEND" />
<category android:name="android.intent.category.DEFAULT" />
<data android:mimeType="audio/*" />
</intent-filter>
</activity>
<activity android:name=".OfflineTransportConnectionActivity" android:label="@string/app_name" android:launchMode="singleTask"
android:theme="@android:style/Theme.Translucent.NoTitleBar" android:excludeFromRecents="true">
</activity>
<service
android:name=".UrlDeviceDiscoveryService" android:enabled="true" android:exported="false">
</service>

<service
android:name=".ScreenListenerService"
android:enabled="true" android:exported="false">
</service>

<receiver
android:name=".AutostartPwoDiscoveryServiceReceiver" android:exported="false">
<intent-filter>
<action android:name="android.intent.action.BOOT_COMPLETED" />
</intent-filter>
</receiver>

<!-- This is required for the scan library. -->
<service
android:name="org.uribeacon.scan.compat.ScanWakefulService" android:exported="false">
</service>
<service
android:name="org.uribeacon.config.GattService" android:exported="false">
</service>
<service
android:name=".PhysicalWebBroadcastService" android:enabled="true" android:stopWithTask="false" android:exported="false">
</service>
<service
android:name=".FileBroadcastService" android:enabled="true" android:stopWithTask="false" android:exported="false">
</service>
<service android:name=".FatBeaconBroadcastService" android:enabled="true" android:stopWithTask="false" android:exported="false">
</service>
<!-- This is required for the scan library. -->
<receiver android:name="org.uribeacon.scan.compat.ScanWakefulBroadcastReceiver">
</receiver>

<activity
android:name=".OobActivity"
android:label="OobActivity" android:theme="@style/Theme.NoActionBar">
</activity>

<provider
android:name="android.support.v4.content.FileProvider" android:authorities="org.physical_web.fileprovider" android:exported="false" android:grantUriPermissions="true">
<meta-data
android:name="android.support.FILE_PROVIDER_PATHS" android:resource="@xml/file_paths" />

</provider>
</application>

</manifest>

Java Code for Scanning near by Beacons
package  org.physical_web.physicalweb;

import  org.physical_web.collection.PhysicalWebCollection; import  org.physical_web.collection.PwPair;
import  org.physical_web.collection.PwsResult; import  org.physical_web.collection.UrlDevice;

import  android.annotation.SuppressLint; import  android.app.ListFragment;
import  android.content.ComponentName; import  android.content.Context; import  android.content.Intent;
import  android.content.ServiceConnection; import  android.graphics.PorterDuff;
import  android.graphics.drawable.AnimationDrawable; import  android.net.wifi.WifiInfo;
import  android.net.wifi.WifiManager; import  android.os.Bundle;
import  android.os.Handler; import  android.os.IBinder; import  android.os.Looper;
import  android.support.v4.content.ContextCompat; import  android.view.LayoutInflater;
import  android.view.Menu; import  android.view.View; import  android.view.ViewGroup;
import  android.widget.BaseAdapter; import  android.widget.Button; import  android.widget.ImageView;
import  android.widget.ListView; import  android.widget.TextView;

import  java.text.DecimalFormat; import  java.util.ArrayList; import  java.util.Collections; import  java.util.Date;
import  java.util.List;
import  java.util.concurrent.TimeUnit;

public  class  NearbyBeaconsFragment  extends  ListFragment
implements UrlDeviceDiscoveryService.UrlDeviceDiscoveryListener,
SwipeRefreshWidget.OnRefreshListener  { private  static  final  String  TAG  =
NearbyBeaconsFragment.class.getSimpleName();
private  static  final  long  FIRST_SCAN_TIME_MILLIS  = TimeUnit.SECONDS.toMillis(2);
private  static  final  long  SECOND_SCAN_TIME_MILLIS  = TimeUnit.SECONDS.toMillis(5);
private  static  final  long  THIRD_SCAN_TIME_MILLIS  = TimeUnit.SECONDS.toMillis(10);
private  List<String>  mGroupIdQueue;
private  PhysicalWebCollection  mPwCollection  =  null; private  TextView  mScanningAnimationTextView;
private  AnimationDrawable  mScanningAnimationDrawable; private  Handler  mHandler;
private  NearbyBeaconsAdapter  mNearbyDeviceAdapter; private  SwipeRefreshWidget  mSwipeRefreshWidget; private  boolean  mSecondScanComplete;
private  boolean  mFirstTime;
private  DiscoveryServiceConnection  mDiscoveryServiceConnection; private  boolean  mMissedEmptyGroupIdQueue  =  false;
private  SwipeDismissListViewTouchListener  mTouchListener; private  WifiDirectConnect  mWifiDirectConnect;
private  BluetoothSite  mBluetoothSite;

private  Runnable  mFirstScanTimeout  =  new  Runnable()  { @Override
public void run() {
Log.d(TAG,  "running  first  scan  timeout"); if  (!mGroupIdQueue.isEmpty())  {
emptyGroupIdQueue(); setRefreshWidgetInvisible();
}
}
};

private  Runnable  mSecondScanTimeout  =  new  Runnable()  { @Override
public void run() {
Log.d(TAG,  "running  second  scan  timeout"); emptyGroupIdQueue();
mSecondScanComplete  =  true; setRefreshWidgetInvisible();
if  (mNearbyDeviceAdapter.getCount()  ==  0)  {
int  tintColor  =  ContextCompat.getColor(getActivity(), R.color.physical_web_logo);
mScanningAnimationDrawable.setColorFilter(tintColor, PorterDuff.Mode.SRC_IN);

mScanningAnimationTextView.setText(R.string.empty_nearby_beacons_list_ text_no_results);
}
}
};

private  Runnable  mThirdScanTimeout  =  new  Runnable()  { @Override
public void run() {
Log.d(TAG,  "running  third  scan  timeout"); mDiscoveryServiceConnection.disconnect();
}
};

private  class  DiscoveryServiceConnection  implements  ServiceConnection
{
private  UrlDeviceDiscoveryService  mDiscoveryService; private  boolean  mRequestCachedUrlDevices;

@Override
public  synchronized  void  onServiceConnected(ComponentName  className, IBinder  service)  {

UrlDeviceDiscoveryService.LocalBinder  localBinder  = (UrlDeviceDiscoveryService.LocalBinder)  service;
mDiscoveryService  =  localBinder.getServiceInstance();

mDiscoveryService.addCallback(NearbyBeaconsFragment.this); if  (!mRequestCachedUrlDevices)  {
mDiscoveryService.restartScan();
}
mPwCollection  =  mDiscoveryService.getPwCollection(); onUrlDeviceDiscoveryUpdate();
startScanningDisplay(mDiscoveryService.getScanStartTime(), mDiscoveryService.hasResults());
}

@Override
public  synchronized  void  onServiceDisconnected(ComponentName className)  {
//  onServiceDisconnected  gets  called  when  the  connection  is unintentionally disconnected,
//  which  should  never  happen  for  us  since  this  is  a  local  service mDiscoveryService  =  null;
}

public  synchronized  void  connect(boolean  requestCachedUrlDevices)  { if  (mDiscoveryService  !=  null)  {
return;
}

mRequestCachedUrlDevices  =  requestCachedUrlDevices; Intent  intent  =  new  Intent(getActivity(),
UrlDeviceDiscoveryService.class); getActivity().startService(intent);
getActivity().bindService(intent,  this,  Context.BIND_AUTO_CREATE);
}

public synchronized void disconnect() { if  (mDiscoveryService  ==  null)  {
return;
}

mDiscoveryService.removeCallback(NearbyBeaconsFragment.this); mDiscoveryService  =  null;
getActivity().unbindService(this); stopScanningDisplay();
}
}

private  void  initialize(View  rootView)  { setHasOptionsMenu(true);
mGroupIdQueue  =  new  ArrayList<>(); mHandler  =  new  Handler();

mSwipeRefreshWidget  =  (SwipeRefreshWidget) rootView.findViewById(R.id.swipe_refresh_widget);

mSwipeRefreshWidget.setColorSchemeResources(R.color.swipe_refresh_widg et_first_color,

R.color.swipe_refresh_widget_second_color); mSwipeRefreshWidget.setOnRefreshListener(this);


getActivity().getActionBar().setTitle(R.string.title_nearby_beacons); mNearbyDeviceAdapter  =  new  NearbyBeaconsAdapter(); setListAdapter(mNearbyDeviceAdapter);
//Get  the  top  drawable mScanningAnimationTextView  =  (TextView)
rootView.findViewById(android.R.id.empty); mScanningAnimationDrawable  =
(AnimationDrawable) mScanningAnimationTextView.getCompoundDrawables()[1];
ListView  listView  =  (ListView) rootView.findViewById(android.R.id.list);
mDiscoveryServiceConnection  =  new  DiscoveryServiceConnection(); mWifiDirectConnect  =  new  WifiDirectConnect(getActivity()); mBluetoothSite  =  new  BluetoothSite(getActivity()); mTouchListener  =
new  SwipeDismissListViewTouchListener( listView,
new  SwipeDismissListViewTouchListener.DismissCallbacks()  { @Override
public  boolean  canDismiss(int  position)  { return true;
}

@Override
public  void  onDismiss(ListView  listView,  int  position)  {

Utils.addBlocked(mNearbyDeviceAdapter.getItem(position)); Utils.saveBlocked(getActivity());
if  (mMissedEmptyGroupIdQueue)  { mMissedEmptyGroupIdQueue  =  false; emptyGroupIdQueue();
}
}
});
listView.setOnTouchListener(mTouchListener);

//  Setting  this  scroll  listener  is  required  to  ensure  that  during ListView  scrolling,
//  we  don't  look  for  swipes. listView.setOnScrollListener(mTouchListener.makeScrollListener()); Utils.restoreFavorites(getActivity()); Utils.restoreBlocked(getActivity());
}

@Override
public  View  onCreateView(LayoutInflater  layoutInflater,  ViewGroup container,
Bundle  savedInstanceState)  {
mFirstTime  =  true; View  rootView  =
layoutInflater.inflate(R.layout.fragment_nearby_beacons,  container, false);
initialize(rootView); return  rootView;
}

@Override
public  void  onResume()  { super.onResume();
getActivity().getActionBar().setTitle(R.string.title_nearby_beacons); getActivity().getActionBar().setDisplayHomeAsUpEnabled(false);
if  (mFirstTime
&&  !PermissionCheck.getInstance().isCheckingPermissions())  { restartScan();
}
mFirstTime  =  false;
}

public  void  restartScan()  {
if  (mDiscoveryServiceConnection  !=  null)  { mDiscoveryServiceConnection.connect(true);
}
}

@Override
public void onPause() { super.onPause();
mDiscoveryServiceConnection.disconnect();
}

@Override
public  void  onPrepareOptionsMenu(Menu  menu)  { super.onPrepareOptionsMenu(menu); menu.findItem(R.id.action_about).setVisible(true);
}

@Override
public  void  onListItemClick(ListView  l,  View  v,  int  position,  long  id)
{
PwPair  item  =  mNearbyDeviceAdapter.getItem(position);
// If we are scanning or user clicked on folder
if  (mScanningAnimationDrawable.isRunning()  ||  isFolderItem(item))  {
//  Don't  respond  to  touch  events return;
}
//  Get  the  url  for  the  given  item PwsResult  pwsResult  =  item.getPwsResult();
if  (Utils.isWifiDirectDevice(item.getUrlDevice()))  {
//  Initiate  WifiDirect  Connection  request  to  device mWifiDirectConnect.connect(item.getUrlDevice(),
pwsResult.getTitle());
}  else  if  (Utils.isFatBeaconDevice(item.getUrlDevice()))  { if  (!mBluetoothSite.isRunning())  {
mBluetoothSite.connect(pwsResult.getSiteUrl(), pwsResult.getTitle());
}
} else {
Intent  intent  =  Utils.createNavigateToUrlIntent(pwsResult); startActivity(intent);
}

@Override
public  void  onUrlDeviceDiscoveryUpdate()  {
for  (PwPair  pwPair  :  mPwCollection.getGroupedPwPairsSortedByRank( new  Utils.PwPairRelevanceComparator()))  {
String  groupId  =  Utils.getGroupId(pwPair.getPwsResult()); Log.d(TAG,  "groupid  to  add  "  +  groupId);
if  (mNearbyDeviceAdapter.containsGroupId(groupId))  { mNearbyDeviceAdapter.updateItem(pwPair);
}  else  if  (!mGroupIdQueue.contains(groupId) &&  !Utils.isBlocked(pwPair))  {
mGroupIdQueue.add(groupId);
}
}

if(mGroupIdQueue.isEmpty()  ||  !mSecondScanComplete)  { return;
}
//  Since  this  callback  is  given  on  a  background  thread  and  we  want
//  to  update  the  list  adapter  (which  can  only  be  done  on  the  UI  thread)
//  we  have  to  interact  with  the  adapter  on  the  UI  thread. new  Handler(Looper.getMainLooper()).post(new  Runnable()  {
@Override
public void run() { emptyGroupIdQueue();
}
});
}

private  void  stopScanningDisplay()  {
//  Cancel  the  scan  timeout  callback  if  still  active  or  else  it  may  fire later.
mHandler.removeCallbacks(mFirstScanTimeout); mHandler.removeCallbacks(mSecondScanTimeout); mHandler.removeCallbacks(mThirdScanTimeout);

// Change the display appropriately setRefreshWidgetInvisible();
}

private void startScanningDisplay(long scanStartTime, boolean hasResults)
{
//  Start  the  scanning  animation  only  if  we  don't  haven't  already  been
scanning
// for long enough
Log.d(TAG, "startScanningDisplay " +  scanStartTime + "  " + hasResults); long  elapsedMillis  =  new  Date().getTime()  -  scanStartTime;
if  (elapsedMillis  <  FIRST_SCAN_TIME_MILLIS
||  (elapsedMillis  <  SECOND_SCAN_TIME_MILLIS  &&  !hasResults))  { mNearbyDeviceAdapter.clear(); mScanningAnimationDrawable.setColorFilter(null);
mScanningAnimationTextView.setText(R.string.empty_nearby_beacons_list_ text);
mScanningAnimationDrawable.start();
} else { setRefreshWidgetInvisible();
}




0);



}
mSecondScanComplete  =  false;
long firstDelay = Math.max(FIRST_SCAN_TIME_MILLIS - elapsedMillis, 0); long  secondDelay  =  Math.max(SECOND_SCAN_TIME_MILLIS  -  elapsedMillis,

long thirdDelay = Math.max(THIRD_SCAN_TIME_MILLIS - elapsedMillis, 0); mHandler.postDelayed(mFirstScanTimeout,  firstDelay); mHandler.postDelayed(mSecondScanTimeout,  secondDelay); mHandler.postDelayed(mThirdScanTimeout,  thirdDelay);

@Override
public  void  onRefresh()  { mGroupIdQueue.clear(); mNearbyDeviceAdapter.clear();

mDiscoveryServiceConnection.disconnect(); mSwipeRefreshWidget.setRefreshing(true); mDiscoveryServiceConnection.connect(false);
}

private  void  emptyGroupIdQueue()  {
if  (SwipeDismissListViewTouchListener.isLocked())  { mMissedEmptyGroupIdQueue  =  true;
return;
}
List<PwPair>  pwPairs  =  new  ArrayList<>(); for  (String  groupId  :  mGroupIdQueue)  {
Log.d(TAG,  "groupid  "  +  groupId);
pwPairs.add(Utils.getTopRankedPwPairByGroupId(mPwCollection, groupId));
}
Collections.sort(pwPairs,  new  Utils.PwPairRelevanceComparator()); for  (PwPair  pwPair  :  pwPairs)  {
mNearbyDeviceAdapter.addItem(pwPair);
}
mGroupIdQueue.clear(); mNearbyDeviceAdapter.notifyDataSetChanged();
}

private  static  boolean  isFolderItem(PwPair  item)  {
return item.getUrlDevice() == null && item.getPwsResult().getSiteUrl()
== null;
}

private  void  setRefreshWidgetInvisible()  { mSwipeRefreshWidget.setRefreshing(false); mScanningAnimationDrawable.stop(); mScanningAnimationTextView.setVisibility(View.INVISIBLE);
}

private  class  NearbyBeaconsAdapter  extends  BaseAdapter  { private  List<PwPair>  mPwPairs;
private  int  mNumberOfHideableResults;

NearbyBeaconsAdapter()  { mPwPairs  =  new  ArrayList<>(); mNumberOfHideableResults  =  0;
}

public  void  addItem(PwPair  pwPair)  {
if  (Utils.isResolvableDevice(pwPair.getUrlDevice()))  { mPwPairs.add(mPwPairs.size()  -  mNumberOfHideableResults,  pwPair); return;
}
if  (mNumberOfHideableResults  ==  0)  {
mPwPairs.add(new  PwPair(null,  new  PwsResult(null,  null))); mNumberOfHideableResults++;
}
mNumberOfHideableResults++; mPwPairs.add(pwPair);
}

public  void  updateItem(PwPair  pwPair)  {
String  groupId  =  Utils.getGroupId(pwPair.getPwsResult()); for  (int  i  =  0;  i  <  mPwPairs.size();  ++i)  {
if  (isFolderItem(mPwPairs.get(i)))  { continue;
}
if (Utils.getGroupId(mPwPairs.get(i).getPwsResult()).equals(groupId))  {
mPwPairs.set(i,  pwPair); return;
}
}
throw new RuntimeException("Cannot find PwPair with group " + groupId);
}

public  boolean  containsGroupId(String  groupId)  { for  (PwPair  pwPair  :  mPwPairs)  {
if  (isFolderItem(pwPair))  { continue;
}
if  (Utils.getGroupId(pwPair.getPwsResult()).equals(groupId))  { return true;
}
}
return false;
}

@Override
public  int  getCount()  { return  mPwPairs.size();
}

@Override
public  PwPair  getItem(int  i)  { return  mPwPairs.get(i);
}

@Override
public  long  getItemId(int  i)  { return i;
}

private  void  setText(View  view,  int  textViewId,  String  text)  { ((TextView)  view.findViewById(textViewId)).setText(text);
}

@SuppressLint("InflateParams") @Override
public  View  getView(int  i,  View  view,  ViewGroup  viewGroup)  { PwPair  pwPair  =  getItem(i);
PwsResult  pwsResult  =  pwPair.getPwsResult(); if  (isFolderItem(pwPair))  {
view  = getActivity().getLayoutInflater().inflate(R.layout.folder_item_nearby_ beacon,
viewGroup,  false);
WifiManager  wifiManager  =  (WifiManager)  getActivity(). getSystemService(Context.WIFI_SERVICE);
WifiInfo  wifiInfo  =  wifiManager.getConnectionInfo(); String  ssid  =  wifiInfo.getSSID().trim();
if  (ssid.charAt(0)  ==  '"'  &&  ssid.charAt(ssid.length()  -  1)  ==  '"')
{
setText(view,  R.id.title,  ssid.substring(1,  ssid.length()  -  1));
} else {
setText(view,  R.id.title,  "Wireless  Network");
}
return  view;
}
view  = getActivity().getLayoutInflater().inflate(R.layout.list_item_nearby_be acon,
viewGroup,  false);
setText(view,  R.id.title,  pwsResult.getTitle());
if  (Utils.isFatBeaconDevice(pwPair.getUrlDevice()))  {
setText(view,  R.id.url,  getString(R.string.FatBeacon_URL)  + pwsResult.getSiteUrl());
} else {
setText(view,  R.id.url,  pwsResult.getSiteUrl());
}
if  (Utils.isResolvableDevice(pwPair.getUrlDevice()))  { ((ImageView)  view.findViewById(R.id.icon)).setImageBitmap(
Utils.getBitmapIcon(mPwCollection,  pwsResult));
} else {
((ImageView)  view.findViewById(R.id.icon))
.setImageResource(R.drawable.unresolved_result_icon);
}
setText(view,  R.id.description,  pwsResult.getDescription()); final  String  siteUrl  =  pwsResult.getSiteUrl();

if  (Utils.isFavorite(siteUrl))  {
((Button)  view.findViewById(R.id.star)).setBackgroundResource( R.drawable.ic_star_black_24dp);
((Button)  view.findViewById(R.id.star)).setOnClickListener(new View.OnClickListener()  {
@Override
public  void  onClick(View  v)  { Utils.toggleFavorite(siteUrl); Utils.saveFavorites(getActivity()); ((Button)
v).setBackgroundResource(R.drawable.ic_star_border_black_24dp); notifyDataSetChanged();
}
});
} else {
((Button)  view.findViewById(R.id.star)).setBackgroundResource( R.drawable.ic_star_border_black_24dp);
((Button)  view.findViewById(R.id.star)).setOnClickListener(new View.OnClickListener()  {
@Override
public  void  onClick(View  v)  { Utils.toggleFavorite(siteUrl); Utils.saveFavorites(getActivity()); ((Button)
v).setBackgroundResource(R.drawable.ic_star_black_24dp); notifyDataSetChanged();
}
});
}

if  (Utils.isDebugViewEnabled(getActivity()))  { updateDebugView(pwPair,  view);

view.findViewById(R.id.ranging_debug_container).setVisibility(View.VIS IBLE);
view.findViewById(R.id.metadata_debug_container).setVisibility(View.VI SIBLE);
} else {

view.findViewById(R.id.ranging_debug_container).setVisibility(View.GON E);

view.findViewById(R.id.metadata_debug_container).setVisibility(View.GO NE);
}
return  view;
}

private  void  updateDebugView(PwPair  pwPair,  View  view)  {
//  Ranging  debug  line
UrlDevice  urlDevice  =  pwPair.getUrlDevice(); if  (Utils.isBleUrlDevice(urlDevice))  {
setText(view,  R.id.ranging_debug_tx_power, getString(R.string.ranging_debug_tx_power_prefix)  +
Utils.getTxPower(urlDevice));
setText(view,  R.id.ranging_debug_rssi, getString(R.string.ranging_debug_rssi_prefix)  +
Utils.getSmoothedRssi(urlDevice));
setText(view,  R.id.ranging_debug_distance, getString(R.string.ranging_debug_distance_prefix)
+ new DecimalFormat("##.##").format(Utils.getDistance(urlDevice)));
setText(view,  R.id.ranging_debug_region, getString(R.string.ranging_debug_region_prefix)  +
Utils.getRegionString(urlDevice));
} else {
setText(view,  R.id.ranging_debug_tx_power,  ""); setText(view,  R.id.ranging_debug_rssi,  ""); setText(view,  R.id.ranging_debug_distance,  ""); setText(view,  R.id.ranging_debug_region,  "");
}

//  Metadata  debug  line
setText(view,  R.id.metadata_debug_scan_time, getString(R.string.metadata_debug_scan_time_prefix)
+ new DecimalFormat("##.##s").format(Utils.getScanTimeMillis(urlDevice)  / 1000.0));

PwsResult  pwsResult  =  pwPair.getPwsResult(); setText(view,  R.id.metadata_debug_rank,
getString(R.string.metadata_debug_rank_prefix)
+  new  DecimalFormat("##.##").format(0));	//  We  currently  do  not use rank.
if  (Utils.isResolvableDevice(urlDevice))  { setText(view,  R.id.metadata_debug_pws_trip_time,
getString(R.string.metadata_debug_pws_trip_time_prefix)
+  new  DecimalFormat("##.##s")
.format(Utils.getPwsTripTimeMillis(pwsResult)  /  1000.0));
}
setText(view,  R.id.metadata_debug_groupid, getString(R.string.metadata_debug_groupid_prefix)  +
Utils.getGroupId(pwsResult));
}

public void clear() { mPwPairs.clear(); mNumberOfHideableResults  =  0; notifyDataSetChanged();
}
}

}
