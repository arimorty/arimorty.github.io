---
title:  "How to position a FloatingSearchView in the center of the screen"
date:   2016-08-17 07:59:23
tags: [floating-search-view]
---

<br/>
**Introduction**

A new issue was opened recently in the ```FloatingSearchView``` library with the following question:
How can I put the ```FloatingSearchView``` in the middle of a fragment and push it up to the
top when it gains focus?

Usually I answer questions like that in the issue that was opened for it, but this time I want
to try a different approach. I will answer the question by implementing an example, and I will also include the example's source code in the sample app.

<br/>
**Let's get started**

First of all, let's define what we want to achieve. Let's assume we want to have some kind of a material design header
in a fragment, and right below the header we want to position our ```FloatingSearchView```. When the search gains focus, we want
our search view to slide all the way to the top in order to start a full screen search.

![Our fragment's look]({{ site.url }}/images/sliding_search_example_goal.png)

If you look at the [layout][sliding_search_fragment_layout] file below, you will see that we have two main views, the header view and a ```FrameLayout``` that contains the search view (I will explain in a minute why we need to wrap the search view in a container). The important thing to take away from this layout is that we are moving the search view to the bottom of the header view by setting its ```translationY``` to the header layout's height (btw, this is the only place where I used a ```dimen``` resource because it's used in more than one place in the code, but of course in production you would perhaps set the rest of the values in  the res directory as well)

``` xml

<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
                xmlns:app="http://schemas.android.com/apk/res-auto"
                xmlns:tools="http://schemas.android.com/tools"
                android:id="@+id/parent_view"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:fitsSystemWindows="true"
                tools:context=".MainActivity">

    <View
        android:id="@+id/header_view"
        android:layout_width="match_parent"
        android:layout_height="@dimen/sliding_search_view_header_height"
        android:background="#01579B"/>

    <FrameLayout
        android:id="@+id/dim_background"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <com.arlib.floatingsearchview.FloatingSearchView
            android:id="@+id/floating_search_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:translationY="@dimen/sliding_search_view_header_height"
            app:floatingSearch_close_search_on_keyboard_dismiss="false"
            app:floatingSearch_dimBackground="false"
            app:floatingSearch_dismissOnOutsideTouch="true"
            app:floatingSearch_leftActionMode="showHamburger"
            app:floatingSearch_menu="@menu/menu_search_view"
            app:floatingSearch_searchBarMarginLeft="@dimen/search_view_inset"
            app:floatingSearch_searchBarMarginRight="@dimen/search_view_inset"
            app:floatingSearch_searchBarMarginTop="@dimen/search_view_inset"
            app:floatingSearch_searchHint="Search..."
            app:floatingSearch_showSearchKey="true"
            app:floatingSearch_suggestionsListAnimDuration="250"/>
    </FrameLayout>
</FrameLayout>

```

With the layout ready, we now move on to the [code][sliding_search_fragment] in the ```Fragment```. In the next code snippet you will see how
we can listen to when the search view gains focus and then slide the search view up to the top (which is ```translationY``` 0).

Here is a good place to explain why we needed that extra wrapper view around the search view. Basically, the dim that comes with ```FloatingSearchView```
will start fading in when the search gains focus, which in our case will be while we are animating the view up, so we will end up **not** having the dim background in the portion of the screen that is above the search bar while the slide-up animation is in progress. This
is why we disable the ```FloatingSearchView```'s dim and implement it on our own by manipulating the alpha of a color drawable that is set as
the background of our layout that wraps the search view. This gives us a dim background behind the search view that will cover the entire screen.

Another thing to notice here is that we wait for the animation-up to end before we swap history suggestions, it just
doesn't look good when the suggestions expand while the search view itself animates up, but that's just my opinion.

``` java
@Override
public void onFocus() {
    int headerHeight = getResources()
    .getDimensionPixelOffset(R.dimen.sliding_search_view_header_height);
    ObjectAnimator anim = ObjectAnimator.ofFloat(mSearchView, "translationY",
                     headerHeight, 0);
    anim.setDuration(350);
    anim.addListener(new AnimatorListenerAdapter() {

        @Override
        public void onAnimationEnd(Animator animation) {
           mSearchView.
           swapSuggestions(DataHelper.getHistory(getActivity(), 3));
        }
     });
    anim.start();
    fadeDimBackground(0, 150);
}

```

The ```fadeDimBackgroud(...)``` is just a helper that we use to fade in/out our dim color drawable.  See the source for more info.

Now, when the search loses focus, we reverse both animations (the moving of the search view and fading of the  background) to put the view back in its original position.

``` java
@Override
public void onFocusCleared() {
  int headerHeight = getResources()
  .getDimensionPixelOffset(R.dimen.sliding_search_view_header_height);
  ObjectAnimator anim = ObjectAnimator.ofFloat(mSearchView, "translationY",
          0, headerHeight);
  anim.setDuration(350);
  anim.start();
  fadeDimBackground(150, 0);
}
```
<br/><br/>
**Conclusion**

I hope this was of help. My main goal here was to show that ```FloatingSearchView``` is intended to help you
implement the persistent search bar quickly, but you still need to find ways to fit it into your code.

Check out the source for this example in the [sample app][github project].

[github project]:      https://github.com/arimorty/floatingsearchview/tree/master/sample/src/main
[sliding_search_fragment_layout]:      https://github.com/arimorty/floatingsearchview/blob/master/sample/src/main/res/layout/fragment_sliding_search_example.xml
[sliding_search_fragment]:      https://github.com/arimorty/floatingsearchview/blob/master/sample/src/main/java/com/arlib/floatingsearchviewdemo/fragment/SlidingSearchViewExampleFragment.java
