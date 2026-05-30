# Introduction
LGL Mod Menu Template updated and maintained version by me. (Updated in 5/2026)

# How to mod and hook
Main file to create function in mod menu is [Main.cpp](https://github.com/0xP1x3L/LGL-ModMenu-Template/blob/f7e47fb361a06132321bb9b809a238f9796784b9/app/src/main/jni/Main.cpp) file, you can read how i use HOOK.

List of tutorial you need to watch before you make mod.

[Video-1](https://www.youtube.com/watch?v=ieMclBtL6Ig&t)

[Video-2](https://www.youtube.com/watch?v=eMQTR3dSNpw&t)

[LIST-MODDING-TUTORIAL](https://platinmods.com/threads/list-of-platinmods-modding-tutorials.19773)

[LIST-OF-HEX-CODE](https://github.com/0xP1x3L/ModMenu-Template/blob/e2f5a03cdf21d7920ede56c45ddced90846542d8/arm_patching_cheatsheet.md)

Tutorial in video is quite old, but some ways still work.

About tutorial below, you can read how to inject mod in to apk file.

# How to inject mod menu in to Game APKs.

Decompile the game's APK file if you haven't done so already.

After you build your app by Android Studio, nevigate to lib app foulder.

PLEASE REMEMBER TO COPY FILE lib .so of app to lib game. 

Make sure you copy correct Architectures lib app to lib game.

## Step 1: Edit the game's AndroidManifest.xml file

Open the AndroidManifest.xml file of the game you want to inject the mod.

Add the following line inside the <manifest> tag if it does not exist:
```XML
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
Add this line in bottom, you can follow video to know where to add.
```XML
<service android:name="com.android.support.Launcher" android:enabled="true" android:exported="false" android:stopWithTask="true"/>
```
You can find these line in your app ```AndroidManifest.xml``` file.

Note this <application> tag, if the decomplie fails due to full dex classes, need to create a new smali_classesX:
```XML
<application android:appCategory="game" android:appComponentFactory="androidx.core.app.CoreComponentFactory" android:hardwareAccelerated="true" android:icon="@mipmap/app_icon" android:isGame="true" android:label="@string/app_name" android:name="androidx.multidex.MultiDexApplication" android:networkSecurityConfig="@xml/tp_network_security_config" android:roundIcon="@mipmap/app_icon_round" android:supportsRtl="true" android:theme="@android:style/Theme.NoTitleBar.Fullscreen" android:usesCleartextTraffic="true">
```
Need to change to ```android:name="androidx.multidex.MultiDexApplication"``` and then create a folder named smali_classesX (where X is a zero in the existing folders)

## Step 2: Edit the smali file of the MainActivity in the game

Decompile the game's APK file if you haven't done so already.

Find the smali file corresponding to the game's MainActivity. 

Example:

GameDecompiledAPK/smali/com/some_path/some_path/MainActivity.smali.

Open the Main smali file with a text editor.

Search for the onCreate method. It should look like this:

```Smali
.method protected onCreate(Landroid/os/Bundle;)V
  .locals 1

  .line 76
  # [We need add code here]
  iget-object v0, p0, Lcom/google/firebase/UnityPlayerActivity;->mUnityPlayer:Lcom/unity3d/player/UnityPlayer;

return-void
.end method
```
Add the following line of code right after the invoke-super line in the onCreate method:

```Smali
invoke-static {p0}, Lcom/android/support/Main;->StartWithoutPermission(Landroid/content/Context;)V
```
Note this: If you have force closing game when open try to change the line of main smali
```Smali
invoke-static {p0}, Lcom/android/support/Main;->Start(Landroid/content/Context;)V
```

Step 3: Recompile and re-sign the game APK

# About game that you use Hex Patch

If you want people know that you mod this game or app.
You need to find MainActivity smali of game/app

Find "onCreate()" in MainActivity smali

```Smali
.method protected onCreate(Landroid/os/Bundle;)V
    .locals 1

    .line 76
    iget-object v0, p0, Lcom/google/firebase/UnityPlayerActivity;->mUnityPlayer:Lcom/unity3d/player/UnityPlayer;

    if-eqz v0, :cond_0

    .line 77
    iget-object v0, p0, Lcom/google/firebase/UnityPlayerActivity;->mUnityPlayer:Lcom/unity3d/player/UnityPlayer;

    invoke-virtual {v0}, Lcom/unity3d/player/UnityPlayer;->quit()V

    const/4 v0, 0x0

    .line 78
    iput-object v0, p0, Lcom/google/firebase/UnityPlayerActivity;->mUnityPlayer:Lcom/unity3d/player/UnityPlayer;

    .line 80
    :cond_0
    invoke-super {p0, p1}, Lcom/unity3d/player/UnityPlayerActivity;->onCreate(Landroid/os/Bundle;)V

    return-void
.end method
```

Change like this (if you dont understand, use AI, it will explain)

```Smali
.method protected onCreate(Landroid/os/Bundle;)V
    .locals 3

    .line 76
    iget-object v0, p0, Lcom/google/firebase/MessagingUnityPlayerActivity;->mUnityPlayer:Lcom/unity3d/player/UnityPlayer;

    if-eqz v0, :cond_0

    .line 77
    iget-object v0, p0, Lcom/google/firebase/MessagingUnityPlayerActivity;->mUnityPlayer:Lcom/unity3d/player/UnityPlayer;

    invoke-virtual {v0}, Lcom/unity3d/player/UnityPlayer;->quit()V

    const/4 v0, 0x0

    .line 78
    iput-object v0, p0, Lcom/google/firebase/MessagingUnityPlayerActivity;->mUnityPlayer:Lcom/unity3d/player/UnityPlayer;

    .line 80
    :cond_0
    invoke-super {p0, p1}, Lcom/unity3d/player/UnityPlayerActivity;->onCreate(Landroid/os/Bundle;)V

    # ========= ADD these lines =========
    const-string v0, "Modded by 0xP1×3L"
    const/4 v1, 0x1
    invoke-static {p0, v0, v1}, Landroid/widget/Toast;->makeText(Landroid/content/Context;Ljava/lang/CharSequence;I)Landroid/widget/Toast;
    move-result-object v0
    invoke-virtual {v0}, Landroid/widget/Toast;->show()V
    # ========= END =========

    return-void
.end method
```

# How to bypass Pairip protections / Bypass Certificate check
[Pairip Bypass](https://platinmods.com/threads/how-to-bypass-pairip-protections-latest-too-easy.203105/)

Or try this, simply remove this line in smali-folder\com\pairip\application\Application.smali
```Smali
    invoke-static {p1}, Lcom/pairip/SignatureCheck;->verifyIntegrity(Landroid/content/Context;)V
```

# Important Note:
Make sure you have backed up the original game APK file before making any changes.

Good luck with your mod injection!

# License
**GNU General Public License 3**

# Disclaimer
This template was originally created by LGL TEAM. I have updated and maintained it, and shared it for continued use.

This project is intended for educational purposes only. I do not support or encourage its use to gain unfair advantages over others. It is developed for learning and entertainment purposes.

Please be cautious when purchasing source code from platforms such as Telegram, even if the seller appears trustworthy, as there is always a risk of scams. I am not responsible for any such transactions.

This project is and will remain free to use.
