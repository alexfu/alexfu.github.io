title: "Notifying ViewPager Fragments"
date: 2013-06-17 12:01
categories: Fragments
---
As the title suggests, this post is about notifying Fragments in a ViewPager. Even though PagerAdapter has a `notifyDataSetChanged()` method, this method is for a different purpose. There are undoubtly more than one way to [solve this problem][1], but I found this method to be very simple to implement and easy to understand. <!-- more -->

One thing to note about the PagerAdapter class is that...

> ...A data set change may involve pages being added, removed, or changing position...

Based off of this info, it can be safe to say that calling `notifyDataSetChanged()` will not have any effect on updating UI components.

### The Solution
This solution uses a very simple observer pattern. The adapter holds on to an Observable and as the adapter creates a Fragment, we register that Fragment with the Obervable. Then we can call `Observable.notifyObservers()` when a change needs to be made. I've posted the code to Gist since it is much easier to read the code there instead of here -- [https://gist.github.com/alexfu/5797429][2]

[1]: http://stackoverflow.com/questions/7263291/viewpager-pageradapter-not-updating-the-view
[2]: https://gist.github.com/alexfu/5797429
