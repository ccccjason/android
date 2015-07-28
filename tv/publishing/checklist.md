<!-- # TV Apps Checklist -->
# TV應用清單

> 編寫:[awong1900](https://github.com/awong1900) - 原文:http://developer.android.com/training/tv/publishing/checklist.html

<!-- Users enjoy the TV app experience when it is consistent, logical, and predictable. They should be able to navigate within your app and throughout Android TV without getting lost or having to "reset" the UI and start over. Users appreciate clear, colorful, and functional interfaces that make the experience magical. With these ideas in mind, you can create an app that fits nicely in Android TV and performs as users expect. -->

用戶喜歡的TV應用應是體驗一致的，有邏輯的和可預測的。他們可以在應用內四處瀏覽，並且不會迷失在應用從而重設UI導致重頭開始。用戶欣賞乾淨的，有色彩的和起作用的界面，這樣的體驗會很好。把這些想法放在腦子中，我們能創造適合Android TV的應用並達到用戶的期望。

<!-- This checklist covers the main aspects of development for both apps and games and provides guidelines to ensure that your app provides the best possible experience. Additional considerations for games only are covered in the Games section. -->

這個清單覆蓋了應用和遊戲的開發的主要方面去確保我們的應用提供了最好的體驗。額外的遊戲注意事項僅被包含在遊戲小節。

<!-- For criteria that qualify an Android TV app on Google Play, see TV App Quality. -->
關於Google Play中Android TV應用的質量標準，參考[TV App Quality](http://developer.android.com/distribute/essentials/quality/tv.html)。

<!-- ## TV Form Factor Support ## -->
## TV格式因素的支持

<!-- These checklist items apply to **Games** and **Apps**. -->
這些清單項目使用在**遊戲**和**應用**中。

<!-- 
1. Identify the main TV activity with the CATEGORY_LEANBACK_LAUNCHER filter in the manifest.  
See Declare a TV Activity.
2. Provide a home screen banner for each language supported by your app
	- Launcher app banner measures 320x180 px
	- Banner resource is located in the drawables/xhdpi directory
 	- Banner image includes localized text to identify the app.  
	See Provide a home screen banner.
3. Eliminate requirements for unsupported hardware in your app.  
See Declaring hardware requirements for TV.
4. Ensure permissions do not imply hardware requirements  
See Declaring permissions that imply hardware features.
-->

1. 確定manifest的主activity使用`CATEGORY_LEANBACK_LAUNCHER`。
	查看[Declare a TV Activity](http://developer.android.com/training/tv/start/start.html#tv-activity)。
2. 提供每種語言的主屏幕橫幅支持。
    - 啟動應用橫幅大小為320x180 px 
    - 橫幅資源放在`drawables/xhdpi`目錄
    - 橫幅圖像包含本地化的文本去識別應用。
    查看[Provide a home screen banner](http://developer.android.com/training/tv/start/start.html#banner)。
3. 消除不支持的硬件要求。
    查看[Declaring hardware requirements for TV](http://developer.android.com/training/tv/start/hardware.html#declare-hardware-requirements)。
4. 確保沒有隱式的權限需求。
    查看[Declaring permissions that imply hardware features](http://developer.android.com/training/tv/start/hardware.html#hardware-permissions)。

<!-- ## User Interface Design ## -->
## 用戶界面設計

<!-- These checklist items apply to **Games** and **Apps**. -->
這些清單項使用在**遊戲**和**應用**中。

<!-- 
1. Provide appropriate layout resources for landscape mode. 
See [Build Basic TV Layouts]().
2. Ensure that text and controls are large enough to be visible from a distance.  
See Build Useable Text and Controls.
3. Provide high-resolution bitmaps and icons for HDTV screens.  
See Manage Layout Resources for TV.
4. Make sure your icons and logo conform to Android TV specifications.  
See Manage Layout Resources for TV.
5. Allow for overscan in your layout.  
See Overscan.
6. Make every UI element work with both D-pad and game controllers.  
See Creating Navigation and Handling Controllers.
7. Change the background image as users browse through content.  
See Update the Background.
8. Customize the background color to match your branding in Leanback fragments.  
See Customize the Card View.
9. Ensure that your UI does not require a touch screen.  
See Touch screen and Declare touch screen not required.
10. Follow guidelines for effective advertising.  
See Provide Effective Advertising.
-->

1. 提供適合橫屏模式的佈局資源。
	查看 [Build Basic TV Layouts](http://developer.android.com/training/tv/start/layouts.html#structure)。
2. 確保文本和控件在一定距離外看是足夠大的。
	查看[Build Useable Text and Controls](http://developer.android.com/training/tv/start/layouts.html#visibility)。
3. 為HDTV屏幕提供高分辨率的位圖和圖標。
	查看 [Manage Layout Resources for TV](http://developer.android.com/design/tv/patterns.html#icons)。
4. 確保我們的圖標和logo符合Android TV的規範。
	查看[Manage Layout Resources for TV](http://developer.android.com/design/tv/patterns.html#icons)。
5. 允許佈局使用overscan。
	查看[Overscan](http://developer.android.com/training/tv/start/layouts.html#overscan)。
6. 使每一個佈局元素都能用D-pad和遊戲控制器操作。
	查看 [Creating Navigation](http://developer.android.com/training/tv/start/navigation.html) 和[Handling Controllers](http://developer.android.com/training/tv/start/navigation.html)。
7. 當用戶通過文本搜索時改變背景圖像。
	查看[Update the Background](http://developer.android.com/training/tv/playback/browse.html#background)。
8. 在Leanback fragments中定製背景顏色去匹配品牌。
	查看[Customize the Card View](http://developer.android.com/training/tv/playback/card.html#background)。
9. 確保我們的UI不需要觸摸屏。
	查看[Touch screen](http://developer.android.com/training/tv/start/hardware.html#no-touchscreen) and [Declare touch screen not required](http://developer.android.com/training/tv/start/start.html#no-touchscreen)。
10. 遵循有效的廣告的指導。
	查看[Provide Effective Advertising](http://developer.android.com/training/tv/start/layouts.html#advertising)。

<!-- ## Search and Content Discovery ## -->
## 搜索和發現內容

<!-- These checklist items apply to **Games** and **Apps**. -->
這些清單項使用在**遊戲**和**應用**中。

<!-- 
1. Provide search results from your app in the Android TV global search box.  
See Provide Data.
2. Provide TV-specific data fields for search.  
See Identify Columns.
3. Make sure your app presents discovered content in a details screen that lets the user start watching the content immediately.  
See Display Your App in the Details Screen.
4. Put relevant, actionable content and categories on the main screen, making it easy to discover content.  
See Recommending TV Content.
-->

1. 在Android TV全局搜索框中提供搜索結果。
	查看[Provide Data](http://developer.android.com/training/tv/discovery/searchable.html#provide)。
2. 提供TV特定數據字段的搜索。
	查看[Identify Columns](http://developer.android.com/training/tv/discovery/searchable.html#columns)。
3. 確保應用的詳情屏幕有可發現的內容以便用戶立即開始觀看。
	查看[Display Your App in the Details Screen](http://developer.android.com/training/tv/discovery/searchable.html#details)。
4. 放置相關的，可操作的內容和目錄在主屏幕，使用戶容易的發現內容。
	查看[Recommending TV Content](http://developer.android.com/training/tv/discovery/recommendations.html)。

<!-- ## Games ## -->
## 遊戲

<!-- These checklist items apply to **Games**. -->
這些清單項目使用在**遊戲**。

<!-- 
1. Show your game on the home screen with the isGame flag in the manifest.  
See Show your game on the home screen.
2. Make sure game controller support does not depend upon the Start, Select, or Menu buttons (not all controllers have these).  
See Input Devices.
3. Use a generic gamepad graphic (without specific controller branding) to show game button mappings.  
See Show controller instructions.
4. Check for both ethernet and WiFi connectivity.  
See Networking.
5. Provide users with a clean exit.  
See Exit. and Apps.
-->

1. 在manifest中用`isGame`標記讓遊戲顯示在主屏幕上。
	查看[Show your game on the home screen](http://developer.android.com/training/tv/games/index.html#Launcher)。
2. 確保遊戲控制器可以不依靠開始，選擇，或者菜單鍵操作(不是所有控制器有這些按鍵)。
	查看[Input Devices](http://developer.android.com/training/tv/games/index.html#control)。
3. 使用通常的遊戲手柄佈局（不包括特殊的控制器品牌）去顯示遊戲按鍵示意圖。
	查看[Show controller instructions](http://developer.android.com/training/tv/games/index.html#ControllerHelp)。
4. 檢查網絡和WiFi連接。
	查看[Networking](http://developer.android.com/training/tv/games/index.html#networking)。
5. 提供給用戶清晰的退出提示。
	查看[Exit](http://developer.android.com/training/tv/games/index.html#exit)。

----------------
