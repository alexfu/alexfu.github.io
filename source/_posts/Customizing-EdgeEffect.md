title: "Customizing EdgeEffect"
date: 2013-12-11 19:51
categories: Android
---
If there's one thing Android is good for, it's being able to customize almost every aspect of your application, down to the nitty gritty. One area that some developers/designers may overlook when customizing is the EdgeEffect.

The [EdgeEffect](http://developer.android.com/reference/android/widget/EdgeEffect.html) is an indicator to let the user know that he/she have reached the end of the scrolling container. You'll also know it as the blue overscroll glow... <!-- more -->

{% img http://cyrilmottier.com/media/2012/03/the-pull-to-refresh-an-anti-ui-pattern-on-android/edge_effect.jpg %}

Unfortunately, customizing the color of the EdgeEffect is not obvious at all. In fact, Android has made it pretty clear that this [EdgeEffect should not be customizable](https://github.com/android/platform_frameworks_base/blob/master/core/java/android/widget/EdgeEffect.java#L139-L140) (either by accident or intentional) since the Drawable is private, there is no public method to change it, and it's final.

The most obvious way to approach this is to completely rewrite the EdgeEffect class to force it to use your version of the Drawable, but then you would have to also rewrite ScrollView, ListView, etc to incorporate your new EdgeEffect. *This is not ideal* since these widgets are subject to change when a new API is released.

A better way, which I did not think to do (kudos to [AndroidAlliance](https://github.com/AndroidAlliance/EdgeEffectOverride)) , is to use a [ContextWrapper](http://developer.android.com/reference/android/content/ContextWrapper.html). A ContextWrapper allows you to "intercept" calls to the usual Context functions, like `getResources()`. If we can change the way `getResources()` behaves, we can then serve up our own customized Drawables when calling `getResources().getDrawable()`. Here's a quick run down on the approach...

1. Create a class which extends ContextWrapper.
2. Create a class which extends Resources.
    * Override `getDrawable()` to serve up a custom Drawable based on the given Id.
3. Override `getResources()` to serve up your custom Resources class.

Check out the code [here](https://gist.github.com/alexfu/7921852).

Now the other half is to actually implement this. Create your own ScrollView or ListView and in the constructor where you pass a Context to `super()`, wrap the Context with your custom ContextWrapper like so...

    public CustomScrollView(Context context, AttributeSet attrs, int defStyle) {
      super(new MyContextWrapper(context), attrs, defStyle);
    }

... and don't forget to reference your custom scrolling widget from your XML layouts. You've now got a customized EdgeEffect...

{% img http://i39.tinypic.com/r6zasl.png %}
