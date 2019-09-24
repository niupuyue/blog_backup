---
title: éćžandroidčˇŻ(ĺäş) material design
date: 2016-07-20 23:29:10
tags:
 - Android
---

ĺ¨Google I/O 2015ĺ¤§äźä¸­ďźGoogleä¸şAndroidĺźĺčäťçťäşDesign Support Libraryăčżä¸ŞlibraryĺŻäťĽčŽŠĺźĺčĺžĺŽšćĺ°ĺŽç°ć´ĺ¤Material DesignćŚĺżľĺ°äťäťŹçĺşç¨ä¸­ďźĺ ä¸şĺžĺ¤ĺłéŽĺç´ ćŻä¸ĺŻç¨çĺ¨ĺćĽçćĄćśĺ¤ăéŚĺĺ°ąćŻĺžćäşä˝żç¨ďźDesign Support Libraryĺä¸ĺźĺŽšĺ°API 7ăDesign Support LibraryĺŻäťĽĺźĺĽĺ°ä˝ çAndroidĺˇĽç¨ä¸­éčżĺŻźĺĽGradleäžčľ.
<!--more-->
čĺ¨GoogleĺŽç˝ä¸­ç<a href='https://developer.android.com/reference/android/support/design/widget/package-summary.html'>android.support.design.widget</a>ä¸­ďźčŻŚçťćčż°äşčżä¸Şĺ ä¸Şçťäťś(çćŹäťĽ24.1.0ä¸şĺ)
ćł¨é

| ćł¨éĺç§° | ĺććčż° |
|---|---|
| CoordinatorLayout.DefaultBehavior	| Defines the default CoordinatorLayout.Behavior of a View class. |

çąť

| çťäťśĺç§° | ĺććčż° |
|--------- |:----------------------- :|
| AppBarLayout | AppBarLayout is a vertical LinearLayout which implements many of the features of material designs app bar concept, namely scrolling gestures. |
| AppBarLayout.Behavior  | The default AppBarLayout.Behavior for AppBarLayout. |
| BottomNavigationView | Represents a standard bottom navigation bar for application. |
| BottomSheetBehavior<V extends View> | An interaction behavior plugin for a child view of CoordinatorLayout to make it work as a bottom sheet. |
| BottomSheetDialog | Base class for Dialogs styled as a bottom sheet. |
| CollapsingToolbarLayout | CollapsingToolbarLayout is a wrapper for Toolbar which implements a collapsing app bar. |
| CoordinatorLayout | CoordinatorLayout is a super-powered FrameLayout. |
| CoordinatorLayout.Behavior<V extends View> | Interaction behavior plugin for child views of CoordinatorLayout. |
| CoordinatorLayout.LayoutParams | Parameters describing the desired layout for a child of a CoordinatorLayout. |
| FloatingActionButton | Floating action buttons are used for a special type of promoted action. |
| FloatingActionButton.Behavior | Behavior designed for use with FloatingActionButton instances. |
| NavigationView | Represents a standard navigation menu for application. |
| Snackbar | Snackbars provide lightweight feedback about an operation. |
| TabLayout | TabLayout provides a horizontal layout to display tabs. |
| TabLayout.Tab	| A tab in this layout. |
| TabLayout.TabLayoutOnPageChangeListener | A ViewPager.OnPageChangeListener class which contains the necessary calls back to the provided TabLayout so that the tab position is kept in sync. |
| TabLayout.ViewPagerOnTabSelectedListener | A TabLayout.OnTabSelectedListener class which contains the necessary calls back to the provided ViewPager so that the tab position is kept in sync.|
| TextInputEditText | 	A special sub-class of EditText designed for use as a child of TextInputLayout |
| TextInputLayout | Layout which wraps an EditText (or descendant) to show a floating label when the hint is hidden due to the user inputting text. |

ĺŚćéčŚä˝żç¨čżäşć°ĺźĺĽççťäťść ˇĺźďźéčŚĺ ĺĽć°çäžčľ

```
compile 'com.android.support:design:22.2.0'
```

# Material Text Input
EditTextčŞäťćĺźĺ§ĺ°ąĺˇ˛çťĺ¨Androidä¸­äşďźĺšśä¸ä˝żç¨ĺžçŽĺďźäťäťŹä¸ç´ć˛Ąćäťäšćšĺăä˝żç¨Design Support LibraryďźGoogleĺˇ˛çťäťçťäşć°çĺĺŤĺŽšĺ¨ĺŤä˝TextInputLayoutăčżä¸Şć°çviewćˇťĺ ĺč˝ĺ°ć ĺçEditTextä¸ďźäžĺŚćŻćčŽŠä˝ çç¨ćˇçé˘ĺźšĺşéčŻŻćśćŻĺĺ¨çťćç¤ş
```
<android.support.design.widget.TextInputLayout
    android:id="@+id/textinput"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <EditText
        android:id="@+id/edittext"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="What is your name?" />

</android.support.design.widget.TextInputLayout>
```
# SnackBar
SnackBaréčżĺ¨ĺąĺšĺşé¨ĺąç¤şçŽć´çäżĄćŻďźä¸şä¸ä¸Şćä˝ćäžäşä¸ä¸Şč˝ťéçş§çĺéŚďźĺšśä¸ĺ¨Snackbarä¸­čżĺŻäťĽĺĺŤä¸ä¸Şćä˝ďźĺ¨ĺä¸ćśé´ĺďźäťä¸ĺŞč˝ćžç¤şä¸ä¸Ş SnackbarďźĺŽçćžç¤şäžčľäşUIďźä¸ĺToastéŁć ˇĺŻäťĽčąçŚťĺşç¨ćžç¤şăĺŽçç¨ćłĺToastĺžç¸äźźďźĺŻä¸ä¸ĺçĺ°ąćŻĺŽççŹŹä¸ä¸Şĺć°ä¸ćŻäź ĺĽContextčćŻäź ĺĽĺŽćäžéççśč§ĺžďźä˝ćŻäťćŻToastć´ĺźşĺ¤§ă
ç¤şäžďź
```
Snackbar.make(rootContainer,"ćç¤şäżĄćŻ",Snackbar.LENGTH_SHORT).show()
```
ćć
![material design SnackBarćć](/assets/material/materialdesign01.png)
SnackBarçćžç¤şććĺToasçć ˇĺźĺşćŹç¸ĺ.ä¸ĺäšĺ¤ĺ¨äşďźćäťŹĺŻäťĽä¸şĺ˝ĺçSnackBarćäžä¸ä¸ŞçšĺťäšĺéčŚć§čĄçActionă
`
Snackbar.make(mDrawerLayout, "SnackbarClicked", Snackbar.LENGTH_SHORT).setAction("Action", new View.OnClickListener() {
@Override
public void onClick(View v) {
Toast.makeText(MainActivity.this, "I'm a Toast", Toast.LENGTH_SHORT).show();
}
}).setActionTextColor(Color.RED).show();
`
čżćŻćäťŹä¸şSnackBarčŽžç˝Žäşä¸ä¸ŞActionďźĺšśä¸čŽžç˝Žäşé˘č˛ćŻçş˘č˛ăĺšśä¸ä¸şäťčŽžç˝Žäşçšĺťäşäťśă
# FloatingActionButton
FloatingActionButtonäťĺĺ­ĺŻäťĽçĺşĺŽćŻä¸ä¸ŞćľŽĺ¨çćéŽďźĺŽćŻä¸ä¸Şĺ¸Śćé´ĺ˝ąçĺĺ˝˘ćéŽďźĺŻäťĽéčżfabSizećĽćšĺĺśĺ¤§ĺ°ďźä¸ťčŚč´č´Łçé˘çĺşćŹćä˝ďźčżä¸ŞćéŽćťä˝ćĽčŻ´čżćŻćŻčžçŽĺçă
1. éťčŽ¤FloatingActionButton çčćŻč˛ćŻĺşç¨ä¸ťé˘ç colorAccentďźĺśĺŽMDä¸­çć§äťśä¸ťé˘éťčŽ¤ĺşćŹé˝ćŻĺşç¨çčżä¸Şä¸ťé˘ďźďźĺŻäťĽéčżapp:backgroundTint ĺąć§ćčsetBackgroundTintList (ColorStateList tint)ćšćłĺťćšĺčćŻé˘č˛ă
2. ä¸é˘ćĺ° Floating action button çĺ¤§ĺ°ĺ°şĺŻ¸ďźĺŻäťĽç¨čż**app:fabSize **ĺąć§čŽžç˝Žďźnormal or miniďź
3. app:rippleColor čĄ¨ç¤şčŽžç˝Žć ˇĺź(ĺŚć°´ćł˘çşš)
4. app:borderWidth: čŽžç˝ŽčžšćĄĺŽ˝ĺşŚ
5. app:elevationčŽžç˝ŽćŽéçśćé´ĺ˝ąçćˇąĺşŚďźéťčŽ¤ćŻ 6dpďź
6. app:pressedTranslationZčŽžç˝Žçšĺťçśćçé´ĺ˝ąćˇąĺşŚďźéťčŽ¤ćŻ 12dpďź
7. android:src ĺąć§ćšĺ drawable
```
<android.support.design.widget.FloatingActionButton
android:id="@+id/fab_search"
android:layout_width="wrap_content"
android:layout_height="wrap_content"
android:layout_gravity="bottom|end"
android:layout_margin="16dp"
android:src="@android:drawable/ic_dialog_email"
app:borderWidth="2dp"
app:fabSize="normal"
app:rippleColor="#ff0000" />
```
# coordinatorLayout
ćŻä¸ä¸Şĺ˘ĺźşççframeLayoutďźäťĺŻäťĽĺč°ĺ­Viewäšé´çäş¤äşďźäťččžžĺ°ć§ĺśćĺżćć
ććĺŚä¸ĺžćç¤şďź
![coordinatorLayoutçä˝żç¨](/assets/material/simple_coordinator.gif)
ĺ¨čżä¸Şäžĺ­ä¸­ćäťŹĺŻäťĽçĺ°Viewäšé´ćŻĺŚä˝ç¸äşéĺçďźViewäźć šćŽĺśäťViewçĺĺ¨ĺç¸ĺşçĺĺă

ä¸é˘ćŻCoordinatorLayoutççŽĺä˝żç¨äžĺ­
```
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/background_light"
    android:fitsSystemWindows="true"
    >
    <android.support.design.widget.AppBarLayout
        android:id="@+id/main.appbar"
        android:layout_width="match_parent"
        android:layout_height="300dp"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
        android:fitsSystemWindows="true"
        >
        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/main.collapsing"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_scrollFlags="scroll|exitUntilCollapsed"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorPrimary"
            app:expandedTitleMarginStart="48dp"
            app:expandedTitleMarginEnd="64dp"
            >
            <ImageView
                android:id="@+id/main.backdrop"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scaleType="centerCrop"
                android:fitsSystemWindows="true"
                android:src="@drawable/material_flat"
                app:layout_collapseMode="parallax"
                />
            <android.support.v7.widget.Toolbar
                android:id="@+id/main.toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
                app:layout_collapseMode="pin"
                />
        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>
    <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        >
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textSize="20sp"
            android:lineSpacingExtra="8dp"
            android:text="@string/lorem"
            android:padding="@dimen/activity_horizontal_margin"
            />
    </android.support.v4.widget.NestedScrollView>
    <android.support.design.widget.FloatingActionButton
        android:layout_height="wrap_content"
        android:layout_width="wrap_content"
        android:layout_margin="@dimen/activity_horizontal_margin"
        android:src="@drawable/ic_comment_24dp"
        app:layout_anchor="@id/main.appbar"
        app:layout_anchorGravity="bottom|right|end"
        />
</android.support.design.widget.CoordinatorLayout>
```
ćäťŹçä¸ä¸čżä¸ŞlayoutçťćďźCoordinatorLayoutĺĺŤ3ä¸Şĺ­ć§äťśďź
AppbarLayoutďź scrolleable view ĺ anchoredFloatingActionBară
```
<CoordinatorLayout>
    <AppbarLayout/>
    <scrollableView/>
    <FloatingActionButton/>
</CoordinatorLayout>
```
## AppBarLayout
AppBarLayoutćŻä¸ä¸Şçť§ćżäşLinearLayoutçViewGroupçĺŽç°ďźéťčŽ¤AppBarLayoutçćšĺćŻĺç´ćšĺďźĺŻäťĽçŽĄçĺé¨çťäťśĺ¨éĄľé˘ćťĺ¨ćśçčĄä¸ş
ĺŻč˝čŻ­č¨ĺčż°ä¸ćŻéŁäšçć¸ć°ďźçä¸ä¸ĺŽćšćäžçgifĺžç
![AppBarLayoutçĺŽćšĺŽç°ĺ¨ĺž](/assets/material/gif01.gif)
AppBarLayoutĺ¨čżä¸Şäžĺ­ä¸­ćśčč˛çViewďźĺ¨ĺśä¸ćžç˝Žäşä¸ä¸ŞĺŻäťĽçźŠćžçĺžçďźĺśä¸­ĺĺŤä¸ä¸ŞToolbarďź
ä¸ä¸ŞLinearLayoutďźĺĺŤć é˘ĺĺŻć é˘ďźďźäťĽĺä¸ä¸ŞTabLayoută
ćäťŹĺŻäťĽéčżčŽžç˝Žlayout_scrollFlagsĺć°ďźćĽć§ĺśAppBarLayoutä¸­çć§äťśčĄä¸şă
ĺ¨ćäťŹçčżä¸Şäžĺ­ä¸­ďźĺ¤§é¨ĺViewçlayout_scrollFlagsé˝čŽžç˝Žä¸şscrollďźĺŚćć˛ĄćčŽžç˝ŽçčŻďź
ĺ˝ĺŻćťĺ¨çViewčżčĄćťĺ¨ćśďźčżäşć˛ĄčŽžç˝Žä¸şscrollçViewä˝ç˝Žäźäżćä¸ĺďź

layout_scrollFlagsčŽžç˝Žä¸snapĺźĺĺŻäťĽéżĺčżĺĽĺ¨çťä¸­é´çśćďź mid-animation-statesďźďź
čżćĺłçĺ¨çťäźä¸ç´ćçť­ĺ°ViewĺŽĺ¨ćžç¤şćĺŽĺ¨éčä¸şć­˘ă

LinearLayoutĺśä¸­ĺĺŤäşä¸ä¸Şć é˘ĺä¸ä¸ŞĺŻć é˘ďźĺ˝ç¨ćˇĺä¸ç§ťĺ¨ćśLinearLayoutćŻä¸ç´ćžç¤şçďźç´ĺ°ç§ťĺşĺąĺšďźenterAlwaysďź;

TabLayoutäźä¸ç´ćŻĺŻč§çďźĺ ä¸şćäťŹć˛Ąćĺ¨TabLayoutä¸čŽžç˝Žäťťä˝flagă

ć­ŁĺŚä˝ ćč§ďźAppbarLayoutçĺźşĺ¤§çŽĄçč˝ĺćŻéčżĺ¨Viewä¸čŽžç˝Žä¸ĺscroll flagsĺŽç°çă
```
<AppBarLayout>
    <CollapsingToolbarLayout
        app:layout_scrollFlags="scroll|snap"
        />
    <Toolbar
        app:layout_scrollFlags="scroll|snap"
        />
    <LinearLayout
        android:id="+id/title_container"
        app:layout_scrollFlags="scroll|enterAlways"
        />
    <TabLayout /> <!-- no flags -->
</AppBarLayout>
```
ä¸é˘AppBarLayoutçflagsďź

| ĺĺź | čŻ´ć |
|------|------|
|SCROLL_FLAG_ENTER_ALWAYS | ĺ˝äťťä˝ĺä¸ćťĺ¨äşäťśĺçćś, Viewé˝äźç§ťĺĽ , ä¸çŽĄscrolling view ćŻĺŚć­Łĺ¨ćťĺ¨ă|
|SCROLL_FLAG_ENTER_ALWAYS_COLLAPSED | âenterAlwaysâçéĺ ć čŻďźĺŽä˝żĺžreturning viewć˘ĺ¤ĺ°ćĺŽçćĺ°éŤĺşŚĺćĺźĺ§ćžç¤şďźçśĺĺć˘ć˘ĺąĺźă|
|SCROLL_FLAG_EXIT_UNTIL_COLLAPSED | ä˝ĺä¸ç§ťĺşĺąĺšćśďźViewäźä¸ç´ćśçźŠĺ°ćĺ°éŤĺşŚĺďźĺç§ťĺşĺąĺšă|
|SCROLL_FLAG_SCROLL | View äźć šćŽćťĺ¨äşäťśčżčĄç§ťĺ¨ă|
|SCROLL_FLAG_SNAP | ä˝ćťĺ¨çťććśďźĺŚćViewĺŞćé¨ĺĺŻč§ďźĺŽĺ°äźčŞĺ¨ćťĺ¨ĺ°ćčżçčžšçďźĺŽĺ¨ĺŻč§ćĺŽĺ¨éčďź|

## CoordinatorLayout behavior
### SwipeDismissBehavior
ćˇąĺĽdesign support libraryçäťŁç ďźćäťŹäźĺç°ä¸ä¸Şć°ççąťďźSwipeDismissBehaviorďźä˝żç¨čżä¸ŞBehaviorďź
ćäťŹĺŻäťĽĺžĺŽšćçä˝żç¨CoordinatorLayoutĺŽç°ćťĺ¨ĺ é¤ĺč˝:
![ćźç¤ş](/assets/material/coordinator03.gif)
```
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_swipe_behavior);
    mCardView = (CardView) findViewById(R.id.swype_card);
    final SwipeDismissBehavior<CardView> swipe
        = new SwipeDismissBehavior();
        swipe.setSwipeDirection(
            SwipeDismissBehavior.SWIPE_DIRECTION_ANY);
        swipe.setListener(
            new SwipeDismissBehavior.OnDismissListener() {
            @Override public void onDismiss(View view) {
                Toast.makeText(SwipeBehaviorExampleActivity.this,
                    "Card swiped !!", Toast.LENGTH_SHORT).show();
            }
            @Override
            public void onDragStateChanged(int state) {}
        });
        LayoutParams coordinatorParams =
            (LayoutParams) mCardView.getLayoutParams();
        coordinatorParams.setBehavior(swipe);
    }
```
### custom Behavior
é¤äşä¸é˘çčżç§äšĺ¤ďźćäťŹčżĺŻäťĽčŞĺŽäšăéŚĺćäťŹĺşčŻĽćć¸ćĽä¸¤ä¸Şć ¸ĺżchildĺdependency
![ćźç¤ş](/assets/material/coordinator04.png)
#### child ĺ dependency
child ćŻćéčŚĺşç¨behaviorçView ďźdependency ćäťťč§Śĺbehaviorçč§č˛ďźĺšśä¸childčżčĄäşĺ¨ă
ĺ¨čżä¸Şäžĺ­ä¸­ďź child ćŻImageViewďź dependency ćŻToolbarďźĺŚćToolbarĺçç§ťĺ¨ďźImageViewäšäźĺç¸ĺşçç§ťĺ¨ă
![](assets/android_material_design/coordinator05.gif)
ç°ĺ¨ćäťŹĺˇ˛çťçĽéćŚĺżľäşďźćĽçćäťŹççćäšĺŽç°ďź
çŹŹä¸ć­ĽćäťŹéčŚçť§ćżCoordinatorLayout.BehaviorďźTćŻććä¸ä¸ŞViewďź
ĺ¨ćäťŹçäžĺ­ä¸­ćŻImageViewďź çť§ćżĺďźćäťŹĺżéĄťĺŽç°äťĽä¸2ä¸Şćšćł:
- layoutDependsOn
- onDependentViewChanged
layoutDependsOnćšćłĺ¨ćŻćŹĄlayoutĺçĺĺćśé˝äźč°ç¨ďźćäťŹéčŚĺ¨dependencyć§äťśĺçĺĺćśčżĺTrueďźĺ¨ćäťŹçäžĺ­ä¸­ćŻç¨ćˇĺ¨ĺąĺšä¸ćťĺ¨ćśďźĺ ä¸şToolbarĺçäşç§ťĺ¨ďźďźçśĺćäťŹéčŚčŽŠchildĺĺşç¸ĺşçĺĺşă
```
@Override
  public boolean layoutDependsOn(     
     CoordinatorLayout parent,
     CircleImageView, child,
     View dependency) {
     return dependency instanceof Toolbar;
 }
 ```
 ä¸ćŚlayoutDependsOnčżĺäşTrueďźçŹŹäşä¸ŞćšćłonDependentViewChangedĺ°ąäźč˘Ťč°ç¨ďź
ĺ¨čżä¸ŞćšćłéćäťŹéčŚĺŽç°ĺ¨çťďźč˝Źĺşç­ććă
```
public boolean onDependentViewChanged(
      CoordinatorLayout parent,
      CircleImageView avatar,
      View dependency) {
      modifyAvatarDependingDependencyState(avatar, dependency);
   }
   private void modifyAvatarDependingDependencyState(
    CircleImageView avatar, View dependency) {
        //  avatar.setY(dependency.getY());
        //  avatar.setBlahBlat(dependency.blah / blah);
    }
```
ć´ĺäšĺçäťŁç ćŻďź
```
public static class AvatarImageBehavior
   extends CoordinatorLayout.Behavior<CircleImageView> {
   @Override
   public boolean layoutDependsOn(
    CoordinatorLayout parent,
    CircleImageView, child,
    View dependency) {
    return dependency instanceof Toolbar;
  }
  public boolean onDependentViewChanged(   
    CoordinatorLayout parent,
    CircleImageView avatar,
    View dependency) {
      modifyAvatarDependingDependencyState(avatar, dependency);
   }
  private void modifyAvatarDependingDependencyState(
    CircleImageView avatar, View dependency) {
        //  avatar.setY(dependency.getY());
        //  avatar.setBlahBlah(dependency.blah / blah);
    }    
}
```
ĺ ä¸ŞĺĺĽ˝çĺ°ć ĺ­đ°ďźäžĺ¤§ĺŽśĺč


# NavigationView
čżä¸ŞĺśĺŽĺ°ąćŻäž§ćťć ďźäšĺćčŞĺˇąĺčżä¸ä¸Şäž§ćťć ďźä˝ćŻĺč˝ćŻčžĺä¸ďźčä¸ĺäşĺĽ˝ĺ¤ďźćč§ĺ¨ĺŽç°ĺč°ç­ćä˝çćśĺďźćŻčžĺ¤ćăä¸čżäšĺĺ°ąćŻç¨äşĺŽćšçNavigationViewďźçĄŽĺŽćŻčžĺĽ˝ç¨
```
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:app="http://schemas.android.com/apk/res-auto"
android:id="@+id/drawer_layout"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:fitsSystemWindows="true">

<!--ĺĺŽšĺş-->
<android.support.design.widget.CoordinatorLayout
    android:id="@+id/main_content"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <include
        android:id="@+id/appbar"
        layout="@layout/toolbar" />

    <FrameLayout
        android:id="@+id/frame_content"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_below="@+id/appbar"
        android:scrollbars="none"
        app:layout_behavior="@string/appbar_scrolling_view_behavior" />
</android.support.design.widget.CoordinatorLayout>

<!--ĺˇŚäž§ĺŻźčŞčĺ-->
<android.support.design.widget.NavigationView
    android:id="@+id/navigation_view"
    android:layout_width="wrap_content"
    android:layout_height="match_parent"
    android:layout_gravity="start"
    app:headerLayout="@layout/navigation_header"
    app:menu="@menu/drawer" />
</android.support.v4.widget.DrawerLayout>
```
ĺŻäťĽçĺ°čżéćäťŹćŻäťĽDrawerLayoutä˝ä¸şĺśçśĺ¸ĺąďźĺŻšäşDrawLayoutäťĺŻäťĽĺŽç°ä¸ç§ć˝ĺąĺźçäž§ćťććďźčżéä¸ĺ¤ĺčŽ˛č§Ł,čżéĺŞçŽĺčŻ´ä¸çš:DrawLayoutä¸­ççŹŹä¸ä¸Şĺ¸ĺąćŻĺĺŽšĺ¸ĺąďźçŹŹäşä¸ŞćŻčĺĺ¸ĺąăç°ĺ¨ćäťŹç´ćĽĺŽä˝ĺ°NavigationViewďźćäťŹçĺ°čżéć** app:headerLayout="@layout/navigation_header"ăapp:menu="@menu/drawer"**čżä¸¤čĄäťŁç ďźĺśä¸­headerLayoutćŻčŽžç˝Žĺśĺ¤´é¨çĺ¸ĺąďźčżä¸Şĺ¸ĺąćäťŹĺŻäťĽéäžżĺďźĺ°ąĺĺćŽéçĺ¸ĺąćäťśä¸ć ˇçăĺŻšäşmenuĺ°ąćŻčĺéĄšçéç˝Žäşďźĺśéç˝ŽćäťśĺŚä¸
```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:tools="http://schemas.android.com/tools"
tools:context=".MainActivity">
<group android:checkableBehavior="single">
<item
android:id="@+id/navigation_item_home"
android:icon="@drawable/iconfont_home"
android:title="éŚéĄľ" />
<item
android:id="@+id/navigation_item_blog"
android:icon="@drawable/iconfont_blog"
android:title="ćçĺĺŽ˘" />

    <item
        android:id="@+id/navigation_item_about"
        android:icon="@drawable/iconfont_about"
        android:title="ĺłäş" />
</group>
</menu>
```
ĺ°ąčżäšçŽĺďźćäťŹčŚĺŽç°ä¸čż°çććĺ°ąĺŞčŚčżäşĺ°ąčśłĺ¤äşăä˝ćŻĺŚććäťŹćłčŚĺŻšItemćˇťĺ ä¸ä¸Şçšĺťçäşäťśćäšĺĺ˘ďźčŻˇçä¸é˘ďź
```
private void setNavigationViewItemClickListener() {
mNavigationView.setNavigationItemSelectedListener(new NavigationView.OnNavigationItemSelectedListener() {
@Override
public boolean onNavigationItemSelected(MenuItem item) {
switch (item.getItemId()) {
case R.id.navigation_item_home:
mToolbar.setTitle("éŚéĄľ");
switchFragment("MainFragment");
break;
case R.id.navigation_item_blog:
mToolbar.setTitle("ćçĺĺŽ˘");
switchFragment("BlogFragment");
break;
case R.id.navigation_item_about:
mToolbar.setTitle("ĺłäş");
switchFragment("AboutFragment");
break;
default:
break;
}
item.setChecked(true);
mDrawerLayout.closeDrawer(Gravity.LEFT);
return false;
}
});
}
```
ä¸é˘çäťŁç ĺžć¸ćĽäşďźĺ°ąćŻä¸şNavigationViewćˇťĺ äşä¸ä¸ŞOnNavigationItemSelectedListenerççĺŹäşäťśďźçśĺćäťŹĺ°ąĺŻäťĽĺćäťŹćłĺçäşäş


# ĺččľć
[androidç](http://www.jcodecraeer.com/a/anzhuokaifa/developer/2015/0531/2958.html)
[ććĄcoordinatorLayout](https://appkfz.com/2015/11/12/mastering-coordinator/?hmsr=toutiao.io)
[ĺĺ­ĺžéżä¸çĽéčŻĽćäšçżťčŻđ](https://lab.getbase.com/introduction-to-coordinator-layout-on-android/)
