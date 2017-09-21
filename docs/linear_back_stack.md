# Linear Back Stack

Linear back stacks are the core of BackStack. They are responsible for linear navigation flow `screenA -> screenB -> screen C`. Linear back stacks have a root container view which it adds all view groups to. Under default behavior, linear back stacks will replace the previous view each time a new view is added, though this can be changed.

## Initialization

Linear back stacks can be initialized in two ways. The first way is to use `BackStackManager#createLinearBackStack(String, ViewGroup, ViewCreator)`. This will initialize a linear back stack with a tag String, parent container ViewGroup and the ViewCreator for the first screen. All back stacks are associated with a unique tag. This facilitates access from anywhere in the code. Another way of initializing a linear back stack is to use a builder. This provides a lot more options on the configuration. To get the builder, call `BackStackManager#builder(String)`.

The options for the builder include:

`setContainer(ViewGroup)`: Sets the parent container

`useCurrentViewGroup(ViewGroup)`: Use the current view group as the first view and deduce the parent container view. This shouldn't be used alongside `setContainer(ViewGroup)`

`setRetain(boolean)`: Sets the retain mode of the first view. Retained views won't be destroyed when more views are added, useful for expensive view groups.

`shouldAllowDuplicates(boolean)`: Sets whether duplicate view creators should be allowed. View creator equality is determined by the equals function of each view creator. Make sure that is implemented properly for this to work.

`viewCreator(ViewCreator)`: The view creator of the first view group on the stack. This must be set.

```Java
backStackManager.createLinearBackStack(TAG, container, (layoutInflater, container) -> {
    return null;
});

backStackManager.builder(TAG)
        .setContainer(container)
        //You shouldn't use both setContainer and useCurrentViewGroup at the same time
        .useCurrentViewGroup(container)
        .shouldAllowDuplicates(false)
        .shouldRetain(false)
        .viewCreator((layoutInflater, container) ->{
            return null;
        }).build();
```

## Adding Subsequent Views

From anywhere in the app, all linear back stacks are accessible using `BackStack#getStack(TAG)`. After retrieval, linear back stacks have three options for adding new views, two overloaded methods and a builder.

The easiest way to add a view is to use `LinearBackStack#add(ViewCreator)`. This will push a view creator onto the stack and use it to inflate the view group. You can also specify a unique container to add the new view group using `LinearBackStack#add(ViewCreator, ViewGroup)`. The container must be on the same branch of and equal or lower down on the view hierarchy than the default container (it must be a parent of the default container or the default container itself). There is also a builder which provides the most options. To access the builder use `LinearBackStack#Builder(ViewCreator)`.

The options for the builder include:

`setRetain(boolean)`: Sets the retain mode of the newly added view. Retained views won't be destroyed when more views are added, useful for expensive view groups.

`setContainer(ViewGroup)`: Sets a custom container to add the view group in. The container must be on the same branch of and equal or lower down on the hierarchy than the default parent view group.

`addAnimator(Animator)`: Sets the add animator. This will be called when new views are added. The `Animator#animate(ViewGroup, Emitter)` method has two parameters: view, the view to play the animation on and emitter, something you have call when the animation ends.

`removeAnimator(Animator)`: Sets the remove animator. This animator is called when back is pressed and a view is removed. The rules are the same as the addAnimator.

```Java
LinearBackStack linearBackStack = (LinearBackStack) BackStack.getStack(TAG);

linearBackStack.add((layoutInflater, viewGroup) -> {
    return null;
});

linearBackStack.add((layoutInflater, viewGroup) -> {
    return null;
}, container);

linearBackStack.builder((layoutInflater, viewGroup) -> {
    return null;
})
        .setRetain(true)
        .setContainer(container)
        .addAnimator((viewGroup, emitter) -> {
            emitter.done();
        })
        .removeAnimator((viewGroup, emitter) -> {
            emitter.done();
        })
        .build();
```

## Handling onBack Events Yourself

To handle onBack events yourself, implement the Reversible interface in the view group that you want to handle the call with. When that view group is about to be removed from the stack because of a onBack call, the `Reversible#onBack` will be called to give the view a chance to handle the call. Returning true from this method means that you have handled the call and that BackStack doesn't need to do anything. Returning false means that you want BackStack to still handle the back event.

```Java
public class ViewGroup1 extends RelativeLayout implements Reversible{
    public ViewGroup1(Context context) {
        super(context);
    }

    @Override
    public boolean onBack() {
        return false;
    }
}
```
