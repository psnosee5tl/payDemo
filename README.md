使用微信支付的正确姿势（附带Android Studio Debug使用签名包配置）

1 首先keystore必须和微信开放平台的签名信息一样

2 第一次请求微信参数问题 ↓ 不能有空的字段 否则这一步请求就出错！
 List<NameValuePair> packageParams = new LinkedList<>();
  packageParams.add(new BasicNameValuePair("appid", "你的appID"));
  packageParams.add(new BasicNameValuePair("body", "1"));
  packageParams.add(new BasicNameValuePair("detail", "123"));
  packageParams.add(new BasicNameValuePair("mch_id", "商户ID"));
  packageParams.add(new BasicNameValuePair("nonce_str", nonceStrs));
  packageParams.add(new BasicNameValuePair("notify_url", "回调网址 随意填写不能为空"));
  packageParams.add(new BasicNameValuePair("out_trade_no", pOutTradeNo + getOutTradeNo()));
  packageParams.add(new BasicNameValuePair("spbill_create_ip", "127.0.0.1"));
  packageParams.add(new BasicNameValuePair("total_fee", "测试的时候最好别写0.01 填1"));
  packageParams.add(new BasicNameValuePair("trade_type", "APP"));
  packageParams.add(new BasicNameValuePair("sign", genPackageSign(packageParams)));
  String xmlstring = toXml(packageParams);
  return new String(xmlstring.toString().getBytes(), "ISO-8859-1");

3 接收微信返回参数，并向微信发起第二次请求
req.appId = Constants.APP_ID;
req.partnerId = Constants.MCH_ID;
req.prepayId = resultunifiedorder.get("prepay_id");
req.packageValue = "Sign=WXPay";
req.nonceStr = genNonceStr();
req.timeStamp = String.valueOf(genTimeStamp());
List<NameValuePair> signParams = new LinkedList<>();
signParams.add(new BasicNameValuePair("appid", req.appId));
signParams.add(new BasicNameValuePair("noncestr", req.nonceStr));
signParams.add(new BasicNameValuePair("package", req.packageValue));
signParams.add(new BasicNameValuePair("partnerid", req.partnerId));
signParams.add(new BasicNameValuePair("prepayid", req.prepayId));
signParams.add(new BasicNameValuePair("timestamp", req.timeStamp));
req.sign = genAppSign(signParams);// 签名
这里必须注意 签名参数的顺序 
顺序一定按照这个来 否则一直出现 -1返回码！

4 AndroidManifest.xml 里的配置
 <activity
            android:name=".wxapi.WXPayEntryActivity"
            android:exported="true"
            android:launchMode="singleTop">
 </activity>
这里并不需要以下的代码 ↓
 <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <data android:scheme="wxc4c5efc62c943111" />
</intent-filter>

5 一定要注意 包名和签名是否和微信开放平台的一致

6  WXAPIFactory.createWXAPI(this, Constants.APP_ID)
 这里无论是填写 this,app_id 还是 this,null  无所谓！没有影响！

7 天下无坑！

8 彩蛋
apply plugin: 'com.android.application'

android {
    compileSdkVersion ..//
    buildToolsVersion ..//
    defaultConfig {
       ..//
    }
    signingConfigs {
        release {
          storeFile file(keystore地址:"C:/Users/Administrator/Desktop/key/yisheng")
            storePassword "123456"
            keyAlias "yh"
            keyPassword "123456"
        }
    }
    buildTypes {
        release {
            //启用混淆配置
            minifyEnabled true
            //Zip代码压缩优化
            zipAlignEnabled true
            //移除无用资源
            shrinkResources true
            //加载默认混淆文件
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
        }
        debug {
            //Zip代码压缩优化
            zipAlignEnabled true
            //移除无用资源
            shrinkResources true
            //debugs使用release签名
            signingConfig signingConfigs.release
        }
    }
}



dependencies {
    ..//
    
}


