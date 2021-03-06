# 室内定位开发包-智石科技

----------
详细流程文档请移步->[帮助与文档](http://help.brtbeacon.com)<br/>
Eclipse版Demo->[室内开发包Eclipse](https://github.com/BrightBeacon/BrightIndoorSDK_Eclipse)

### 一、简介
室内定位开发包是基于GIS框架和GEOS几何计算开源库，为开发者提供了的室内地图显示、路径规划、室内定位等相关GIS功能。**本开发包支持的Android版本为18或更高，库架构支持'armeabi', 'armeabi-v7a', 'x86'** 
### 二、项目集成
下载本资源，目录结构：
<br />Example/ 示例工程
<br />Documents/ API文档
<br />SDKs/ jar、so库和图标资源
#### 参见[详细集成文档](快速集成.md)

### 三、基础地图展示 
#### 3.1.配置AndroidManifest.xml所需权限

```
    <!-- 在线更新数据 -->
    <uses-permission android:name="android.permission.INTERNET" />
    <!-- 定位扫描beacon用 -->
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
	<!-- 拷贝数据到sd卡用，targetSdkVersion>=23还需手动申请 -->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
	<!--Android6.0 后蓝牙扫描需要蓝牙定位权限：COARSE或者FINE均可，targetSdkVersion>=23还需手动申请-->
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
	<application>
	 <!-- add your Activity -->
  	 <!-- 扫描beacon服务 -->
   	 <service android:name="com.ty.locationengine.ibeacon.BeaconService" />
	</application>
	
```

#### 3.2.新建actvity类，按需引人地图、定位支持库 
```
static { 
	System.loadLibrary("TYMapSDK"); 
	System.loadLibrary("TYLocationEngine");
} 
```
#### 3.3. 在Activity的布局文件xml中添加地图控件 
```
<com.ty.mapsdk.TYMapView android:id="@+id/map"
android:layout_width="match_parent"
android:layout_height="match_parent" />
```
#### 3.4.初始化mapView,代码详见BaseMapViewActivity.java 
##### 1.初始化地图环境；并设置地图数据保存在SD卡的位置，默认为TYData

```
TYMapEnvironment.initMapEnvironment(); 
TYMapEnvironment.setRootDirectoryForMapFiles(dir); 
```

##### 2.初始化地图监听、并加载地图数据 - 加载地图需要用到建筑标识buildingID，并传人授权appKey以验证地图使用权限

```
mapView = (TYMapView) findViewById(R.id.map);
//设置地图回调
mapView.addMapListener(this); 
//预设楼层
mapView.setFloor("F1");
//加载地图资源
mapView.init(Constants.BUILDING_ID,Constants.APP_KEY); 申请appKey:(http://open.brtbeacon.com)
```
#### 3.5.以上步骤完成后，地图资源准备就绪后回调如下方法；此处也可设置、获取楼层信息 
```
@Override public void mapViewDidLoad(TYMapView mapView,Error error) { 
	if(error == null){ 
		List<TYMapInfo> mapInfos = mapView.allMapInfo(); 
		//mapView.setFloor(mapInfos.get(0));
	}else { 
		Utils.showToast(this, error.toString()); 
	} 
}
```

### 四、地图弹窗
#### 4.1.弹窗样式配置文件callout_style.xml到res文件夹下的xml文件夹 
```
<calloutViewStyle backgroundColor="#ffffff"//弹窗背景颜色 
				  backgroundAlpha="255" //弹窗背景透明度 
				  frameColor="#66FFCC" //弹窗边框颜色 
				  cornerCurve="10" //弹窗边框拐角度数 
				  maxHeight ="800" //最大高度pixels 
				  maxWidth ="800" //最大宽度pixels 
				  anchor="5" /> //锚指向【0~8：左上/上中/右上/右中/右下/下中/左下/左中/自动】
```

#### 4.2.加载样式并显示弹窗 
```
Callout mapCallout = mapView.getCallout(); 
mapCallout.setStyle(R.xml.callout_style);//加载样式，或代码设定样式 
//mapCallout.setMaxWidth(Utils.dip2px(this, 300)); 
//mapCallout.setMaxHeight(Utils.dip2px(this, 300)); 
mapCallout.setContent(loadCalloutView(title, detail));//设置弹窗内容view 
mapCallout.show(location); 
```
### 五、线路规划 
#### 5.1.初始化路径管理器 
```
@Override public void mapViewDidLoad(final TYMapView mapView, Error error) { 
	if (error != null) return; 
	mapView.routeManager().addRouteManagerListener(this); 
} 
```
#### 5.1.设置起点 
```
mapView.showRouteStartSymbolOnCurrentFloor(startPoint);
``` 
#### 5.2.设置终点 
```
mapView.showRouteEndSymbolOnCurrentFloor(endPoint);
```

#### 5.3.开始规划，并实现接口TYOfflineRouteManager.TYOfflineRouteManagerListener 
```
//开始规划路线 routeManager.requestRoute(startPoint, endPoint); 
@Override public void didSolveRouteWithResult(TYOfflineRouteManager tyOfflineRouteManager, TYRouteResult rs) { 
	mapView.setRouteResult(rs); 
	mapView.showRouteResultOnCurrentFloor();//地图上显示线路
} 
@Override public void didFailSolveRouteWithError(TYOfflineRouteManager tyOfflineRouteManager, Exception e) { 
	//线路规划错误 
}
```

### 六、室内定位 
#### 6.1. 需要添加并申请蓝牙定位服务、相关权限见3.1 
#### 6.2. 初始化定位引擎，添加回调类，并启动定位

```
locationManager = new TYLocationManager(this, Constants.BUILDING_ID, Constants.APP_KEY);
locationManager.addLocationEngineListener(this);
locationManager.startLocation();
```
#### 6.3. 实现接口TYLocationManager.TYLocationManagerListener接口

```
@Override
public void didRangedBeacons(TYLocationManager arg0, List<TYBeacon> arg1) {
	//  Beacon扫描结果事件回调，返回符合扫描参数的所有Beacon
}

@Override
public void didRangedLocationBeacons(TYLocationManager arg0, List<TYPublicBeacon> arg1) {
	//  定位Beacon扫描结果事件回调，返回符合扫描参数的定位Beacon，定位Beacon包含坐标信息。此方法可用于辅助巡检，以及基于定位beacon的相关触发事件。
}

@Override
public void didUpdateDeviceHeading(TYLocationManager arg0, double newHeading) {
	// 设备方向改变事件回调。结合地图MapMode可以处理地图自动旋转，或箭头方向功能。
	//mapView.setMapMode(TYMapViewMode.TYMapViewModeDefault);
	//mapView.setMapMode(TYMapViewMode.TYMapViewModeFollowing);
	Log.i(TAG,"地图初始北偏角："+mapView.building.getInitAngle()+"；当前设备北偏角："+newHeading);
	 mapView.processDeviceRotation(newHeading);
}

@Override
public void didUpdateImmediateLocation(TYLocationManager arg0, TYLocalPoint arg1) {
	//  *  位置更新事件回调，位置更新并返回新的位置结果。
	// 与[TYLocationManager:didUpdateLocatin:]方法相近，此方法回调结果未融合设备传感器信息，灵敏度较高，适合用于行车场景下或传感器无效
}

@Override
public void didFailUpdateLocation(TYLocationManager tyLocationManager, Error error) {
	if (error != null) Log.i(TAG,error.toString());
}

@Override
public void didUpdateLocation(TYLocationManager arg0, TYLocalPoint newLocalPoint) {
	//  位置更新事件回调，位置更新并返回新的位置结果。
	//  与[TYLocationManager:didUpdateImmediationLocation:]方法相近，此方法回调结果融合设备传感器信息，稳定性较好，用于步行场景下。
	Log.i(TAG, newLocalPoint.getX()+" "+newLocalPoint.getY());
	mapView.showLocation(newLocalPoint);
	//mapView.showRemainingRouteResultOnCurrentFloor(newLocalPoint);
}
```
### 七、示例工程演示定位（不支持模拟器）

#### 配置示例工程演示定位 
使用与地图数据配套的iBeacon设备部署方案，才可以实现室内地图定位。 配置导航的beacon信息,[请下载配置端BrightBeacon](http://app.brtbeacon.com) 示例地图，需要准备6个iBeacon设备；配置参数列表如下：

<table>
<thead>
<tr>
<th>Floor</th>
<th>UUID </th>
<th> Major </th>
<th> Minor</th>
</tr>
</thead>
<tbody>
<tr>
<td>F1</td>
<td rowspan＝'2'> E2C56DB5-DFFB-48D2-B060-D0F5A71096E0 </td>
<td> 10000  </td>
<td> 11261 </td>
</tr>
<tr>
<td>F1</td>
<td> E2C56DB5-DFFB-48D2-B060-D0F5A71096E0 </td>
<td> 10000  </td>
<td> 11262 </td>
</tr>
<tr>
<td>F1</td>
<td> E2C56DB5-DFFB-48D2-B060-D0F5A71096E0 </td>
<td> 10000  </td>
<td> 11263 </td>
</tr>
<tr>
<td>F1</td>
<td> E2C56DB5-DFFB-48D2-B060-D0F5A71096E0 </td>
<td> 10000  </td>
<td> 11264 </td>
</tr>
<tr>
<td>F2</td>
<td> E2C56DB5-DFFB-48D2-B060-D0F5A71096E0 </td>
<td> 10000  </td>
<td> 11265 </td>
</tr>
<tr>
<td>F2</td>
<td> E2C56DB5-DFFB-48D2-B060-D0F5A71096E0 </td>
<td> 10000  </td>
<td> 11266 </td>
</tr>
</tbody>
</table>

### 八、使用你的地图

#### 获取你的地图参数 
①前往开放平台[开放平台http://open.brtbeacon.com](http://open.brtbeacon.com)并登录

②首次注册用户需创建【应用AppKey】，即可申请地图

②登录查看你的【建筑列表】获取AppKey，【设备管理】获取UUID等参数，填入示例工程即可

* [文档中心](http://help.brtbeacon.com)
* [GIS文档](https://developers.arcgis.com/android/10-2/api-reference/reference/classes.html)
* [社区提问](http://bbs.brtbeacon.com)
* [智石官网](http://www.brtbeacon.com)

#### 商务合作、地图绘制咨询[4000-999-023](tel:4000999023)