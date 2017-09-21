# Getting Started

## Adding BackStack

Using gradle:

build.gradle (Project)
~~~Gradle
allprojects{
    repositories{
        jcenter()
        maven{
            url 'https://jitpack.io'
        }
    }
}
~~~

build.gradle (Module)
~~~Gradle
dependencies{
    compile 'com.github.kevinwang5658:backstack:v2.3'
}
~~~

## Initialization

To initialize BackStack simply add the following code in onCreate().

~~~Java
public class MainActivity extends AppCompatActivity {

    BackStackManager backStackManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...

        BackStack.install(this);
        backStackManager = BackStack.getBackStackManager();
    }

    @Override
    public void onBackPressed() {
        if (!backStackManager.goBack()) {
            super.onBackPressed();
        }
    }
}
~~~

## Creating a Simple Back Stack

To begin we're going to focus on getting a linear back stack up and running. Linear back stacks are the main back stacks of this library. They are responsible for linear navigation flow `screenA -> screenB -> screenC`. Linear back stacks will automatically implement back navigation and view hierarchy persistence.

Let's start shall we?
~~~Java
public static final String TAG = "TAG";

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    BackStack.install(this);
    backStackManager = BackStack.getBackStackManager();

    //This is the container you want to be the root of your back stack
    root = findViewById(R.id.root);

    LinearBackStack linearBackStack = backStackManager.createLinearBackStack("TAG", root, (layoutInflater, container) -> {
        //This is our first view group in the stack
        CustomViewGroup cvg = new CustomViewGroup(layoutInflater.getContext());

        //Make sure that the view is added to container by the end of this block
        container.addView(cvg);

        //Return the view group that was newly inflated
        return cvg;
    });
}
~~~

Let's make a view group called CustomViewGroup.

~~~Java
public class CustomViewGroup extends RelativeLayout{
    public CustomViewGroup(Context context) {
        super(context);

        //Let's set the background color so that we know
        //that the view group has been added
        setBackgroundColor(Color.GRAY);

        //Let's inflate an xml file with the children of this view group
        LayoutInflater.from(context).inflate(R.layout.custom_view_group, this, true);
    }
}
~~~

And that's it. If you run this, you should see a gray screen instead of the default white. The first view group has been added to your stack.

## Adding Subsequent View Groups

Now let's say that this gray screen is the home of our app, and that when we click on something we want to change to the next screen. Let's simulate this by setting an onClick listener on the entire screen.

~~~Java
public class CustomViewGroup extends RelativeLayout{
    public CustomViewGroup(Context context) {
        super(context);

        //Let's set the background color so that we know
        //that the view group has been added
        setBackgroundColor(Color.GRAY);

        //Let's inflate an xml file with the children of this view group
        LayoutInflater.from(context).inflate(R.layout.custom_view_group, this, true);

        setOnClickListener(v -> {

        });

    }
}
~~~

This time let's inflate the view through xml as well just to change things up. So let's first create an xml layout file called added_view.xml

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<com.rievo.test.AddedViewGroup xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/white">
    <!-- Let's set the background color to white so that we can see a change-->
</com.rievo.test.AddedViewGroup>
~~~

Let's change our on click listener
~~~Java
setOnClickListener(v -> {
    //All back stacks can be retrieved anywhere using their tag
    LinearBackStack linearBackStack = (LinearBackStack) BackStack.getStack(MainActivity.TAG);
    linearBackStack.add((layoutInflater, container) -> {
        //if attach to root is true layoutInflater.inflate() returns the container instead
        ViewGroup viewGroup = (ViewGroup) layoutInflater
                .inflate(R.layout.added_view, container, false);
        container.addView(viewGroup);
        return viewGroup;
    });
});
~~~

Now let's try it. Clicking on the screen should change the screen color to white. Let's say this represents an info page or something. If you rotate your device you'll see that the screen remains white, this means that navigation stack was persisted. Pressing back should lead you back to the home screen. For more advanced usage of this library please proceed to the [next section](linear_back_stack.md).
