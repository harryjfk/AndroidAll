Prior to Android "L" preview, the easiest way to setup tabs with Fragments was to use ActionBar Tabs as described in [ActionBar Tabs with Fragments](http://guides.codepath.com/android/ActionBar-Tabs-with-Fragments) guide. However, all methods related to navigation modes in the ActionBar class (such as `setNavigationMode()`, `addTab()`, `selectTab()`, etc.) are now deprecated.

As a result, tabs are now best implemented by leveraging the [[ViewPager|ViewPager-with-FragmentPagerAdapter]] with a custom "tab indicator" on top. In this guide, we will be using the [PagerSlidingTabStrip](https://github.com/astuetz/PagerSlidingTabStrip) to produce tabbed navigation within our app. 

![Tabs](http://i.imgur.com/a2wpJ80.png)

### Install PagerSlidingTabStrip

First, we need to add `PagerSlidingTabStrip` to our application by adding the following to our `app/build.gradle` file:

```gradle
dependencies {
    compile 'com.astuetz:pagerslidingtabstrip:1.0.1'
}
```

Once you have included the library and synced with Gradle, we can use the `SlidingTabLayout` in our layout file to display tabs. Your layout file will have tabs on the top and a `ViewPager` on the bottom as shown in the code snippet below:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <com.astuetz.PagerSlidingTabStrip
        android:id="@+id/tabs"
        app:pstsShouldExpand="true"
        android:layout_width="match_parent"
        android:layout_height="48dp">
    </com.astuetz.PagerSlidingTabStrip>

    <android.support.v4.view.ViewPager
        android:id="@+id/viewpager"
        android:layout_width="match_parent"
        android:layout_height="0px"
        android:layout_weight="1"
        android:background="@android:color/white" />

</LinearLayout>
```

### Create Fragment

Now that we have the `ViewPager` and our tabs in our layout, we should start defining the content of each of the tabs. Since each tab is just a fragment being displayed, we need to create and define the `Fragment` to be shown. You may have one or more fragments in your application depending on your requirements.

In `res/layout/fragment_page.xml` define the XML layout for the fragment which will be displayed on screen when a particular tab is selected:

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center" />
```

In `PageFragment.java` define the inflation logic for the fragment of tab content:

```java
// In this case, the fragment displays simple text based on the page
public class PageFragment extends Fragment {
    public static final String ARG_PAGE = "ARG_PAGE";

    private int mPage;

    public static PageFragment newInstance(int page) {
        Bundle args = new Bundle();
        args.putInt(ARG_PAGE, page);
        PageFragment fragment = new PageFragment();
        fragment.setArguments(args);
        return fragment;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mPage = getArguments().getInt(ARG_PAGE);
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_page, container, false);
        TextView textView = (TextView) view;
        textView.setText("Fragment #" + mPage);
        return view;
    }
}
```

### Implement FragmentPagerAdapter

The next thing to do is to implement the adapter for your `ViewPager` which controls the order of the tabs, the titles and their associated content. The most important methods to implement here are `getPageTitle(int position)` which is used to get the title for each tab and `getItem(int position)` which determines the fragment for each tab.

```java
public class SampleFragmentPagerAdapter extends FragmentPagerAdapter {
    final int PAGE_COUNT = 3;
    private String tabTitles[] = new String[] { "Tab1", "Tab2", "Tab3" };

    public SampleFragmentPagerAdapter(FragmentManager fm) {
        super(fm);
    }

    @Override
    public int getCount() {
        return PAGE_COUNT;
    }

    @Override
    public Fragment getItem(int position) {
        return PageFragment.newInstance(position + 1);
    }

    @Override
    public CharSequence getPageTitle(int position) {
        // Generate title based on item position
        return tabTitles[position];
    }
}
```

### Setup Sliding Tabs

Finally, we need to attach our `ViewPager` to the `SampleFragmentPagerAdapter` and then configure the sliding tabs with a two step process:

* In the `onCreate()` method of your activity, find the `ViewPager` and connect the adapter.
* Set the `ViewPager` on the `PagerSlidingTabStrip` to connect the pager with the tabs.

```java
public class MainActivity extends FragmentActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Get the ViewPager and set it's PagerAdapter so that it can display items
        ViewPager viewPager = (ViewPager) findViewById(R.id.viewpager);
        viewPager.setAdapter(new SampleFragmentPagerAdapter(getSupportFragmentManager()));

        // Give the PagerSlidingTabStrip the ViewPager
        PagerSlidingTabStrip tabsStrip = (PagerSlidingTabStrip) findViewById(R.id.tabs);
        // Attach the view pager to the tab strip
        tabsStrip.setViewPager(pager);
    }

}
```

Heres the output:

![Screen 1](http://i.imgur.com/rhRXjLIl.png)

### Customize Tab Styles

You can change the style of the tabs by adding any of the following properties to the tab strip in the layout XML:

```xml
<com.astuetz.PagerSlidingTabStrip
    android:id="@+id/tabs"
    app:pstsShouldExpand="true"
    android:layout_width="match_parent"
    android:layout_height="48dp"
    app:pstsDividerColor="#00000000"
    app:pstsIndicatorColor="#FF33B5E6"
    app:pstsTabPaddingLeftRight="14dip"
    app:pstsUnderlineColor="#FF33B5E6">
</com.astuetz.PagerSlidingTabStrip>
```

### Add Icons to SlidingTabLayout

We can add icons instead of text to each our tabs by implementing the `IconTabProvider` interface within our pager adapter:

```java
public class SampleFragmentPagerAdapter extends FragmentPagerAdapter implements IconTabProvider {
    final int PAGE_COUNT = 3;
    private String tabIcons[] = {R.drawable.ic_tab_one, R.drawable.ic_tab_two, R.drawable.ic_tab_three}

    public SampleFragmentPagerAdapter(FragmentManager fm) {
        super(fm);
    }

    @Override
    public int getCount() {
        return PAGE_COUNT;
    }

    @Override
    public Fragment getItem(int position) {
        return PageFragment.newInstance(position + 1);
    }

    @Override
    public int getPageIconResId(int position) {
        return tabIcons[position];
    }
}
```

And the result is:

![Icons](http://i.stack.imgur.com/c7sm4.png)

## Customization

To not just look like another Play Store styled app, go and adjust these values to match your brand:

 * `pstsIndicatorColor` Color of the sliding indicator
 * `pstsUnderlineColor` Color of the full-width line on the bottom of the view
 * `pstsDividerColor` Color of the dividers between tabs
 * `pstsIndicatorHeight`Height of the sliding indicator
 * `pstsUnderlineHeight` Height of the full-width line on the bottom of the view
 * `pstsDividerPadding` Top and bottom padding of the dividers
 * `pstsTabPaddingLeftRight` Left and right padding of each tab
 * `pstsScrollOffset` Scroll offset of the selected tab
 * `pstsTabBackground` Background drawable of each tab, should be a StateListDrawable
 * `pstsShouldExpand` If set to true, each tab is given the same weight, default false
 * `pstsTextAllCaps` If true, all tab titles will be upper case, default true

All attributes have their respective getters and setters to change them at runtime.

## References

* [PagerSlidingTabStrip](https://github.com/astuetz/PagerSlidingTabStrip)
* [StackOverflow for Styling Icons](http://stackoverflow.com/questions/24838668/icon-selector-not-working-with-pagerslidingtabstrips)