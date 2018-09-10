#Tether
`Version` - 1.0.0  

`Kotlin` - 1.2.61  
`Gradle Tools` - 3.2.0-rc03  
`Min Version` - 21  
`Compile Version` - 28

## Known Issues
* The RxBinding libraries below must be explicitly defined by your application

## Dependencies
#### Transitive
* [RxBinding 2.1.1](https://github.com/JakeWharton/RxBinding)
* [RxBinding AppCompat 2.1.1](https://github.com/JakeWharton/RxBinding)
 
#### Implementation
This library is currently using:

* [Architecture Components 2.0.0-rc01](https://developer.android.com/topic/libraries/architecture/index.html)
* lifecycle-runtime  
* lifecycle-common-java8
* lifecycle-extensions

## Philosophy

Tether is built around the concept of databinding which is the process of establishing connection between an application's UI and Business layers. There are many libraries to facilitate this on Android including the currently accepted way which is Google's Databinding library.  

However, there are flaws with this pure approach if developers want to implement MVVM with more sophisticated functional features.  The workflow this library facilitates is the combination of RxJava/RxKotlin with Architecture Components. Architecture components is used to handle the different problems that arise with the Android framework and RxJava/RxKotlin is used to maniputate the data that is passed around by applications via event streams.  

Tether is the glue between those worlds that provides an API to take an Rx Observable and connect it to an UI Android class. The benefit of using Tether is its ability to manage the Disposables that are generated by these bindings. With Tether, you never again have to manage your own dispose collection and hope you don't forget to dispose them.  

Finally Tether provides Kotlin extensions off of the Android framework that allows users to monitor events in a more concise and idiomatic way.

## Usage

You will access a `LiveData` object from your `ViewModel` and call `bind`, passing in the `LifecycleOwner` and a `Driver` that has data flowing through it.

```
viewModel.mutableLiveData.bind(this, driver)
```

The below example has a `LiveData` object that stores changes to the first name.  This is bound with text changes from a user entering text into the first name field in the UI.

```
// With Driver Extension
viewModel.firstName.bind(this, 
			binding.firstNameEditText.textChanges)
```
```
// Without Driver Extension
viewModel.firstName.bind(
			this,
			      binding.firstNameEditText.textChanges()
                        .skipInitialValue()
                        .map { it.toString() }
                        .toDriver()
        )
```

The next example shows how to use the `Action` class from your `View` by accessing it from your `ViewModel`.  The below example is using the `observe` method instead of manually observing on each `LiveData` on the `Action` class.  You can call observe as a setup process and make the `start` call at a later point after user interaction.

```
viewModel.apiAction.observe(
        owner = this,
        executing = { executing -> },
        output = { output -> },
        error = { error -> }
        )
        .input(Any())
        .start()
```

## Definitions

#### LiveDataExtensions
* `bind` - This extension of `MutableLiveData` is the main function you will use. For linking an `Observable` to lifecycle callbacks with emission through `LiveData`

see: `BindableImpl` and `SourceImpl` in the `binder/impl` package for how this works

#### Action
* `constructor` - pass a block of code that returns an `Observable`
* `observe` - pass all of your lambdas that will be called at the appropriate time.
	* The other option is to setup observering directly on the `LiveData` properties. (`completed`, `error`, `executing`, and `Action` itself)
* `input` - data that is passed to your lambda from the constructor
* `start` - subscribes to the underlying `Observable` to begin fetching information


#### Optional
* A lightweight class to work around the limitation of RxJava2 not being able to pass null values through the chain.
* Used in `Action`


Classes
--------
#### Bindable
Interface for `BindableImpl` for updating `LiveData` values

#### Source
Interface for returning an `Observable`

#### BindableImpl
Creates a `Binder` and updates `MutableLiveData` with `MutableLiveData.postValue`

#### SourceImpl
Stores an `Observable` that is used in the `Binder`

#### Binder
Binds to a `LifecycleOwner` by subscribing to `Source.observable` and updating `Bindable.bindValue`.

Handles disposing the subscription from `Lifecycle.Event.ON_PAUSE` or `Lifecycle.Event.ON_DESTROY` while also resuming the subscription if necessary from `Lifecycle.Event.ON_START` or `Lifecycle.Event.ON_RESUME`.

This is a one directional bind between the `Source.observable` and `Bindable.bindValue`

#### Optional
Simple Optional class for Kotlin

#### Driver
- Represents a contract for binding that guarantees delivery on the main thread as well as never failing
- The typical usage of this class will be calling the `toDriver` extension on an `Observable`

#### Action
Typical usage flow for this class:  
- Setup callbacks through `observe` or directly with LiveData observe method off `observe`(`Action, error, etc`)  
- Attach the input value with `input`  
- Begin the action with `start`  

 This class encapsulates the subscribing and monitoring of an underlying
 observable. It exposes convenience LiveData objects that allow the user to monitor
 meaningful events throughout the execution of said observable. The goal of this is to make
 the language stronger around what is happening throughout the lifetime of the observable.
 The Action itself is also a LiveData object and is the hook into observing the data emitted from the
 underlying observable.

 If you need a short lived operation, we have to null out the values in the internal live data so that
 observing again will no emit a value.  This happens when sharing view models across fragments.

 To use this with RxJava we had to create a class that contains a nullable type.
 This is because RxJava assumes everything to be non-null but LiveData can hold null values. I hate that we're going
 to have to check against this but until LiveData and RxJava play nice together we have to.

## Library developers

### Environment

This library was developed using Android Studio 3.2 and the embedded Java JDK.

### Local installation

Installing a build of the library locally allows you to test the library in the context of an app you're building in parallel.
This does not replace the need for a sample app module in the library code repo. It just allows for deeper testing and validation in a real development context.

To install a build of the library, issue the following command in terminal from the root project's directory:

    ./gradlew install

This will produce the appropriate binaries and copy it into your local maven repository (~/.m2/repository).

In your project, include your dependency as you normally would. For example:

    dependencies {
        classpath 'com.fjordnet.tether:tether:1.0.0'
    }

You will need to add the local maven repository for Gradle to be able to find it.

    repositories {
        mavenLocal()
    }

## License

    Copyright 2017-2018 Fjord

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
