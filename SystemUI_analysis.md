
#SystemUI 分析
@[Android, aosp, SystemUI]

[TOC]
## SystemUI 文件分析
###  SystemUI的界面
SystemUI的界面：
	状态栏， 导航栏，锁屏界面， 音量改变界面， 手机连接电脑授权提示框， 截屏服务， 最近打开应用， 亮度调节， 下拉栏， 搜索， 流量使用信息弹出框, 通知, 锁屏 ,低电量提示, 铃声播放. 这些都可以从SystemUI里面的包名以及文件可以看得出来

### SystemUI的makefile文件
同时看看SystemUI的Makefile文件
```makefile
LOCAL_PATH:= $(call my-dir)  #编译的路径, 即当前路径下
include $(CLEAR_VARS)   #编译一个模块的开始, 清除以前开始设置的环境变量

LOCAL_MODULE_TAGS := optional

LOCAL_SRC_FILES := $(call all-java-files-under, src) \
    src/com/android/systemui/EventLogTags.logtags

LOCAL_STATIC_JAVA_LIBRARIES := Keyguard
LOCAL_JAVA_LIBRARIES := telephony-common

LOCAL_PACKAGE_NAME := SystemUI  #编译的模块名
LOCAL_CERTIFICATE := platform
LOCAL_PRIVILEGED_MODULE := true #添加次属性的编译路径为/system/priv-app/SystemUI/ 在同样的系统权限下, priv-app下的apk的权限比system/app下的要大

LOCAL_PROGUARD_FLAG_FILES := proguard.flags

LOCAL_RESOURCE_DIR := \
    frameworks/base/packages/Keyguard/res \
    $(LOCAL_PATH)/res
# aapt打包命令的编译参数, --auto-add-overlay 参数表示overlay下可以新增资源文件.
LOCAL_AAPT_FLAGS := --auto-add-overlay --extra-packages com.android.keyguard

#使用增量编译,
ifneq ($(SYSTEM_UI_INCREMENTAL_BUILDS),)
    LOCAL_PROGUARD_ENABLED := disabled
    LOCAL_JACK_ENABLED := incremental
endif

include frameworks/base/packages/SettingsLib/common.mk

include $(BUILD_PACKAGE) #

ifeq ($(EXCLUDE_SYSTEMUI_TESTS),)
    include $(call all-makefiles-under,$(LOCAL_PATH))
endif

```


### SystemUI的清单文件
```xml
	android:name=".SystemUIApplication"
	android:allowBackup="false"
	android:allowClearUserData="false"
	android:hardwareAccelerated="true"
	android:icon="@drawable/icon"
	android:label="@string/app_label"
	android:persistent="true" 
	android:process="com.android.systemui" 
	android:supportsRtl="true"
	android:theme="@style/systemui_theme">
```
android:name=".SystemUIApplication" 为这个应用实践改的Application子类限定名称. 当应用启动时, 这个类将在应用的其他组件之前实例化.
android:allowBackup="false" 允许备份
android:allowClearUserData="false"  是否给用户删除用户数据的权限.
 android:persistent="true" 
 android:process="com.android.systemui"  应用的进程名每个组件可以定义自己的进程的名称, 默认情况下进程的名称和包名相同
 android:supportsRtl="false" ， 如果Service等设置为false， 表示该服务或者activity不能跨进程使用。设置为true表示可以为其他应用调用

 
以下是SystemUI的清单文件的连接[SystemUI](https://github.com/android/platform_frameworks_base/blob/master/packages/SystemUI/AndroidManifest.xml)
 对其中的一些的参数进行说明
 android:exported="false"  

//底层全部弄好, 设置那个指纹选项出来.

SystemUI下的文件结构
```
├── assist
│   ├── AssistDisclosure.java
│   ├── AssistManager.java
│   ├── AssistOrbContainer.java
│   └── AssistOrbView.java
├── BatteryMeterView.java
├── BitmapHelper.java
├── BootReceiver.java   //用来接收启动信息的
├── DejankUtils.java
├── DemoMode.java
├── DessertCaseDream.java
├── DessertCase.java
├── DessertCaseView.java
├── doze   //android 6.0 新增的省电模式
│   ├── DozeHost.java
│   ├── DozeLog.java
│   └── DozeService.java
├── egg   //android的彩蛋, M表示android表示版本号, android的内置游戏,长按android的版本号就可以出现
│   ├── MLandActivity.java
│   └── MLand.java
├── EventLogConstants.java
├── EventLogTags.logtags
├── ExpandHelper.java
├── FontSizeUtils.java
├── Gefingerpoken.java
├── GuestResumeSessionReceiver.java
├── ImageWallpaper.java
├── keyboard
│   ├── BluetoothDialog.java  //蓝牙的dialog
│   └── KeyboardUI.java
├── keyguard //keyguard就是手机亮屏，但是还未解锁状态，这时上面的通知也是SystemUI的
│   ├── KeyguardService.java
│   └── KeyguardViewMediator.java
├── LoadAverageService.java
├── media  //通知的声音， 铃声播放
│   ├── MediaProjectionPermissionActivity.java
│   ├── NotificationPlayer.java
│   └── RingtonePlayer.java
├── net
│   └── NetworkOverLimitActivity.java
├── power  //手机低电量警告, 以及长按电源键的关机
│   ├── PowerNotificationWarnings.java
│   └── PowerUI.java
├── Prefs.java
├── qs  //这些就是下拉视图， qs 是quick Setting的意思
│   ├── DataUsageGraph.java  
│   ├── GlobalSetting.java   //下拉视图出现的设置按钮
│   ├── PseudoGridView.java
│   ├── QSContainer.java
│   ├── QSDetailClipper.java
│   ├── QSDetailItems.java
│   ├── QSDualTileLabel.java
│   ├── QSFooter.java
│   ├── QSPanel.java    
│   ├── QSTile.java
│   ├── QSTileView.java
│   ├── SecureSetting.java
│   ├── SignalTileView.java
│   ├── tiles    //这文件夹里的东西就是下拉视图的快速设置项
│   │   ├── AirplaneModeTile.java  //飞行模式
│   │   ├── BluetoothTile.java    //蓝牙
│   │   ├── CastTile.java    //castscreen android投射屏幕到电视的
│   │   ├── CellularTile.java      //Cellular项,信号
│   │   ├── ColorInversionTile.java   //颜色反转， 夜间模式
│   │   ├── DataUsageDetailView.java   //流量使用情况
│   │   ├── DndTile.java      //Do not disturb 免打扰模式, 情景模式
│   │   ├── FlashlightTile.java     //闪光灯
│   │   ├── HotspotTile.java      //热点
│   │   ├── IntentTile.java
│   │   ├── LocationTile.java     //位置开关
│   │   ├── RotationLockTile.java   //自动旋转
│   │   ├── UserDetailItemView.java   //android 6.0 是多用户, 点击上面的账户按钮, 会出现账户列表, 下同
│   │   ├── UserDetailView.java
│   │   └── WifiTile.java      //wifi设置
│   └── UsageTracker.java
├── recents // 按下recent键是用的东西，
│   ├── Constants.java
│   ├── misc
│   │   ├── Console.java
│   │   ├── DebugTrigger.java
│   │   ├── DozeTrigger.java
│   │   ├── NamedCounter.java
│   │   ├── ReferenceCountedTrigger.java
│   │   ├── SystemServicesProxy.java
│   │   └── Utilities.java
│   ├── model
│   │   ├── BitmapLruCache.java
│   │   ├── DrawableLruCache.java
│   │   ├── KeyStoreLruCache.java
│   │   ├── RecentsPackageMonitor.java
│   │   ├── RecentsTaskLoader.java
│   │   ├── RecentsTaskLoadPlan.java
│   │   ├── StringLruCache.java
│   │   ├── TaskGrouping.java
│   │   ├── Task.java
│   │   └── TaskStack.java
│   ├── RecentsActivity.java
│   ├── RecentsAppWidgetHost.java
│   ├── RecentsAppWidgetHostView.java
│   ├── RecentsConfiguration.java
│   ├── Recents.java
│   ├── RecentsResizeTaskDialog.java
│   ├── RecentsUserEventProxyReceiver.java
│   ├── ScreenPinningRequest.java
│   └── views
│       ├── AnimateableViewBounds.java
│       ├── DebugOverlayView.java
│       ├── FakeShadowDrawable.java
│       ├── FixedSizeImageView.java
│       ├── RecentsView.java
│       ├── RecentsViewLayoutAlgorithm.java
│       ├── SwipeHelper.java
│       ├── SystemBarScrimViews.java
│       ├── TaskStackViewFilterAlgorithm.java
│       ├── TaskStackView.java
│       ├── TaskStackViewLayoutAlgorithm.java
│       ├── TaskStackViewScroller.java
│       ├── TaskStackViewTouchHandler.java
│       ├── TaskViewHeader.java
│       ├── TaskView.java
│       ├── TaskViewThumbnail.java
│       ├── TaskViewTransform.java
│       ├── ViewAnimation.java
│       └── ViewPool.java
├── RecentsComponent.java
├── screenshot  //和截屏有关的
│   ├── GlobalScreenshot.java
│   └── TakeScreenshotService.java
├── settings    //屏幕亮度有关的
│   ├── BrightnessController.java
│   ├── BrightnessDialog.java
│   ├── CurrentUserTracker.java
│   ├── ToggleSeekBar.java
│   └── ToggleSlider.java
├── Somnambulator.java
├── statusbar   //和状态栏有关系的
│   ├── ActivatableNotificationView.java
│   ├── AlphaImageView.java
│   ├── AlphaOptimizedButton.java
│   ├── AlphaOptimizedFrameLayout.java
│   ├── AlphaOptimizedImageView.java
│   ├── AlphaOptimizedView.java
│   ├── AnimatedImageView.java
│   ├── BackDropView.java
│   ├── BaseStatusBar.java
│   ├── CommandQueue.java
│   ├── DismissViewButton.java
│   ├── DismissView.java
│   ├── DragDownHelper.java
│   ├── EmptyShadeView.java
│   ├── ExpandableNotificationRow.java
│   ├── ExpandableOutlineView.java
│   ├── ExpandableView.java
│   ├── FlingAnimationUtils.java
│   ├── GestureRecorder.java
│   ├── KeyguardAffordanceView.java
│   ├── KeyguardIndicationController.java
│   ├── NotificationBackgroundView.java
│   ├── NotificationBigMediaNarrowViewWrapper.java
│   ├── NotificationBigPictureViewWrapper.java
│   ├── NotificationContentView.java
│   ├── NotificationCustomViewWrapper.java
│   ├── NotificationData.java
│   ├── NotificationGuts.java
│   ├── NotificationMediaViewWrapper.java
│   ├── NotificationOverflowContainer.java
│   ├── NotificationOverflowIconsView.java
│   ├── NotificationTemplateViewWrapper.java
│   ├── NotificationViewWrapper.java
│   ├── phone     // 和手机有关的， 以及导航栏有关的, keyguard
│   │   ├── ActivityStarter.java
│   │   ├── BarTransitions.java
│   │   ├── BounceInterpolator.java
│   │   ├── DemoStatusIcons.java
│   │   ├── DozeParameters.java
│   │   ├── DozeScrimController.java
│   │   ├── FingerprintUnlockController.java
│   │   ├── HeadsUpTouchHelper.java
│   │   ├── IconMerger.java
│   │   ├── KeyguardAffordanceHelper.java
│   │   ├── KeyguardBottomAreaView.java
│   │   ├── KeyguardBouncer.java
│   │   ├── KeyguardClockPositionAlgorithm.java
│   │   ├── KeyguardIndicationTextView.java
│   │   ├── KeyguardPreviewContainer.java
│   │   ├── KeyguardStatusBarView.java
│   │   ├── LockIcon.java
│   │   ├── MultiUserSwitch.java
│   │   ├── NavigationBarTransitions.java
│   │   ├── NavigationBarView.java
│   │   ├── NavigationBarViewTaskSwitchHelper.java
│   │   ├── NoisyVelocityTracker.java
│   │   ├── NotificationGroupManager.java
│   │   ├── NotificationPanelView.java
│   │   ├── NotificationsQuickSettingsContainer.java
│   │   ├── ObservableScrollView.java
│   │   ├── PanelBar.java
│   │   ├── PanelHolder.java
│   │   ├── PanelView.java
│   │   ├── PhoneStatusBar.java
│   │   ├── PhoneStatusBarPolicy.java
│   │   ├── PhoneStatusBarTransitions.java
│   │   ├── PhoneStatusBarView.java
│   │   ├── PlatformVelocityTracker.java
│   │   ├── QSTileHost.java
│   │   ├── ScrimController.java
│   │   ├── SettingsButton.java
│   │   ├── StatusBarHeaderView.java
│   │   ├── StatusBarIconController.java
│   │   ├── StatusBarKeyguardViewManager.java
│   │   ├── StatusBarWindowManager.java
│   │   ├── StatusBarWindowView.java
│   │   ├── SystemUIDialog.java
│   │   ├── TrustDrawable.java
│   │   ├── UnlockMethodCache.java
│   │   ├── UserAvatarView.java
│   │   ├── VelocityTrackerFactory.java
│   │   └── VelocityTrackerInterface.java
│   ├── policy  //这里的都是一些控制器, 一些规则
│   │   ├── AccessibilityContentDescriptions.java
│   │   ├── AccessibilityController.java
│   │   ├── AccessPointControllerImpl.java
│   │   ├── BatteryController.java
│   │   ├── BluetoothControllerImpl.java
│   │   ├── BluetoothController.java
│   │   ├── BrightnessMirrorController.java
│   │   ├── CallbackHandler.java
│   │   ├── CastControllerImpl.java
│   │   ├── CastController.java
│   │   ├── Clock.java
│   │   ├── DateView.java
│   │   ├── DeadZone.java
│   │   ├── EthernetIcons.java
│   │   ├── EthernetSignalController.java
│   │   ├── FlashlightController.java
│   │   ├── HeadsUpManager.java
│   │   ├── HotspotControllerImpl.java
│   │   ├── HotspotController.java
│   │   ├── KeyButtonRipple.java
│   │   ├── KeyButtonView.java
│   │   ├── KeyguardMonitor.java
│   │   ├── KeyguardUserSwitcher.java
│   │   ├── KeyguardUserSwitcherScrim.java
│   │   ├── Listenable.java
│   │   ├── LocationControllerImpl.java
│   │   ├── LocationController.java
│   │   ├── MobileDataControllerImpl.java
│   │   ├── MobileSignalController.java
│   │   ├── NetworkControllerImpl.java
│   │   ├── NetworkController.java
│   │   ├── NextAlarmController.java
│   │   ├── PreviewInflater.java
│   │   ├── RemoteInputView.java
│   │   ├── RotationLockControllerImpl.java
│   │   ├── RotationLockController.java
│   │   ├── ScrollAdapter.java
│   │   ├── SecurityControllerImpl.java
│   │   ├── SecurityController.java
│   │   ├── SignalCallbackAdapter.java
│   │   ├── SignalController.java
│   │   ├── SplitClockView.java
│   │   ├── TelephonyIcons.java
│   │   ├── UserInfoController.java
│   │   ├── UserSwitcherController.java
│   │   ├── WifiIcons.java
│   │   ├── WifiSignalController.java
│   │   ├── ZenModeControllerImpl.java
│   │   └── ZenModeController.java
│   ├── ScrimView.java
│   ├── ServiceMonitor.java
│   ├── SignalClusterView.java
│   ├── SpeedBumpView.java
│   ├── stack
│   │   ├── AmbientState.java
│   │   ├── AnimationFilter.java
│   │   ├── HeadsUpAppearInterpolator.java
│   │   ├── NotificationChildrenContainer.java
│   │   ├── NotificationStackScrollLayout.java
│   │   ├── PiecewiseLinearIndentationFunctor.java
│   │   ├── StackIndentationFunctor.java
│   │   ├── StackScrollAlgorithm.java
│   │   ├── StackScrollState.java
│   │   ├── StackStateAnimator.java
│   │   ├── StackViewState.java
│   │   └── ViewState.java
│   ├── StackScrollerDecorView.java
│   ├── StatusBarIconView.java
│   ├── StatusBarState.java
│   ├── SystemBars.java
│   └── tv    //TV 设备的StatusBar
│       └── TvStatusBar.java
├── SwipeHelper.java
├── SystemUIApplication.java
├── SystemUI.java
├── SystemUIService.java   //最重要的的类
├── SysUIToast.java
├── tuner // 在下拉视图里长按Setting按钮就可以出现, 就是快速设置面板的调节
│   ├── AutoScrollView.java
│   ├── DemoModeFragment.java
│   ├── QsTuner.java
│   ├── StatusBarSwitch.java
│   ├── TunerActivity.java
│   ├── TunerFragment.java
│   └── TunerService.java
├── usb //和usb相关的东西
│   ├── StorageNotification.java   
│   ├── UsbAccessoryUriActivity.java
│   ├── UsbConfirmActivity.java  
│   ├── UsbDebuggingActivity.java    
│   ├── UsbDebuggingSecondaryUserActivity.java
│   ├── UsbDisconnectedReceiver.java
│   ├── UsbPermissionActivity.java
│   ├── UsbResolverActivity.java
│   └── UsbStorageActivity.java
├── ViewInvertHelper.java
└── volume  // 和音量有关的, 当手机按下音量键时出现的弹框也是
    ├── D.java
    ├── Events.java
    ├── IconPulser.java
    ├── Interaction.java
    ├── MediaSessions.java
    ├── SafetyWarningDialog.java
    ├── SegmentedButtons.java
    ├── SpTexts.java
    ├── Util.java
    ├── VolumeComponent.java
    ├── VolumeDialogComponent.java
    ├── VolumeDialogController.java
    ├── VolumeDialog.java
    ├── VolumeDialogMotion.java
    ├── VolumePrefs.java
    ├── VolumeUI.java
    ├── ZenFooter.java
    └── ZenModePanel.java   // 勿扰模式的

24 directories, 311 files
```




SystemUI的相关问题
```
SystemUI 的启动流程
系统由灭屏到keyguard的界面, 再到bouncer界面, 再解锁, 最后进入到launcher
按下recent键系统执行流程

```


参考文章:
[Android M: What’s that “Broadcast Tile” for?](https://medium.com/@kcoppock/android-m-what-s-that-broadcast-tile-for-d1cd3a477a5f#.ln5z814xk) author: Kevin Coppock; from: medium
[ SystemUI之功能介绍和UI布局实现](http://blog.csdn.net/azhengye/article/details/50419409)
[ Android4.0 Keyguard解锁屏机制](http://blog.csdn.net/ocean2006/article/details/8079457)
[android 6.0 SystemUI源码分析(1)-SystemUI介绍](http://www.voidcn.com/blog/zhudaozhuan/article/p-5721383.html)
[寻找Android彩蛋之路](http://blog.liudonghua.com/?p=290)
[framework EventLog分析](http://blog.csdn.net/darkengine/article/details/8477502)