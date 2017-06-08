---
title: Android连接WIFI
date: 2016-08-04 10:22:01
tags: 
	- WIFI
	- Android 
---

---
转载请说明出处！
作者：[kqw攻城狮](http://kongqw.github.io/about/index.html)
出处：[个人站](http://kongqw.github.io) | [CSDN](http://blog.csdn.net/q4878802/)

---

# 效果图
## 打开WIFI并获取WIFI列表

![打开WIFI并获取WIFI列表](http://img.blog.csdn.net/20160818142227952)

## 连接到指定WIFI

![连接到指定WIFI](http://img.blog.csdn.net/20160818142248858)

## 直接连接配置过的WIFI

![直接连接配置过的WIFI](http://img.blog.csdn.net/20160818142308905)

## 密码错误

![密码错误](http://img.blog.csdn.net/20160818142330936)



# 源码

[KqwWifiManagerDemo](https://github.com/kongqw/KqwWifiManagerDemo)


WIFI的获取、连接状态等等的信息，都是通过广播获取的.

下面介绍了主要的方法，更多请查看[KqwWifiManager](https://github.com/kongqw/KqwWifiManagerDemo/blob/master/app/src/main/java/kong/qingwei/kqwwifimanagerdemo/KqwWifiManager.java)

# 注册广播接收者

``` java
<!-- 监听网络状态的广播接收者 -->
<receiver android:name=".KqwWifiManager$NetworkBroadcastReceiver">
    <intent-filter>
        <!-- AP扫描完成，客户端得到可用的结果集 -->
        <action android:name="android.net.wifi.SCAN_RESULTS" />
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
        <action android:name="android.net.wifi.WIFI_STATE_CHANGED" />
        <action android:name="android.net.wifi.STATE_CHANGE" />
        <action android:name="android.net.wifi.supplicant.STATE_CHANGE" />
    </intent-filter>
</receiver>
```
# 广播接收者

``` java
/**
 * 广播接收者
 */
public static class NetworkBroadcastReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        try {
            String action = intent.getAction();
            Log.i(TAG, "onReceive: action = " + action);
            // 监听WIFI的启用状态
            if (WifiManager.WIFI_STATE_CHANGED_ACTION.equals(action)) {
                int wifiState = intent.getIntExtra(WifiManager.EXTRA_WIFI_STATE, 0);
                switch (wifiState) {
                    case WifiManager.WIFI_STATE_ENABLING:
                        Log.i(TAG, "onReceive: 正在打开 WIFI...");
                        break;
                    case WifiManager.WIFI_STATE_ENABLED:
                        Log.i(TAG, "onReceive: WIFI 已打开");
                        if (null != mOnWifiEnabledListener) {
                            mOnWifiEnabledListener.onWifiEnabled(true);
                            mOnWifiEnabledListener.onFinish();
                        }
                        break;
                    case WifiManager.WIFI_STATE_DISABLING:
                        Log.i(TAG, "onReceive: 正在关闭 WIFI...");
                        break;
                    case WifiManager.WIFI_STATE_DISABLED:
                        Log.i(TAG, "onReceive: WIFI 已关闭");
                        if (null != mOnWifiEnabledListener) {
                            mOnWifiEnabledListener.onWifiEnabled(false);
                            mOnWifiEnabledListener.onFinish();
                        }
                        break;
                    case WifiManager.WIFI_STATE_UNKNOWN:
                        Log.i(TAG, "onReceive: WIFI 状态未知!");
                        break;
                    default:
                        break;
                }
            }

            // WIFI扫描完成
            if (WifiManager.SCAN_RESULTS_AVAILABLE_ACTION.equals(action)) {
                if (null != mOnWifiScanResultsListener) {
                    mOnWifiScanResultsListener.onScanResults(getScanResults());
                    mOnWifiScanResultsListener.onFinish();
                }
            }

            // WIFI 连接状态的监听（只有WIFI可用的时候，监听才会有效）
            if (WifiManager.NETWORK_STATE_CHANGED_ACTION.equals(action)) {
                NetworkInfo networkInfo = intent.getParcelableExtra(WifiManager.EXTRA_NETWORK_INFO);
                if (null != networkInfo && networkInfo.isConnected()) {
                    WifiInfo wifiInfo = intent.getParcelableExtra(WifiManager.EXTRA_WIFI_INFO);
                    if (null != wifiInfo && String.format("\"%s\"", mConnectingSSID).equals(wifiInfo.getSSID()) && null != mOnWifiConnectListener) {
                        // WIFI连接成功
                        mOnWifiConnectListener.onSuccess(wifiInfo.getSSID());
                        mOnWifiConnectListener.onFinish();
                        mOnWifiConnectListener = null;
                    }
                }
            }

            // WIFI连接过程的监听
            if (intent.getAction().equals(WifiManager.SUPPLICANT_STATE_CHANGED_ACTION)) {
                WifiInfo wifiInfo = mWifiManager.getConnectionInfo();
                SupplicantState state = wifiInfo.getSupplicantState();
                switch (state) {
                    case INTERFACE_DISABLED: // 接口禁用
                        Log.i(TAG, "onReceive: INTERFACE_DISABLED 接口禁用");
                        break;
                    case DISCONNECTED:// 断开连接
                    case INACTIVE: // 不活跃的
                        Log.i(TAG, "onReceive: INACTIVE 不活跃的 DISCONNECTED:// 断开连接");
                        if (null != mOnWifiConnectListener) {
                            // 断开当前网络失败
                            mOnWifiConnectListener.onFailure();
                            // 连接完成
                            mOnWifiConnectListener.onFinish();
                            mOnWifiConnectListener = null;
                            mConnectingSSID = null;
                        }
                        break;
                    case SCANNING: // 正在扫描
                        Log.i(TAG, "onReceive: SCANNING 正在扫描");
                        break;
                    case AUTHENTICATING: // 正在验证
                        Log.i(TAG, "onReceive: AUTHENTICATING: // 正在验证");
                        if (null != mOnWifiConnectListener) {
                            mOnWifiConnectListener.onConnectingMessage("正在验证");
                        }
                        break;
                    case ASSOCIATING: // 正在关联
                        Log.i(TAG, "onReceive: ASSOCIATING: // 正在关联");
                        if (null != mOnWifiConnectListener) {
                            mOnWifiConnectListener.onConnectingMessage("正在关联");
                        }
                        break;
                    case ASSOCIATED: // 已经关联
                        Log.i(TAG, "onReceive: ASSOCIATED: // 已经关联");
                        if (null != mOnWifiConnectListener) {
                            mOnWifiConnectListener.onConnectingMessage("已经关联");
                        }
                        break;
                    case FOUR_WAY_HANDSHAKE:
                        Log.i(TAG, "onReceive: FOUR_WAY_HANDSHAKE:");
                        break;
                    case GROUP_HANDSHAKE:
                        Log.i(TAG, "onReceive: GROUP_HANDSHAKE:");
                        break;
                    case COMPLETED: // 完成
                        Log.i(TAG, "onReceive: COMPLETED: // 完成");
                        if (null != mOnWifiConnectListener) {
                            mOnWifiConnectListener.onConnectingMessage("正在连接...");
                        }
                        break;
                    case DORMANT:
                        Log.i(TAG, "onReceive: DORMANT:");
                        break;
                    case UNINITIALIZED: // 未初始化
                        Log.i(TAG, "onReceive: UNINITIALIZED: // 未初始化");
                        break;
                    case INVALID: // 无效的
                        Log.i(TAG, "onReceive: INVALID: // 无效的");
                        break;
                    default:
                        break;
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```


# 权限

``` java
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

# 打开WIFI

``` java
/**
 * 打开Wifi
 */
public void openWifi(@NonNull OnWifiEnabledListener listener) {
    if (!mWifiManager.isWifiEnabled()) {
        mOnWifiEnabledListener = listener;
        mOnWifiEnabledListener.onStart(true);
        mWifiManager.setWifiEnabled(true);
    }
}
```

# 关闭WIFI

``` java
/**
 * 关闭Wifi
 */
public void closeWifi(@NonNull OnWifiEnabledListener listener) {
    if (mWifiManager.isWifiEnabled()) {
        mOnWifiEnabledListener = listener;
        mOnWifiEnabledListener.onStart(false);
        mWifiManager.setWifiEnabled(false);
    }
}
```

# 扫描附近的WIFI

``` java
/**
 * 扫描附近的WIFI
 *
 * @param listener 扫描完成的回调接口
 */
public void startScan(@NonNull OnWifiScanResultsListener listener) {
    try {
        mOnWifiScanResultsListener = listener;
        mOnWifiScanResultsListener.onStart();
        // 先返回缓存
        mOnWifiScanResultsListener.onScanResults(getScanResults());
        // 重新开始扫描
        mWifiManager.startScan();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

# 获取WIFI列表

``` java
/**
 * 获取Wifi列表
 *
 * @return Wifi列表
 */
private static List<ScanResult> getScanResults() {
    try {
        // 得到扫描结果
        return mWifiManager.getScanResults();
    } catch (Exception e) {
        e.printStackTrace();
        return null;
    }
}
```

# 通过密码连接到WIFI

``` java
/**
 * 通过密码连接到WIFI
 *
 * @param scanResult 要连接的WIFI
 * @param pwd        密码
 * @param listener   连接的监听
 */
public void connectionWifiByPassword(@NonNull ScanResult scanResult, @Nullable String pwd, @NonNull OnWifiConnectListener listener) {
    // SSID
    String SSID = scanResult.SSID;
    // 加密方式
    SecurityMode securityMode = getSecurityMode(scanResult);

    // 生成配置文件
    WifiConfiguration addConfig = createWifiConfiguration(SSID, pwd, securityMode);
    int netId;
    // 判断当前配置是否存在
    WifiConfiguration updateConfig = isExists(addConfig);
    if (null != updateConfig) {
        // 更新配置
        netId = mWifiManager.updateNetwork(updateConfig);
    } else {
        // 添加配置
        netId = mWifiManager.addNetwork(addConfig);
    }
    // 通过NetworkID连接到WIFI
    connectionWifiByNetworkId(SSID, netId, listener);
}
```

# 直接连接配置过的WIFI

``` java
/**
 * 通过NetworkId连接到WIFI （配置过的网络可以直接获取到NetworkID，从而不用再输入密码）
 *
 * @param SSID      WIFI名字
 * @param networkId NetworkId
 * @param listener  连接的监听
 */
public void connectionWifiByNetworkId(@NonNull String SSID, int networkId, @NonNull OnWifiConnectListener listener) {
    // 正要连接的SSID
    mConnectingSSID = SSID;
    // 连接的回调监听
    mOnWifiConnectListener = listener;
    // 连接开始的回调
    mOnWifiConnectListener.onStart(SSID);
    /*
     * 判断 NetworkId 是否有效
     * -1 表示配置参数不正确
     */
    if (-1 == networkId) {
        // 连接WIFI失败
        if (null != mOnWifiConnectListener) {
            // 配置错误
            mOnWifiConnectListener.onFailure();
            // 连接完成
            mOnWifiConnectListener.onFinish();
            mOnWifiConnectListener = null;
            mConnectingSSID = null;
        }
        return;
    }
    // 获取当前的网络连接
    WifiInfo wifiInfo = getConnectionInfo();
    if (null != wifiInfo) {
        // 断开当前连接
        boolean isDisconnect = disconnectWifi(wifiInfo.getNetworkId());
        if (!isDisconnect) {
            // 断开当前网络失败
            if (null != mOnWifiConnectListener) {
                // 断开当前网络失败
                mOnWifiConnectListener.onFailure();
                // 连接完成
                mOnWifiConnectListener.onFinish();
                mOnWifiConnectListener = null;
                mConnectingSSID = null;
            }
            return;
        }
    }

    // 连接WIFI
    boolean isEnable = mWifiManager.enableNetwork(networkId, true);
    if (!isEnable) {
        // 连接失败
        if (null != mOnWifiConnectListener) {
            // 连接失败
            mOnWifiConnectListener.onFailure();
            // 连接完成
            mOnWifiConnectListener.onFinish();
            mOnWifiConnectListener = null;
            mConnectingSSID = null;
        }
    }
}
```

# 断开指定WIFI

``` java
/**
 * 断开WIFI
 *
 * @param netId netId
 * @return 是否断开
 */
public boolean disconnectWifi(int netId) {
    boolean isDisable = mWifiManager.disableNetwork(netId);
    boolean isDisconnect = mWifiManager.disconnect();
    return isDisable && isDisconnect;
}
```

# 删除配置

``` java
/**
 * 删除配置
 *
 * @param netId netId
 * @return 是否删除成功
 */
private boolean deleteConfig(int netId) {
    boolean isDisable = mWifiManager.disableNetwork(netId);
    boolean isRemove = mWifiManager.removeNetwork(netId);
    boolean isSave = mWifiManager.saveConfiguration();
    return isDisable && isRemove && isSave;
}
```


# 生成配置信息

``` java
/**
 * 生成新的配置信息 用于连接Wifi
 *
 * @param SSID     WIFI名字
 * @param password WIFI密码
 * @param mode     WIFI加密类型
 * @return 配置
 */
private WifiConfiguration createWifiConfiguration(@NonNull String SSID, @Nullable String password, @NonNull SecurityMode mode) {
    WifiConfiguration config = new WifiConfiguration();
    config.allowedAuthAlgorithms.clear();
    config.allowedGroupCiphers.clear();
    config.allowedKeyManagement.clear();
    config.allowedPairwiseCiphers.clear();
    config.allowedProtocols.clear();
    config.SSID = "\"" + SSID + "\"";

    if (mode == SecurityMode.OPEN) {
        //WIFICIPHER_NOPASS
        // config.wepKeys[0] = "";
        config.allowedKeyManagement.set(WifiConfiguration.KeyMgmt.NONE);
        // config.wepTxKeyIndex = 0;
    } else if (mode == SecurityMode.WEP) {
        //WIFICIPHER_WEP
        config.hiddenSSID = true;
        config.wepKeys[0] = "\"" + password + "\"";
        config.allowedAuthAlgorithms.set(WifiConfiguration.AuthAlgorithm.SHARED);
        config.allowedGroupCiphers.set(WifiConfiguration.GroupCipher.CCMP);
        config.allowedGroupCiphers.set(WifiConfiguration.GroupCipher.TKIP);
        config.allowedGroupCiphers.set(WifiConfiguration.GroupCipher.WEP40);
        config.allowedGroupCiphers.set(WifiConfiguration.GroupCipher.WEP104);
        config.allowedKeyManagement.set(WifiConfiguration.KeyMgmt.NONE);
        config.wepTxKeyIndex = 0;
    } else if (mode == SecurityMode.WPA) {
        //WIFICIPHER_WPA
        config.preSharedKey = "\"" + password + "\"";
        config.hiddenSSID = true;
        config.allowedAuthAlgorithms.set(WifiConfiguration.AuthAlgorithm.OPEN);
        config.allowedGroupCiphers.set(WifiConfiguration.GroupCipher.TKIP);
        config.allowedKeyManagement.set(WifiConfiguration.KeyMgmt.WPA_PSK);
        config.allowedPairwiseCiphers.set(WifiConfiguration.PairwiseCipher.TKIP);
        //config.allowedProtocols.set(WifiConfiguration.Protocol.WPA);
        config.allowedGroupCiphers.set(WifiConfiguration.GroupCipher.CCMP);
        config.allowedPairwiseCiphers.set(WifiConfiguration.PairwiseCipher.CCMP);
        config.status = WifiConfiguration.Status.ENABLED;
    }
    return config;
}
```


# 目前存在的问题
> 目前存在的问题是，当删除配置以后，再连接WIFI，调用`addNetwork`或者`updateNetwork`都会返回`-1`，需要重启一下WIFI才会正常，有知道原因还请指点一下。

