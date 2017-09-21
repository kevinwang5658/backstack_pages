# View Creators

View creators is the core of the BackStack library. They allow the library to instantiate any view group at any time regardless of the order and conditions they were added in. A view creator is just a functional interface that the developer will have to implement. View creators have 2 rules that developers must abide by. First, by the end of the create function, the inflated view must be added to the provided container. Second, the view creator should not capture a reference to the activity in any way. This means that anonymous inner functions can't be used since they implicitly capture the surrounding outer class. Lambdas don't implicitly capture and can be used.

Here's some tips and tricks with view creators.

## Inflating xml

Instead of creating a creating a custom view group through code and then inflating a layout xml inside of the constructor, you can directly inflate an xml with the custom view group as a base. This prevents an unnecessary container from being added.

ViewGroup layout xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<com.rievo.test2.ViewGroup1
   xmlns:android="http://schemas.android.com/apk/res/android"
   android:orientation="vertical" android:layout_width="match_parent"
   android:layout_height="match_parent"
   android:background="@android:color/holo_orange_light">
   <TextView
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:text="1"/>

</com.rievo.test2.ViewGroup1>
```

Inflation code
```Java
linearBackStack.add((layoutInflater, container) -> {
   ViewGroup viewGroup = (ViewGroup) layoutInflater.inflate(R.layout.view_group1, container, false);
   container.addView(viewGroup);
   return viewGroup;
});
```

## Storing State

If inflating using a constructor, it may be useful to pass state using the view group's constructor. This can't be done when using fragments or activities which require you use the default constructor. Passing in state  is useful in situations where you want to pass in information for the new view group to display, such as an information page. In order to not capture the activity, a standalone class or a static inner class must be used.

```Java
public static class MyViewCreator implements ViewCreator{

    Object state;

    MyViewCreator(Object myState){
        state = myState;
    }

    @Override
    public ViewGroup create(LayoutInflater layoutInflater, ViewGroup container) {
        ViewGroup1 viewGroup = new ViewGroup1(layoutInflater.getContext(), state);
        container.addView(viewGroup);
        return viewGroup;
    }
}
```
