# AppLoveFrom SDK Integration

1. [Introduction](#introduction)

3. [Integration AppLoveFrom SDK](#integration)

    [2.1 Integrating the AppLoveFrom SDK in to project](#step1)

    [2.2 Initialize the AppLoveFrom SDK](#step2)

    [2.3 Android code obfuscation](#step3)

4. [Integration Notes](#note)

4. [Request Ad](#request)

    [3.1 Native](#native)

    * [Element-Native](#common)
    * [Element-Native with adCache](#cache)
    * [Element-Native for Multiple](#multi)

    [3.2 Banner](#banner)

    [3.3 Interstitial](#interstitial)

    [3.4 Appwall](#appwall)

    [3.5 Rewarded Video](#reward)

    [3.6 Native Video](#native_video)

5. [Error Code For SDK](#error)

## <a name="introduction">1.Introduction</a>

- AppLoveFrom SDK supports Banner, Interstitial, Native, Native Video andRewarded Video.
- AppLoveFrom Android SDK supports Android API 15+.
- Please make sure you have added an app and at least one ad slot in AppLoveFrom Platform.

## <a name="integration">2.Integration AppLoveFrom SDK</a>  

### <a name="step1">2.1 Integrating the AppLoveFrom SDK in to project</a>

* Check SDK in the rar.

* Detail of the different jars：

  | jar name                 | jar function                                    | require(Y/N) |
  | ------------------------ | ----------------------------------------------- | ------------ |
  | applovefrom_base_xx.jar     | basic functions(banner\interstitial\native ads) | Y            |
  | applovefrom_imageloader_xx.jar | imageloader functions                           | N            |
  | applovefrom_appwall_xx.jar  | appwall ads functions                           | N            |
  | applovefrom_video_xx.jar    | video ads functions                             | N            |

* Configure the module's build.gradle for basic functions：

``` groovy
    dependencies {
        compile files('libs/applovefrom_base_xx.jar')
    }
```

* Configure AndroidManifest.xml

```xml
	<!--Necessary Permissions-->
	<uses-permission android:name="android.permission.INTERNET"/>
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

	<!-- Necessary -->
	<activity android:name="com.applovefrom.base.view.InnerWebViewActivity" />

	<provider
            android:authorities="${applicationId}.xxprovider"
            android:name="com.applovefrom.base.core.AppLoveFromProvider"
            android:exported="false"/>
	

	<!--If your targetSdkVersion is 28, you need update <application> as follows:-->
  	<application
    	...  	
        android:usesCleartextTraffic="true"
        ...>
        ...
    </application>
```



## <a name="step2">2.2 Initialization</a>  

> Init the SDK in your Application as detailed below: 

**Please make sure to initialize AppLoveFrom SDK before doing anything.**

```java
    AppLoveFromSDK.initialize(context, "Your slotID");
```

**Use this interface to upload consent from affected users for GDPR**

```java
    /**
     * @param context       context
     * @param consentValue  whether the user agrees
     * @param consentType   the agreement name signed with users
     * @param listener      callback
     */
     AppLoveFromSDK.uploadConsent(this, true, "GDPR", new HttpRequester.Listener() {
            @Override
            public void onGetDataSucceed(byte[] data) {
            }


            @Override
            public void onGetDataFailed(String error) {
            }
     });
```
**Set schema https**
```java
      AppLoveFromSDK.setSchema(true);
```

## <a name="step3">2.3 Obfuscation Configuration</a> 
> If it needs to obfuscate the codes in building the project process, you should add the following codes into the proguard file:

``` java
    #for sdk
    -keep public class com.applovefrom.**{*;}
    -dontwarn com.applovefrom.**

    #for gaid
    -keep class **.AdvertisingIdClient$** { *; }

    #for js and webview interface
    -keepclassmembers class * {
        @android.webkit.JavascriptInterface <methods>;
    }
    
```


## <a name="note">3.Integration Notes</a>

​	If you live in a country, such as China, which is forbidden google play, two prerequisites to get AppLoveFrom ads: 
> * GooglePlay has installed on your device.
> * Your device have connect to VPN.

   

​	We suggest you define a class to implement the CTAdEventListener yourself , then you can just override the methods you need when you getBanner or getNative. See the following example:

``` java
public class MyCTAdEventListener extends CTAdEventListener {
    @Override
    public void onReceiveAdSucceed(AppLoveFromNative result) {
    }

    @Override
    public void onReceiveAdVoSucceed(AdsNativeVO result) {
    }

    @Override
    public void onInterstitialLoadSucceed(AppLoveFromNative result) {
    }

    @Override
    public void onReceiveAdFailed(AppLoveFromNative result) {
        Log.i("sdksample", "==error==" + result.getErrorsMsg());
    }

    @Override
    public void onLandpageShown(AppLoveFromNative result) {
    }

    @Override
    public void onAdClicked(AppLoveFromNative result) {
    }

    @Override
    public void onAdClosed(AppLoveFromNative result) {
    }
}
```

## <a name="native">4.1 Native Ads Integration</a>

### <a name="common">Native ads interface</a>

> The container and the layout for Native ad:

```java
    ViewGroup container = (ViewGroup) view.findViewById(R.id.container);
    ViewGroup adLayout = (ViewGroup)View.inflate(context,R.layout.native_layout, null);
```

> The method to load Native Ads:

``` java
    /**
     * @param slotId     applovefrom id
     * @param context    context
     * @param adListener callback listener 
     */
 	AppLoveFromSDK.getNativeAd("Your slotID", context, new MyCTAdEventListener(){
        @Override
        public void onReceiveAdSucceed(AppLoveFromNative result) {
            if (result == null) {
                return;
            }
            AppLoveFromAdvanceNative alfAdvanceNative = (AppLoveFromAdvanceNative) result;
            showAd(alfAdvanceNative);
            Log.e(TAG, "onReceiveAdSucceed");
            super.onReceiveAdSucceed(result);
        }
     });
```

* Show the Native ad

``` java
   private void showAd(AppLoveFromAdvanceNative alfAdvanceNative) {
        ImageView img = (ImageView) adLayout.findViewById(R.id.iv_img);
        ImageView icon = (ImageView) adLayout.findViewById(R.id.iv_icon);
        TextView title = (TextView)adLayout.findViewById(R.id.tv_title);
        TextView desc = (TextView)adLayout.findViewById(R.id.tv_desc);
        Button click = (Button)adLayout.findViewById(R.id.bt_click);
        ImageView adChoice = (ImageView)adLayout.findViewById(R.id.choice_icon);
        
        //show the image and icon yourself 
        String imageUrl = alfAdvanceNative.getImageUrl();
        String iconUrl = alfAdvanceNative.getIconUrl();          
        title.setText(alfAdvanceNative.getTitle());
        desc.setText(alfAdvanceNative.getDesc());
        click.setText(alfAdvanceNative.getButtonStr());
        adChoice.setImageURI(alfAdvanceNative.getAdChoiceIconUrl());
        //offerType（1 : download ads; 2 : content ads）
        int offerType = alfAdvanceNative.getOfferType();  
         
        alfAdvanceNative.registeADClickArea(adLayout);
        container.addView(adLayout);
   }
```



### <a name="cache">Native ads interface for AdCache</a>

> Get Ads for cache

```java
    /**
     * @param slotId     applovefrom id
     * @param context    context
     * @param adListener callback listener 
     */
    AppLoveFromSDK.getNativeAdForCache("Your slotID",context,new MyCTAdEventListener() {
        @Override
        public void onReceiveAdVoSucceed(AdsNativeVO result) {
            if (result == null) {
                return;
            }
            Log.e(TAG,"onReceiveAdVoSucceed");
            AdHolder.adNativeVO = result;
            super.onReceiveAdVoSucceed(result);
        }
    });

```

> Show ad from cache

```java
      AppLoveFromAdvanceNative alfAdvanceNative = new 	AppLoveFromAdvanceNative(getContext());
      AdsNativeVO nativeVO = AdHolder.adNativeVO;

	  if (nativeVO != null) {
            alfAdvanceNative.setNativeVO(nativeVO);
            alfAdvanceNative.setSecondAdEventListener(new MyCTAdEventListener() {
                @Override
                public void onAdClicked(com.applovefrom.base.core.AppLoveFromNative result) {
                    Log.e(TAG, "onAdClicked");
                    super.onAdClicked(result);
                }
            });
            showAd(alfAdvanceNative);
       }
```



### <a name="multi">Multi Native ad interface</a>

> The method to load multi Native ad

``` java
    /**
     * @param reqAdNumber request ads num
     * @param slotId      applovefrom id
     * @param context     context
     * @param adListener  callback listener 
     */
	AppLoveFromSDK.getNativeAds(10, "Your slotID", getContext(), new MultiAdsEventListener() {
        public void onMultiNativeAdsSuccessful(List<AppLoveFromAdvanceNative> res) {
        }

        @Override
        public void onReceiveAdFailed(AppLoveFromNative result) {
        }

        @Override
        public void onAdClicked(AppLoveFromNative result) {
            super.onAdClicked(result);
        }
    });
```



## <a name="banner">4.2 Banner Ad Integration</a>

> The method to load Banner Ad:

``` java
	ViewGroup container = (ViewGroup) view.findViewById(R.id.container);

    /**
     * @param context           context
     * @param slotId            applovefrom id
     * @param adSize			AdSize.AD_SIZE_320X50,
     							AdSize.AD_SIZE_320X100,
     							AdSize.AD_SIZE_300X250;
     * @param adListener        callback listener 
     */
	 AppLoveFromSDK.getBannerAd(context, "Your slotID", adSize,new MyCTAdEventListener() {
         @Override
         public void onReceiveAdSucceed(AppLoveFromNative result) {
             if (result != null) {
                 container.setVisibility(View.VISIBLE);
                 container.removeAllViews();
                 container.addView(result);   //把广告添加到容器
             }
             super.onReceiveAdSucceed(result);
         }


         @Override
         public void onReceiveAdFailed(AppLoveFromNative result) {
             super.onReceiveAdFailed(result);
         }


         @Override
         public void onAdClicked(AppLoveFromNative result) {
             super.onAdClicked(result);
         }


         @Override
         public void onAdClosed(AppLoveFromNative result) {
             container.removeAllViews();
             container.setVisibility(View.GONE);
             super.onAdClosed(result);
         }
     });
```

> When you successfully integrated the Banner Ad, you will see the ads are like this


![-1](https://user-images.githubusercontent.com/20314643/42366029-b6289f2a-8132-11e8-9c3e-86557d164d85.png)
![320x100](https://user-images.githubusercontent.com/20314643/42370991-c4188812-8140-11e8-80e9-ab6947c12e92.png)
![300x250](https://user-images.githubusercontent.com/20314643/42370999-c74139f8-8140-11e8-91ff-ba0cdb0ae08a.png)



## <a name="interstitial">4.3 Interstitial Ads Integration</a>

> Configure the AndroidManifest.xml for Interstitial

```xml
	<activity android:name="com.applovefrom.base.view.InterstitialActivity" />    
```

> The method to show Interstitial Ads

``` java
    /**
     * @param context           context
     * @param slotId            applovefrom id
     * @param adListener        callback listener 
     */
    AppLoveFromSDK.preloadInterstitialAd(context, "Your slotID",new MyCTAdEventListener() {
                    
        @Override
        public void onReceiveAdSucceed(AppLoveFromNative result) {              
            if (result != null && result.isLoaded()) {
                Log.w(TAG, "onReceiveAdSucceed");
                if (AppLoveFromSDK.isInterstitialAvailable(result)) {
            		AppLoveFromSDK.showInterstitialAd(result);
        		}
            }
            super.onReceiveAdSucceed(result);
        }

        @Override 
        public void onLandpageShown(AppLoveFromNative result) {
            super.onLandpageShown(result);
            Log.e(TAG, "onLandpageShown:");
        }


        @Override
        public void onReceiveAdFailed(AppLoveFromNative error) {
            Log.w(TAG, "onReceiveAdFailed: " + error.getErrorsMsg());
            super.onReceiveAdFailed(error);
        }


        @Override
        public void onAdClosed(AppLoveFromNative result) {
            super.onAdClosed(result);
            Log.e(TAG, "onAdClosed:");
        }
    });
```

> When you successfully integrated the Interstitial Ad, you will see the ads are like this

![image](https://user-images.githubusercontent.com/20314643/41895879-b4536200-7955-11e8-9847-587f175c4a54.png)
![image](https://user-images.githubusercontent.com/20314643/41895941-e0c6ad1a-7955-11e8-9393-ed91e4a4906f.png)



## <a name="appwall">4.4 Appwall integration</a>

> Configure the module's build.gradle for Appwall：

``` groovy
	dependencies {
        	compile files('libs/applovefrom_base_xx.jar')
        	compile files('libs/applovefrom_appwall_xx.jar')       // for appwall        
        	compile files('libs/applovefrom_imageloader_xx.jar')   // for imageloader
	}
```

> Configure the AndroidManifest.xml for Appwall

``` xml
    <activity
            android:name="com.applovefrom.appwall.AppwallActivity"
            android:screenOrientation="portrait" />  
```

> Preload appwall

> It‘s better to preload ads for Appwall, to ensure they show properly and in a timely fashion. You can have ads preload with the following line of code:

``` java
   AppLoveFromAppwall.preloadAppwall(context, "Your slotID");
```

> Customize the appwall color theme(optional).

``` java
    CustomizeColor custimozeColor = new CustomizeColor();
    custimozeColor.setMainThemeColor(Color.parseColor("#ff0000ff"));
    AppLoveFromAppwall.setThemeColor(custimozeColor);
```

> Show Appwall.

``` java
     AppLoveFromAppwall.showAppwall(context, "Your slotID");
```

> When you successfully integrated the APP Wall Ad, you will see the ads are like this

![image](https://user-images.githubusercontent.com/20314643/42366246-47526c9c-8133-11e8-963c-bd0eb7a3e1a6.png)



## <a name="reward">4.5 Rewarded Video Ad Integration</a>

> **Google Play Services**

1. Google Advertising ID

 The Rewarded Video function requires access to the Google Advertising ID in order to operate properly.
     See this guide on how to integrate [Google Play Services](https://developers.google.com/android/guides/setup).

2. Google Play Services in Your Android Manifest

    Add the following  inside the <application> tag in your AndroidManifest:

    ```
    <meta-data
    	android:name="com.google.android.gms.ads.AD_MANAGER_APP"
        android:value="true" />
    ```

3. If you have integrated the admob-sdk for basic ads, it's not necessary to do this.


> Configure the module's build.gradle for Rewarded Video：

``` groovy
	dependencies {
        	compile files('libs/applovefrom_base_xx.jar')
        	compile files('libs/applovefrom_video_xx.jar')
        	compile files('libs/applovefrom_imageloader_xx.jar')
	}
```

> Configure the AndroidManifest.xml for Rewarded Video:

``` xml    
<activity
	android:name="com.applovefrom.video.view.RewardedVideoActivity"           android:configChanges="keyboard|keyboardHidden|orientation|screenLayout|uiMode|screenSize|smallestScreenSize" />
```

> Setup the UserID

> You should set the userID for s2s-postback before preloading the RewardedVideo function call


```java
	AppLoveFromVideo.setUserId("custom_id");
```

> Preload the RewardedVideo

> It‘s best to call this interface when you want to show RewardedVideo ads to the user. This will help reduce latency and ensure effective, prompt delivery of ads


``` java
 	AppLoveFromVideo.preloadRewardedVideo(getContext(), Config.slotIdReward,
                new VideoAdLoadListener() {
                    @Override
                    public void onVideoAdLoadSucceed(AppLoveFromVideo videoAd) {
                        video = videoAd;
                        Log.w(TAG, "onVideoAdLoadSucceed: ");
                    }


                    @Override
                    public void onVideoAdLoadFailed(AppLoveFromError error) {
                        Log.w(TAG, "onVideoAdLoadFailed: " + error.getMsg());
                    }
                });

```

> Show Rewarded Video ads to Your Users
> Before showing the video, you can request or query the video status by calling:

```java
    boolean available = AppLoveFromVideo.isRewardedVideoAvailable(video);
```

> Because it takes a while to load the video creatives. You may need to wait until video creatives loads completely. Once you get an available Reward Video, you are ready to show this video ad to your users by calling the showRewardedVideo() method as in the following example:

```java
  AppLoveFromVideo.showRewardedVideo(video, new RewardedVideoAdListener() {
                @Override
                public void videoStart() {
                    Log.e(TAG, "videoStart: ");
                }

                @Override
                public void videoFinish() {
                    Log.e(TAG, "videoFinish: ");
                }

                @Override
                public void videoError(Exception e) {
                    Log.e(TAG, "videoError: ");
                }

                @Override
                public void videoClosed() {
                    Log.e(TAG, "videoClosed: ");
                }

                @Override
                public void videoClicked() {
                    Log.e(TAG, "videoClicked: ");
                }

                @Override
                public void videoRewarded(String rewardName, String rewardAmount) {
                    Log.e(TAG, "videoRewarded: ");
                }
            });

```

> Reward the User

The SDK will fire the videoRewarded event each time the user successfully completes a video. The RewardedVideoListener will be in place to receive this event so you can reward the user successfully.

The Reward object contains both the Reward Name & Reward Amount of the SlotId as defined in your AppLoveFrom SSP:

```java

public void videoRewarded(String rewardName, String rewardAmount) {
      //TODO - here you can reward the user according to the given amount
       Log.e(TAG, "videoRewarded: ");
   }
    
```

> When you successfully integrated the Rewarded Video, you will see the ads are like this

![image](https://user-images.githubusercontent.com/20314643/42371626-94e8e6a2-8142-11e8-9598-eb75de753070.png)


## <a name="native_video">4.6 Native Video Ad Integration</a>

> Configure the module's build.gradle for Native Video：

```groovy
	dependencies {
        compile files('libs/applovefrom_base_xx.jar')
        compile files('libs/applovefrom_video_xx.jar')
	}
```

> Load the NativeVideo

```java
   AppLoveFromVideo.getNativeVideo(context, Config.slotIdNativeVideo, new VideoAdLoadListener(){
            @Override
            public void onVideoAdLoadSucceed(AppLoveFromVideo video) {
                showNativeVideo((AppLoveFromNativeVideo) video);
            }


            @Override
            public void onVideoAdLoadFailed(AppLoveFromError error) {
            }
        });

```

> Show the NativeVideo

```java
    private void showNativeVideo(AppLoveFromNativeVideo alfNativeVideo) {
        if (video == null) {
            return;
        }
        
        //layout for NativeVideo
        ViewGroup videoLayout = (ViewGroup) View.inflate(context, R.layout.native_video_layout, null);
      
        //whether autoplay for 4G
        alfNativeVideo.setWWANPlayEnabled(false);

        TextView video_title = videoLayout.findViewById(R.id.video_title);
        RelativeLayout video_container = videoLayout.findViewById(R.id.video_container);
        SimpleDraweeView video_choice = videoLayout.findViewById(R.id.video_choice);
        TextView video_desc = videoLayout.findViewById(R.id.video_desc);
        TextView video_button = videoLayout.findViewById(R.id.video_button);

        video_title.setText(alfNativeVideo.getTitle());
        video_container.addView(alfNativeVideo.getNativeVideoAdView());
        video_choice.setImageURI(Uri.parse(alfNativeVideo.getAdChoiceIconUrl()));
        video_desc.setText(Html.fromHtml(alfNativeVideo.getDesc()));
        video_button.setText(alfNativeVideo.getButtonStr());

        //register for tracking， videolayout is the clickarea
        alfNativeVideo.registerForTracking(videoLayout, 
            new NativeVideoAdListener() {
                @Override
                public void videoPlayBegin() {
                }

                @Override
                public void videoPlayFinished() {
                }

                @Override
                public void videoPlayClicked() {
                }

                @Override
                public void videoPlayError(Exception e) {
                }
        });

        //container in your app
        rl_container.removeAllViews();
        rl_container.addView(videoLayout);
    }
```
## <a name="error">Error Code For SDK</a>

| Error Code                        | Description                              |
| --------------------------------- | ---------------------------------------- |
| ERR\_000\_TRACK                   | Track exception                          |
| ERR\_001\_INVALID_INPUT           | Invalid parameter                        |
| ERR\_002\_NETWORK                 | Network exception                        |
| ERR\_003\_REAL_API                | Error from Ad Server                     |
| ERR\_004\_INVALID_DATA            | Invalid advertisement data               |
| ERR\_005\_RENDER_FAIL             | Advertisement render failed              |
| ERR\_006\_LANDING_URL             | Landing URL failed                       |
| ERR\_007\_TO_DEFAULT_MARKET       | Default Landing URL failed               |
| ERR\_008\_DL_URL                  | Deep-Link exception                      |
| ERR\_009\_DL_URL_JUMP             | Deep-Link jump exception                 |
| ERR\_010\_APK_DOWNLOAD_URL        | Application package download failed      |
| ERR\_011\_APK_INSTALL             | Application install failed               |
| ERR\_012\_VIDEO_LOAD              | Load the video exception                 |
| ERR\_013\_PAGE_LOAD               | HTML5 page load failed                   |
| ERR\_014\_JAR_UPDATE_VERSION      | Update jar check failed                  |
| ERR\_015\_GET_GAID                | Cannot get google advertisement id - GAID retrieval failed |
| ERR\_016\_GET_AD_CONFIG           | Cannot get the account configuration or template |
| ERR\_017\_INTERSTITIAL_SHOW_NO_AD | Tried to load the interstitial advertisement, but the advertisement is not ready/loading |
| ERR\_018\_AD_CLOSED               | Ad slotId has been closed                |
| ERR\_999\_OTHERS                  | All other errors                         |


