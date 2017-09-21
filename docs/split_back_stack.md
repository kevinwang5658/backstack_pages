# Split Back Stack

Split back stacks are responsible for distributing back events to the correct linear back stack in split navigation flows like bottom bars and view pagers. Currently the way that BackStack handles split navigation flows is that each tab will have it's own separate back stack. This means that in a given tab, pressing back will only give you the previous view in the same tab. We might add multiple tab back stacks in the future.

A split back stack isn't a real back stack. It doesn't hold view creators and can't inflate views. This is left to be the responsibility of the developer to implement since it varies greatly between different containers (bottom bars are way different to view pagers). A split back stack simply delegates back events to the correct linear back stack to handle the touch event.

## Basic Usage

Split back stacks are created similarly to linear back stacks. Use `BackStackManager#createSplitBackStack(String, int)`. The string is the unique tag associated with the split back stack. The int is the default position of the back stack.

Split back stacks don't store view creators, instead they store TAGs that are associated with the linear back stacks within them. To add a new linearBackStack use `SplitBackStack#add(int, String)`. The int is the position of the linear back stack within the split back stack and the String is the TAG associated with it.

```Java
SplitBackStack splitBackStack = backStackManager.createSplitBackStack(SPLIT_TAG, 0);

splitBackStack.add(position, tag);
```

## Examples

#### View Pagers

To use a split back stack with a view pager, you have to add a unique linear back stack in `PagerAdapter#instantiateItem(ViewGroup, int)`. This method comes with a container and it may be tempting to use this container as the root container of the linear back stack but this won't work properly. View pagers don't like it when views are added or removed from the container independent of the adapter. To remedy this, we need to add an extra view to be the container of our linear back stacks.

```Java
@Override
public Object instantiateItem(ViewGroup container, int position) {
    // View Pagers doesn't like it when you remove a view from the container.
    // We need to add an extra view so that we can add and remove views to it at will
    ViewGroup parent = new RelativeLayout(container.getContext());
    container.addView(parent);

    //Just being lazy. If you're inflating unique view groups, you should store a unique tag
    //in each one
    String tag = TAG + position;

    backStackManager.createLinearBackStack(tag, parent, (layoutInflater, viewGroup) -> {
        ViewGroup vg = (ViewGroup) layoutInflater.inflate(R.layout.view_group1, viewGroup, false);
        viewGroup.addView(vg);
        return vg;
    });

    //Remember to actually add the linear back stack to the split back stack
    splitBackStack.add(position, tag);

    return parent;
}
```

Additionally we need to override `PagerAdapter#destroyItem(ViewGroup, int, Object)` to remove our container view group and destroy our added linear back stacks.

```Java
@Override
public void destroyItem(ViewGroup container, int position, Object object) {
    //If you remove a view you must call splitBackStack#remove(int)
    splitBackStack.remove(position);

    //Standard view pager adapter procedure
    container.removeView((ViewGroup) object);
}
```

Overall the class looks like this:

```Java
public class ViewPagerAdapter extends PagerAdapter {

    public static final int NUM_PAGES = 3;
    public static final String SPLIT_TAG = "SPLIT_TAG";
    public static final String TAG = "TAG";

    private BackStackManager backStackManager = BackStack.getBackStackManager();
    private SplitBackStack splitBackStack = backStackManager.createSplitBackStack(SPLIT_TAG, 0);

    @Override
    public Object instantiateItem(ViewGroup container, int position) {
        // View Pagers doesn't like it when you remove a view from the container.
        // We need to add an extra view so that we can add and remove views to it at will
        ViewGroup parent = new RelativeLayout(container.getContext());
        container.addView(parent);

        //Just being lazy. If you're inflating unique view groups, you should store a unique tag
        //in each one
        String tag = TAG + position;

        backStackManager.createLinearBackStack(tag, parent, (layoutInflater, viewGroup) -> {
            ViewGroup vg = (ViewGroup) layoutInflater.inflate(R.layout.view_group1, viewGroup, false);
            viewGroup.addView(vg);
            return vg;
        });

        //Remember to actually add the linear back stack to the split back stack
        splitBackStack.add(position, tag);

        return parent;
    }

    @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        //If you remove a view you must call splitBackStack#remove(int)
        splitBackStack.remove(position);

        //Standard view pager adapter procedure
        container.removeView((ViewGroup) object);
    }

    @Override
    public int getCount() {
        return NUM_PAGES;
    }

    @Override
    public boolean isViewFromObject(View view, Object object) {
        return view == object;
    }
}
```

We also need to make the split back stack aware of page changes. This way it knows which linear back stack should receive the go back calls.

```Java
viewPager.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {
    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {

    }

    @Override
    public void onPageSelected(int position) {
        ((SplitBackStack) BackStack.getStack(SPLIT_TAG)).changePosition(position);
    }

    @Override
    public void onPageScrollStateChanged(int state) {

    }
});
```

Your split back stack should now work properly with the view pager. Under this configuration, each page will have it's own independent back stack.

#### Bottom bar

Split back stacks also works with bottom bars. The developer is responsible for adding all the view groups needed and making an independent linear back stack for each tab. They are also responsible for switching view groups and alerting the split back stack when the user presses a new tab.

Let's make a bottom bar with 3 tabs. Config the bottom bar as normal using a menu xml file. For this demo we're going to keep all three tabs visible and we're going to use `ViewGroup#bringChildToFront(View)` to switch which child is shown.
```Java
BackStack.install(this);
backStackManager = BackStack.getBackStackManager();
splitBackStack = backStackManager.createSplitBackStack(SPLIT_TAG, 0);

root = findViewById(R.id.root);
bottomBar = findViewById(R.id.bottom_bar);

backStackManager.createLinearBackStack(ViewGroup1.TAG, root, (layoutInflater, container)->{
   ViewGroup1 viewGroup = new ViewGroup1(layoutInflater.getContext());
   container.addView(viewGroup);
   return viewGroup;
});
splitBackStack.add(0, ViewGroup1.TAG);

backStackManager.createLinearBackStack(ViewGroup2.TAG, root, (layoutInflater, container) -> {
    ViewGroup2 viewGroup = new ViewGroup2(layoutInflater.getContext());
    container.addView(viewGroup);
    return viewGroup;
});
splitBackStack.add(1, ViewGroup2.TAG);

backStackManager.createLinearBackStack(ViewGroup3.TAG, root, (layoutInflater, container) -> {
    ViewGroup3 viewGroup = new ViewGroup3(layoutInflater.getContext());
    container.addView(viewGroup);
    return viewGroup;
});
splitBackStack.add(2, ViewGroup3.TAG);

root.bringChildToFront(((LinearBackStack) backStackManager.getStack(ViewGroup1.TAG)).getCurrentView());
```

Next we're going to add a on tab change listener to the bottom bar. This will alert the split back stack which position the bar is currently at. In this method, we're also going to bring the selected view to the front so that it's shown.

```Java
bottomBar.setOnNavigationItemSelectedListener(new BottomNavigationView.OnNavigationItemSelectedListener() {
    @Override
    public boolean onNavigationItemSelected(@NonNull MenuItem item) {
        //Couldn't find a way of getting the position. Had to do this instead
        int position = 0;
        switch (item.getItemId()){
            case R.id.first:position = 0; break;
            case R.id.second: position = 1; break;
            case R.id.third: position = 2; break;
        }

        root.bringChildToFront(((LinearBackStack)BackStack.getStack(splitBackStack.getTAG(position)))
                .getCurrentView());
        splitBackStack.changePosition(position);

        return true;
    }
});
```
