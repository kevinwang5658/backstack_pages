# BackStack

BackStack is a light weight back stack library for view based navigation on Android. Using BackStack provides an easy way to switch between view groups without having to worry about back navigation. A navigation tree built by BackStack will also be persisted through rotation and activity restarts. BackStack works with linear navigation flows as well as split flows like bottom bars and view pagers.

## Philosophy

#### History

The navigation style that BackStack uses is view group based navigation. This means that instead of using Activities and Fragments to provide different screens, a light-weight ViewGroup is used instead. Over the years, there have been many discussions on this subject. If you want to go through it, a summary of the history can be found [here](https://academy.realm.io/posts/michael-yotive-state-of-fragments-2017/?).

Google officially recommends the use of fragments. However, people in the android community, myself included, find that fragments are large bloated objects that obscures a lot it's functionality behind layers of code. There have also been problem reported weird memory leaks and problems with the asynchronous nature of fragment transactions. The consensus in the community is that people generally don't like using fragments.

In 2014 square began a movement,  [here](https://medium.com/square-corner-blog/advocating-against-android-fragments-81fd0b462c97), to use view groups in place of fragments. They argued that view groups are light weight containers that adhere to the Android lifecycle, something that seems perfect to use as a container for a screen. View groups, however have their own limitations. Mainly that, they don't inherently come back navigation and persistence through rotations. This is where BackStack comes in.

#### BackStack

BackStack is view group based back navigation library. It aims to overcome the limitations of view groups and make them a viable choice for use in-place of fragments or activities. BackStack is built around the concept of a view creator, a contextless object that does what it's name says, creates views. Each view creator is associated with a particular view group. View creators don't hold state about what they create or who told them to create what. By doing this, each view group can be inflated independent of each other regardless of the order or conditions they were first created in. BackStack stores a stack of view creators in chronological order. The view creator at the top of the stack will inflate the view that is actually shown to the user. Using this stack, BackStack can return to any previous views. View creators are contextless, and therefore they can also be persisted through activity restarts without leaking.

## Using BackStack

```Java
public static class MyViewCreator implements ViewCreator{
    @Override
    public ViewGroup create(LayoutInflater layoutInflater, ViewGroup container) {
        CustomViewGroup vg = new CustomViewGroup(layoutInflater.getContext());
        container.addView(vg);
        return vg;
    }
}
...

public void foo(){
    linearBackStack.add(new MyViewCreator());
}
```

View creators are very simple to make. They are simply classes that implement the ViewCreator interface. All view creators must adhere to 2 rules. First, by the end of the create function the inflated view group must be added to the parent container, sort of like in a ListView . Secondly, the class must not capture any references to the Android context. This means that only regular classes, static inner classes and lambdas that don't explicitly capture are eligible to be view creators; anonymous inner classes can't be used because they implicitly capture. This is done so that the view creator doesn't leak when it's persisted.

```Java
linearBackStack.add((layoutInflater, container) -> {
     CustomViewGroup vg = new CustomViewGroup(layoutInflater.getContext());
     container.addView(vg);
     return vg;
 });
```

When using this library, it is highly recommended that you [enable Java 8 language features](https://developer.android.com/studio/write/java8-support.html). This allows you to use lambdas which greatly simplifies the code and improves the readability.

For more information please visit the [documentation](getting_started.md).

## Getting BackStack

Using Gradle:

build.gradle (Project)
```Gradle
allprojects {
    repositories {
        jcenter()
        maven{
            url 'https://jitpack.io'
        }
    }
}
```

build.gradle (Module)
```Gradle
dependencies {
    compile 'com.github.kevinwang5658:backstack:v2.3'
}
```
