title: "Using Custom ActionBar Title Views"
date: 2013-04-27 12:42
categories: Android
comments: true
---
### The Problem
The default ActionBar is good for most use cases, however some applications may require further customization to establish a brand. Customizing the ActionBar title may seem quite trivial, however, there are a couple of things that you can do to create an even more polished experience.

As you may have guessed already, [setting a custom View][1] is the way to go...

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    ActionBar ab = getActionBar();
    ab.setDisplayShowTitleEnabled(false);
    ab.setDisplayShowCustomEnabled(true);
    View customTitle = getLayoutInflater().inflate(R.layout.custom_title, null);
    ab.setCustomView(mTitleView);
}
```

The above code will absolutely work with zero problems... <!--more-->except for one minor, minor detail. If you run the above code, you will indeed get the custom view, however, *right before* the custom view is displayed, your original application title will breifly make an appearance.

Granted this will probably only be noticeable when you run the app for the very first time or if you force stop the app and start it back up, but nevertheless the experience is still a little strange and it's best to simply eliminate it.

### The Solution
Simply including the `android:displayOptions` item in your custom ActionBar style will allow you to control what you want to show in the ActionBar. Options include...

+ `showCustom`
+ `showHome`
+ `none`
+ `disableHome`
+ `homeAsUp`
+ `useLogo`

You can even use a combination of these options. The benefits of using this method over the previous is that

1. You no longer have to call `setDisplayShowTitleEnabled()` and `setDisplayShowCustomEnabled()`
2. The original title will no longer appear before your custom View.


```xml
<resources>
    <style name="Theme.Default" parent="@android:Theme.Holo.Light.DarkActionBar">
        <item name="android:actionBarStyle">@style/Widget.ActionBar</item>
    </style>

    <style name="Widget.ActionBar" parent="@android:style/Widget.Holo.Light.ActionBar.Solid.Inverse">
        <item name="android:displayOptions">showCustom|showHome</item>
    </style>
</resources>
```

... and for ActionBarSherlock users ...

```xml
<resources>
    <style name="Theme.Default" parent="Theme.Sherlock.Light.DarkActionBar">
        <item name="actionBarStyle">@style/Widget.ActionBar</item>
    </style>

    <style name="Widget.ActionBar" parent="Widget.Sherlock.Light.ActionBar.Solid.Inverse">
        <item name="displayOptions">showCustom|showHome</item>
    </style>
</resources>
```
[1]: http://developer.android.com/reference/android/app/ActionBar.html#setCustomView(android.view.View)
