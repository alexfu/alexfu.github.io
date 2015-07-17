title: "Managing Fragment States Manually"
date: 2013-12-09 14:27
categories: Android
---
If you find yourself in a bit of a sticky situation where you need to save Fragment state *manually*, thankfully Android has a simply way of accomplishing this.

## Step 1 - Get the Fragment

Obtain the target Fragment. Assuming you've already attached it to your Activity, use `FragmentManager.findFragmentById()` or `FragmentManager.findFragmentByTag()`. For example...

    Fragment myFragment = getFragmentManager.findFragmentById(R.id.content);

## Step 2 - Save with saveFragmentInstanceState

Call `saveFragmentInstanceState` of your FragmentManager and pass the target Fragment through. For example...

    Fragment.SavedState myFragmentState = getFragmentManager().saveFragmentInstanceState(myFragment);

The `saveFragmentInstanceState` method will return a `Fragment.SavedState` object which you will hold on to until you need to restore your Fragments.

## Step 3 - Restore with setInitialSavedState

When you're ready to restore your Fragment, call `setInitialSavedState()`, passing through the `Fragment.SavedState` object you held on to. For example...

    Fragment myFragment = new MyFragment();
    myFragment.setInitialSavedState(myFragmentState);

    // Attach Fragments....

You'll also want to implement the `onSavedInstanceState()` method in your target Fragment. This method will be called when saving state. Put anything you want into the provided `Bundle` that needs to be persisted (like counters and such) and restore in `onCreateView` or `onCreate`.
