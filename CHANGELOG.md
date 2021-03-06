# Change log

-Simple Stack 1.9.1 (2018-XX-XX)
--------------------------------
- Fix: `History.single()` should return `History<T>`, not `List<T>`.

-Simple Stack 1.9.0 (2018-03-04)
--------------------------------

- DEPRECATED: HistoryBuilder's factory methods are moved from HistoryBuilder to the newly added `History` class.

`HistoryBuilder.from(T... objects)` -> `History.builderOf(T... objects)`

`HistoryBuilder.from(...)` -> `History.builderFrom(...)`

`HistoryBuilder.single()` -> `History.single()`

`HistoryBuilder.newBuilder()` -> `History.newBuilder()`

Also adds `History.of(T... objects)` instead of `HistoryBuilder.of(...).build()`.

- ADDED: `History` class, an immutable list with additional operations over `List<T>` - as some methods' return type.

I also changed some return types (in history building) from `List<Object>` to `<T> List<T>`,
this resulted in having to change some `List<Object>`s to `List<?>` on the receiving side if used in assignment.

So if you run into that, just change `List<Object>` to `List<?>` and it should work.

- ADDED: Long-overdue empty constructor for `BackstackDelegate` and `backstack.setStateChanger(StateChanger)`.

- UPDATE: Kotlin sample replaces `PaperParcel` with `@Parcelize`.

-Simple Stack 1.8.2 (2018-01-20)
--------------------------------

- CRITICAL FIX: `1.8.1 (2018-01-17)` didn't retrieve `state-bundle 1.2.0` as transitive dependency because of `com.github.dcendents:android-maven-gradle-plugin:1.5`. It is updated to `2.0` to fix publishing to Jitpack.

- ADDED / CHANGE: `Navigator.findActivity(Context)` is now public. It also casts the returned Activity to whatever subclass type is expected.

- ADDED: `Backstack.fromTop(int offset)` method, which provides the element in the backstack from the top with a given offset. 

- Updated State-Bundle to 1.2.0.

- Updated to use implementation/api and AS 3.0's tooling.

- Updated implementation lib versions in samples and tests.

- Updated Kotlin example to use Fragment-based usage.

-Simple Stack 1.8.0 (2017-10-25) 
--------------------------------

- BREAKING(?) CHANGE / FIX: when `goBack()` returns false, then the backstack is not cleared automatically. Added `reset()` to allow imitating the previous behavior. 

Previous behavior would now be the same as:

    if(!backstack.goBack() && !backstack.isStateChangePending()) {
        backstack.reset();
    }
    
Honestly, this might have been unexpected, as `goBack()` returning `false` had the side-effect of clearing the stack, and next state change using the initial key!

The test that checks for this has been changed to use the above construct. Another test of course has been added to validate new behavior.

Also, to eliminate the possibility of `reset()` misuse, it is only allowed when there are no pending state changes.

- BREAKING CHANGE: `getInitialParameters()` is renamed to `getInitialKeys()`. 

- FIX: `getInitialParameters()` returned the actual list instead of an unmodifiable copy, it returns the keys provided at initialization.

- ADDED: `replaceTop()` backstack operator, which replaces current top with the provided key.

- ADDED: `goUp()` backstack operator, which will go back to the provided key if exists, replace top with new key otherwise.

- ADDED: `goUpChain()` backstack operator, which will:

  - If the chain of parents is found as previous elements, then it works as back navigation to that chain.
  - If the whole chain is not found, but at least one element of it is found, then the history is kept up to that point, then the chain is added, any duplicate element in the chain is added to the end as part of the chain.
  - If none of the chain is found, the current top is removed, and the provided parent chain is added.
  
  I added a bunch of tests for this, hopefully I didn't forget anything!

- ENHANCEMENT/FIX: `HistoryBuilder.get()` is now `@NonNull`, because `HistoryBuilder.from(List<?>)` throws if List contains `null`.

- ENHANCEMENT: `getHistory()` and `getInitialParameters()`  also returns a `List<T>` in which each element is cast to `T`. 

- FIX: `BackstackDelegate.setStateChanger()` should have been allowed even without calling `backstackDelegate.onCreate()` first. All the samples use `new BackstackDelegate(null)` so it never came up.

- ENHANCEMENT: Some improvement to `persistViewToState()` exception message if the view has no key (so that it references `KeyContextWrapper` and `stateChange.createContext()`).

-Simple Stack 1.7.2 (2017-07-24)
--------------------------------
- MINOR CHANGE + ENHANCEMENT: `StateChange.getNewState()` and `StateChange.getPreviousState()` return a copy of the list (it was already a copy, don't worry), where each item is casted to `<T>` specified as generic parameter.

For example, the following can be changed from:

``` java
for(Object _newKey : stateChange.getNewState()) {
    Key newKey = (Key)_newKey;
    // ...
}
```

to:

``` java
for(Key newKey : stateChange.<Key>getNewState()) {
   // ...
}
```

And the following works now as well:

``` java
List<Key> newKeys = stateChange.getNewState(); // used to return List<Object>
```

-Simple Stack 1.7.1 (2017-07-11)
--------------------------------
- ADDED: `BackstackDelegate.registerForLifecycleCallbacks(Activity)` convenience method (API 14+).

This method allows you to call this after `BackstackDelegate.onCreate()`, after which the following 4 methods no longer need to be called manually:

    - `onPostResume()`
    - `onPause()`
    - `onSaveInstanceState(Bundle)`
    - `onDestroy()`

Therefore the callbacks that ought to be called remain as `onCreate()`, `onRetainCustomNonConfigurationInstance()`, and of course `onBackPressed()`.

-Simple Stack 1.7.0 (2017-07-04)
--------------------------------
- REMOVED: `BackstackManager.StateChangeCompletionListener`. It is replaced by `Backstack.CompletionListener`, which was added back in 0.9.1 (and is more reliable).

This also fixes a possible bug with incorrect call order of state change completion listeners.

The API is otherwise exactly the same, `StateChangeCompletionListener` should have been `Backstack.CompletionListener` from the start.

- ADDED: `backstackDelegate.getManager()`, just to make sure its API mirrors `Navigator`.

-Simple Stack 1.6.3 (2017-06-28)
--------------------------------
- Added missing `@NonNull` and `@Nullable` annotations where applicable.

-Simple Stack 1.6.2 (2017-05-14)
--------------------------------
- MINOR CHANGE: `DefaultStateChanger` no longer explicitly demands a `StateKey`, because both `LayoutInflationStrategy` and `GetViewChangeHandlerStrategy` can be re-defined for custom behavior.

- Added `GetViewChangeHandlerStrategy` to `DefaultStateChanger` to allow re-defining the view change handler behavior.

- Added `ContextCreationStrategy` to `DefaultStateChanger` to support Mortar scopes, or anything similar in design.

- Added `BackstackManager.StateChangeCompletionListener` to add a hook where you can listen for the completion of state changes reliably - even if they were forced to execute.

Also added `addStateChangeCompletionListener` to `BackstackDelegate` and `Navigator.Installer` accordingly. 

Please make sure it does not retain a reference to enclosing Activity, to avoid memory leaks.

- Minor fix: `setKeyFilter()`, `setKeyParceler()`, and `setStateClearStrategy()` in `BackstackDelegate` now throw if they are set after calling `onCreate()`, as per docs.

- Bump `state-bundle` version to `1.1.5`

- ADDED: `simple-stack-mortar-sample`, which is based on `mortar-sample` from `square/mortar`, but using `Simple-Stack` and `Service-Tree`.

-Simple Stack 1.6.1 (2017-05-08)
--------------------------------
- Added `GetPreviousViewStrategy` to `DefaultStateChanger` as per request (#36).

-Simple Stack 1.6.0 (2017-05-03)
--------------------------------
- Added `KeyFilter` to allow clearing out keys on process death that should not be restored.
- Added `stateChange.backstack()` which returns the backstack this state change was executed by.
- Added `DefaultStateChanger.ViewChangeStartListener` for *before* the view change, but *after* the state restore
- Added `DefaultStateChanger.LayoutInflationStrategy` to support asynchronous layout inflation (if you need it) or hopefully Anko

-Simple Stack 1.5.3 (2017-04-21)
--------------------------------
- Bump `state-bundle` version to `1.1.4`

-Simple Stack 1.5.2 (2017-04-12)
--------------------------------
- Bump `state-bundle` version to `1.1.0`

-Simple Stack 1.5.1 (2017-04-07)
--------------------------------
- Added a method to `DefaultStateChanger` to allow perform view change with an externally specified direction.

-Simple Stack 1.5.0 (2017-04-07)
--------------------------------
- Merged `zhuinden/navigator` into `zhuinden/simple-stack`.

- ADDED: `Navigator` class as an optional replacement for `BackstackDelegate` (API 11+).

`BackstackDelegate` had a lot of callbacks to remember (`onPause()`, `onResume()`, `onRetainCustomNonConfigurationInstance()`, `onDestroy()`),
but more importantly you had to manage saving out the current view's state in `onSaveInstanceState()` manually.

With Navigator, this is all hidden in the `BackstackHost` installed by `Navigator.install()` (or `Navigator.configure()...install()`, so this problem is solved for you.

- ADDED: `DefaultStateChanger` that by default uses **`Navigator`-based installation**.

To use `DefaultStateChanger` with `BackstackDelegate`, you must provide `DefaultStateChanger.configure().setStatePersistenceStrategy()` and delegate persistence calls to your delegate.

- ADDED: `StateKey` interface used by `DefaultStateChanger`.

- ADDED: default view change handlers for `DefaultStateChanger`.

- All examples (**except** the `fragment` and `multistack` samples) were updated to use `Navigator`.

- `simple-stack-example-mvp` no longer uses `square/coordinator`, it uses custom viewgroups instead.

This is because `Coordinator` gets created only after `container.addView()`, which makes it hard to work with.

-Simple Stack 1.4.4 (2017-03-28)
--------------------------------
- ADDED: `backstack.top()` method that returns the last element in the backstack or `null`

-Simple Stack 1.4.3 (2017-03-25)
--------------------------------
- FIX: Fixed a bug that if a restored backstack is cleared and an initialize state change is triggered,
       then the restored keys were used instead of the initial key
    (this only surfaced if you attempt to use multiple backstacks, and a cleared backstack is re-used)

-Simple Stack 1.4.2 (2017-03-20)
--------------------------------
- CHANGE: Decreased minSDK to 1.

-Simple Stack 1.4.1 (2017-03-09)
--------------------------------
- MINOR FIX: Adjusted exception message in `BackstackManager` to say `setup()`.

-Simple Stack 1.4.0 (2017-03-06)
--------------------------------
- BREAKING CHANGE: `StateBundle` is moved from `com.zhuinden.simplestack.StateBundle` to `com.zhuinden.statebundle.StateBundle`

- CHANGE: `StateBundle` is moved to https://github.com/Zhuinden/state-bundle and is a compile dependency of `simple-stack`

-Simple Stack 1.3.0 (2017-03-03)
--------------------------------
- REFACTOR: `BackstackDelegate` is separated into `BackstackManager`.

- ADDED: `BackstackManager` class to help with creating backstacks inside views.

- ENHANCEMENT: `BackstackManager` is now `Bundleable`, therefore its state can be automatically restored along with other managed services (see examples).

- CHANGE: `clearStatesNotIn()` is no longer a method of `BackstackDelegate` or `BackstackManager`, it can be specified as custom using a `BackstackManager.StateClearStrategy`.

- ADDED: `simple-stack-example-services` that shows how to use `service-tree` with `simple-stack` to store scoped services that can have their states restored and survive configuration change.

- ADDED: `simple-stack-example-nestedstack` that shows how to use view-level `BackstackManager` stored in `service-tree` to have nested backstacks inside views.

-Simple Stack 1.2.1 (2017-02-28)
--------------------------------
- FIX: `HistoryBuilder` should receive `List<?>` as parameters, not `List<Object>` on all methods.

-Simple Stack 1.2.0 (2017-02-28)
--------------------------------
- BREAKING CHANGE: `Bundleable` and `SavedState` now use `StateBundle` class.

- ENHANCEMENT: Added `StateBundle` class to replace `android.os.Bundle`.

-Simple Stack 1.1.1 (2017-02-19)
--------------------------------
- FIX: A bug that allowed the possibility that an uninitialized backstack could restore its history to be an empty list after process death.

- ADDED: `simple-stack-example-multistack` for handling multiple backstacks in the same Activity using a BottomNavigationView.

-Simple Stack 1.1.0 (2017-02-18)
--------------------------------
- BREAKING CHANGE: `Backstack`'s APIs return `Object` instead of `Parcelable` (that includes `StateChange`, initial keys, `HistoryBuilder`, etc.)

- ENHANCEMENT: Added `KeyParceler` interface to allow defining custom strategy in order to turn keys into `Parcelable` (for example, using `Parceler` library instead)

-Simple Stack 1.0.0 (2017-02-15)
--------------------------------
- RELEASE: 1.0.0!

- ENHANCEMENT: Javadoc for all public classes and methods.

-Simple Stack 0.9.6 (2017-02-14)
---------------------------------
- INTERNAL CHANGE: Hidden `stateChangeComplete()` from public API of `BackstackDelegate`, it shouldn't have been public.

-Simple Stack 0.9.5 (2017-02-13)
---------------------------------
- INTERNAL CHANGE: `clearStatesNotIn()` now receives both `keyStateMap` and `StateChange`, instead of just the new state.

- ENHANCEMENT: Added `HistoryBuilder.from(Backstack)` and `HistoryBuilder.from(BackstackDelegate)` convenience methods.

- ENHANCEMENT: Added `HistoryBuilder.isEmpty()` method, and implements `Iterable<Parcelable>`.

- ADDED: `flow-masterdetail-fragments` example.

- FIX: A bug in `flow-masterdetail` sample that prevented Master's state from being persisted if detail directly opens a detail.

-Simple Stack 0.9.3 (2017-02-12)
---------------------------------
- ENHANCEMENT: Added ability to force execute pending state changes with `Backstack.executePendingStateChange()`.

- INTERNAL CHANGE: `BackstackDelegate.onDestroy()` calls `backstack.executePendingStateChange()` to prevent hanging state changes.

- ADDED: `ObjectAnimator`-based segue animation to MVP example.

-Simple Stack 0.9.2 (2017-02-11)
---------------------------------
- BREAKING CHANGE(?): `CompletionListener` no longer receives `isPending` parameter.

- ADDED: `Backstack.isStateChangePending()` to replace `isPending`.

- ENHANCEMENT: Added some missing `@NonNull` and `@Nullable` annotations.

- ADDED: Apache license notes, and improved the README.

-Simple Stack 0.9.1 (2017-02-09)
---------------------------------
- BREAKING CHANGE(!): `BackstackDelegate` has a new method which **must be called**: `backstackDelegate.onDestroy()`
    Not calling `backstackDelegate.onDestroy()` will most likely result in memory leak, so please make sure you call it paired with `onCreate()`.

- BREAKING CHANGE: `BackstackDelegate.clearStatesNotIn()` is no longer public, because it is automatically managed on state change completion.

- ENHANCEMENT: Added `Backstack.CompletionListener` which listens to when backstack has completed a state change.
    Added `Backstack.addCompletionListener()` and `Backstack.removeCompletionListener()` methods.
    The backstack keeps a strong reference to your completion listener, so make sure you remove your change listener when no longer needed.

- ENHANCEMENT: It is no longer the responsibility of the `StateChanger` to call `backstackDelegate.clearStatesNotIn()`.
    The `BackstackDelegate` registers itself as a `CompletionListener`, and therefore it can call `clearStatesNotIn()` automatically.

- ENHANCEMENT: Added `flow-sample` changed to use Simple-Stack, as name `simple-stack-flow-masterdetail`.

-Simple Stack 0.8.3 (2017-02-04)
---------------------------------
- ENHANCEMENT: Added `BackstackDelegate.setPersistenceTag(String)` for support of multiple backstacks. It must be called before `BackstackDelegate.onCreate()`.

-Simple Stack 0.8.2 (2017-02-03)
---------------------------------
- CHANGE: `KeyContextWrapper` is public again
- ENHANCEMENT: Created `fragments` example based on `mvp` example.

-Simple Stack 0.8.1 (2017-02-02)
---------------------------------
- BREAKING(?) CHANGE: Renamed `HistoryBuilder.peek()` to `HistoryBuilder.getLast()`
- ENHANCEMENT: Added the following new methods to `HistoryBuilder`:
    `HistoryBuilder.get(index)`,
    `HistoryBuilder.contains(key)`,
    `HistoryBuilder.containsAll(keys)`,
    `HistoryBuilder.add(key, index)`,
    `HistoryBuilder.size()`,
    `HistoryBuilder.removeAt(index)`,
    `HistoryBuilder.remove(key)`,
    `HistoryBuilder.clear()`,
    `HistoryBuilder.retainAll(keys)`,
    `HistoryBuilder.indexOf(key)`,

-Simple Stack 0.8.0 (2017-02-02)
---------------------------------
- BREAKING CHANGE: Removed `StateChange.Direction`, it is now an `int` annotated with `@IntDef`.
This means that `StateChange.Direction.FORWARD` is now `StateChange.FORWARD`, same for `BACKWARD` and `REPLACE`.
- Fix: `@StateChangerRegisterMode` shouldn't have been public

-Simple Stack 0.7.0 (2017-01-31)
---------------------------------
- BREAKING CHANGE: Removed `Backstack.get(Context)`, `BackstackDelegate.isSystemService(String)` and `BackstackDelegate.getSystemService(Context)`.

These can be easily done manually with the following setup:

``` java
    public static Backstack get(Context context) {
        // noinspection ResourceType
        return (Backstack)context.getSystemService(BACKSTACK);
    }
```

and

``` java
    @Override
    public Object getSystemService(String name) {
        if(name.equals(BACKSTACK)) {
            return backstackDelegate.getBackstack();
        }
        return super.getSystemService(name);
    }
```

Therefore the preferred solution is to provide the `Backstack` instance via `@Inject` instead of `Backstack.get(Context)`.

Example for `Backstack.get(Context)` was moved to `simple-stack-example` as `BackstackService`.
Example for `@Inject Backstack backstack;` is seen in `simple-stack-example-mvp`.

-Simple Stack 0.6.1 (2017-01-27)
---------------------------------
- It is now allowed to initialize `BackstackDelegate` without a `StateChanger`, in which case `setStateChanger()` must be called before `onPostResume()`.`
This way it is possible to postpone the initialization state change of the `Backstack`.

-Simple Stack 0.6.0 (2017-01-23)
---------------------------------
- **Simple Stack is now a library!**
- Added `BackstackDelegate.getBackstack()` for convenience over `Backstack.get(this)` in Activity

-Simple Stack 0.5.1 (2017-01-23)
---------------------------------
- Added `Bundleable` interface to allow saving view's state to Bundle
- Added `BackstackDelegate.restoreViewFromState()` method to mirror `persistViewToState()`
- `getSavedState()` now returns a new `SavedState` instead of throwing error if the key has no state bound to it
- Added `SavedState.viewHierarchyState` default value `new SparseArray<>()`, null is prohibited

-Simple Stack 0.5.0 (2017-01-22)
---------------------------------
- Added `BackstackDelegate` class to hide activity lifecycle integration
- Moved `SavedState` into library
- Added `Backstack.get(Context)` method to obtain Backstack of instance shared by the delegate
- Moved `KeyContextWrapper` into library, and it is now package-private
- Added `StateChange.createContext(base, key)` method to create `KeyContextWrapper`
- `KeyContextWrapper.getKey(Context)` is now `Backstack.getKey(Context)`

-Simple Stack 0.4.0 (2017-01-21)
---------------------------------
- Rename packages from `demostack` to `simplestack`

-Simple Stack 0.3.3 (2017-01-21)
---------------------------------
- Rename `State` to `SavedState`

-Simple Stack 0.3.2 (2017-01-21)
---------------------------------
- Add check for if `key` is `null` in `State`'s `Builder.build()`

-Simple Stack 0.3.1 (2017-01-20)
---------------------------------
- Added missing `equals()`/`hashCode()` to `State` class in example

-Simple Stack 0.3.0 (2017-01-20)
---------------------------------
- Added view persistence to example code (`MainActivity`)

-Simple Stack 0.2.6 (2017-01-20)
---------------------------------
- Added `HistoryBuilder` for convenience

-Simple Stack 0.2.5 (2017-01-17)
---------------------------------
- Minor bug fixes and simplifications

-Simple Stack 0.2.1 (2017-01-17)
---------------------------------
- Added `ReentranceTest` and ported to `simple-stack-demo` codebase
- Fixed some bugs based on `ReentranceTest` - all tests pass now

-Simple Stack 0.2.0 (2017-01-16)
---------------------------------
- State changes are now enqueued while `StateChanger` is not available (after `onPause()`) or a state change is already in progress
- Added `FlowTest` and ported to `simple-stack-demo` codebase

Simple Stack 0.1.1 (2017-01-14)
---------------------------------
- Key and backstack are now provided to custom viewgroup via `getSystemService()`

Simple Stack 0.1.0 (2017-01-13)
---------------------------------
- Added initial `Backstack`, `StateChange` and `StateChanger` classes.
- Backstack allows manipulation of state via `goTo()`, `goBack()` and `setHistory()`.
- Demo persists backstack history through config change and process death.

Limitations:
- ViewState is not persisted
- scheduling state changes (starting a state change while another is in progress) is not allowed
- there is a possibility that state change can occur even after `onPause()`
- key and backstack are manually set to the custom viewgroup, which means these are not directly accessible in their child views (and the interfaces are ugly anyways)
