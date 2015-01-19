Prior to Android "L" preview, the easiest way to setup tabs with Fragments was to use ActionBar Tabs as described in [ActionBar Tabs with Fragments](http://guides.codepath.com/android/ActionBar-Tabs-with-Fragments) guide. However, all methods related to navigation modes in the ActionBar class (such as setNavigationMode(), addTab(), selectTab() etc.) are now deprecated.

As a direct replacement, there are two examples on how you can implement this on the official samples page:

* [SlidingTabsBasic](http://developer.android.com/samples/SlidingTabsBasic/index.html)
* [SlidingTabsColors](http://developer.android.com/samples/SlidingTabsColors/index.html)

To implement Google Play style sliding tabs, you first need to include the following two java source files to your application. Please make sure to copy the files directly from the Google IO links specified below. The [official samples page](http://developer.android.com/samples/SlidingTabsBasic/index.html) does not seem to have the latest source code as of this writing.
  
* [SlidingTabLayout.java](https://github.com/google/iosched/blob/0a90bf8e6b90e9226f8c15b34eb7b1e4bf6d632e/android/src/main/java/com/google/samples/apps/iosched/ui/widget/SlidingTabLayout.java)
* [SlidingTabStrip.java](https://github.com/google/iosched/blob/0a90bf8e6b90e9226f8c15b34eb7b1e4bf6d632e/android/src/main/java/com/google/samples/apps/iosched/ui/widget/SlidingTabStrip.java)

You may choose to move them to a suitable package in your project.

Once you have included `SlidingTabLayout.java` and `SlidingTabStrip.java` files within your app, you can use the `SlidingTabLayout` in your layout file. Your layout file will have tabs on the top and a `ViewPager` on the bottom as shown in the code snippet below.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <com.example.android.slidingtabsexample.SlidingTabLayout
        android:id="@+id/sliding_tabs"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <android.support.v4.view.ViewPager
        android:id="@+id/viewpager"
        android:layout_width="match_parent"
        android:layout_height="0px"
        android:layout_weight="1"
        android:background="@android:color/white" />

</LinearLayout>
```

### Create Fragment
Next, create the `Fragment` to be shown on click of the tabs. You may have have one or more fragments in your applications depending on your requirements.

fragment_page.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center" />
```

PageFragment.java
```java
public class PageFragment extends Fragment {
    public static final String ARG_PAGE = "ARG_PAGE";

    private int mPage;

    public static PageFragment create(int page) {
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
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_page, container, false);
        TextView textView = (TextView) view;
        textView.setText("Fragment #" + mPage);
        return view;
    }
}
```

### Setup Sliding Tabs
Setting up the sliding tabs is a two step process:
* In the `onCreate` method of your activity, find the `ViewPager` and set its adapter.
* Set the `ViewPager` on the `SlidingTabLayout`.

```java
public class MainActivity extends FragmentActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Get the ViewPager and set it's PagerAdapter so that it can display items
        ViewPager viewPager = (ViewPager) findViewById(R.id.viewpager);
        viewPager.setAdapter(new SampleFragmentPagerAdapter(getSupportFragmentManager()));

        // Give the SlidingTabLayout the ViewPager
        SlidingTabLayout slidingTabLayout = (SlidingTabLayout) findViewById(R.id.sliding_tabs);
        // Center the tabs in the layout
        slidingTabLayout.setDistributeEvenly(true);
        slidingTabLayout.setViewPager(viewPager);
    }

}
```

### Implement FragmentPagerAdapter
The last thing to do is to implement the adapter for your `ViewPager`. The most important method to implement here is `getPageTitle` which is used to get the title for each tab.

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
        return PageFragment.create(position + 1);
    }

    @Override
    public CharSequence getPageTitle(int position) {
        // Generate title based on item position
        return tabTitles[position];
    }
}
```

![Screen 1](http://i.imgur.com/rhRXjLIl.png)