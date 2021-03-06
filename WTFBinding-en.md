# WTF is DataBinding

Good morning (or evening)! Today I bring a brief explanation of what __"Data Binding"__ is, why I think you should use it and how it changed my daily life. My name is Daniel and I currently belong to the Apps development team at MásMovil Group. We have recently incorporated _data binding_ into our Yoigo and MásMovil apps, streamlining the process of laying out the views.

--

_Data Binding_ is a data link library between our native code and our views declared in _xml_. You can find all the official documentation here: [https://developer.android.com/topic/libraries/data-binding/?hl=en-419] (https://developer.android.com/topic/libraries/data-binding).

Before explaining why you should add data binding to your project, let's start with a little bit of history. The native way that Android gives us access to our views from our native Java or Kotlin code is through the tedious `findViewById`, having to write this statement to access our views, either to assign listener or change a text.
 
```kotlin
findViewById<TextView>().text = "Text"
```

Does the case of having a ViewHolder of a RecyclerView with 10 lines full of `findViewById` sound familiar to you? You're not alone! Luckily this has an easy solution.

My first experience with  _data binding_ began when I discovered __ButterKnife__ (more info [here](http://jakewharton.github.io/butterknife/)).

_ButterKnife_ was one of the first _data binding_ libraries that emerged to make our lives a bit easier. Because we could refer to our views in a much more comfortable way from the code:

```kotlin
@BindView (R.id.title) TextView title;
// ...
title.text = "Title"
```

Or to implement in a simple way an `onClickListener`:

```kotlin
@OnClick (R.id.title)
public void titleClicked (View view) {
	// Any logic
}
```

With the use of _ButterKnife_ we got a little less than boilerplate and a cleaner code, but still we had to declare the views as variables in our class.

At the beginning of 2017 came to my hands a new library of data binding developed by Google. Google released its own data binding library for Android, which allows us to write expressions directly in the _XML_ to make certain view-related logic like show a certain text in a _TextView_, making that by just pressing a button it executes a certain method, or even show/hide a view.

This way we can simplify things up to this level:

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout>
    <data>
        <variable name="beer" type="your.package.Beer"/>
    </data>
    <android.support.constraint.ConstraintLayout
            ...>

        <TextView
                android:text="@{beer.name}"
                .../>

        <TextView
                android:text='@{beer.style ?? "Unknown style" }'
                .../>

        <TextView
                android:text='@{beer.brewer ?? "Unknown brewer"}'
                .../>

    </android.support.constraint.ConstraintLayout>
</layout>
```

![](blow_mind. gif)

## Is it worth using Data Binding?

Currently at MásMovil we are working on a multibrand application and our goal is to reuse code as much as possible by changing only the visual part of the application. But when we have faced with this change of appearance the results are that much of the visualization logic is coupled to our activities and fragments. This leads us to have to create a new _activity_ or a _fragment_ for each flavor of the project every time the view logic is different. With _data binding_ We move this logic to the _XML_ file, which yes are indeed to each flavor.

## How can I add it to my project?

Enable it is as simple as adding this into your app Build.Grade:

```groovy
android {
    dataBinding {
        enabled = true
    }
}
```

## Okay, but how do I use it?

The number of features provided by the _data binding_ library is extensive. In this article I will show the most basic concepts and I think that by just adding them makes the change worthy.

The _data binding_'s starter pack consists of:

- View reference in order to use our views in an _activity_, _fragment_ or custom view.
- Link Data in _XML_.
- Event management to be able to make `onClickListeners` a little more intelligent.
- Binding adapters with which we will provide more logic to our _XML_.

### View Reference

The necessary to use _ data binding in a given view is to add the label `<layout>` in XML in which we want to use it. For each _XML_ that includes the `<layout>` tag, a class is generated with the name of the _XML_ and the suffix "Binding". Ex: ` main_activity.xml` --> `MainActivityBinding`.
From this class we will have access to all the views declared in the _XML_ as well as the methods to link data, if we have declared one in the view.

We can get our binding class by:

```kotlin
// Instead of using the setContentView in the onCreate of an activity
val binding: MainActivityBinding = DataBindingUtil.setContentView (this, R.layout.main_activity);

// Or we can inflate a view using
val binding: ItemBeerBinding = ItemBeerBinding.inflate (getLayoutInflater());
val binding: ItemBeerBinding = DataBindingUtil.inflate (layoutInflater, R.layout.detail_fragment, viewGroup, false);
```

Once we have obtained our Binding object we will have access to all declared views in the _XML_ from it:

```kotlin
val title: TextView = binding.titleTv
```

### Data Link

Of all the things that made me fall in love with this library was without a doubt the power to pass an object directly to an _XML_ , leaving the _XML_ to paint the information you need from that object.
In the _XML_ we can add the label `<data>` to be able to link variables, by means of the label `<variable>`, within the _XML_. This variable will have two parameters: 
__name__, which will be used to access the variable from the _XML_, and the __type__, which will be the class of which is our variable.

```xml
<data>
	<variable name="beer" type="your.package.beer"/>
</data>
```
and to link an object to the view:

```kotlin
binding.beer = Beer("Black Cat", "British Ale")
```

Once we have linked our object, we will be able to access this in a view of the _XML_ using the syntax `@{}`.
For example:

```xml
<TextView
	android:text = "@{beer.name}"
	.../>
```
            
### Event Handling

In the _XML_ we can link the events of the views, such as `OnClickListener`, with an object on which to invoke one of its methods. This invocation can be by reference to the class method or a lambda.

We can pass it to the view an implementation of the interface:

```kotlin
interface OnClick {
    	fun clicked (view: View)
}
```
And how to invoke this method from the _XML_ will be:

```kotlin

<data>
	<variable name="onBeerLogoClick" type="***.OnClick"/>
</data>

//...

<ImageView
	android:onClick="@{(view)-> onBeerLogoClick.clicked(view)}"
	.../>
		
// Or:
		
<ImageView
	android:onClick="@{onBeerLogoClick::clicked}"
 	.../>

```

### Binding Adapters

The _data binding_ library allows us to create referential methods from the _XML_ to make a certain logic.
For example, we can create a _binding adapter_ to directly load an image from a glide URL:

```kotlin
	@BindingAdapter("imageUrl")
	fun setUrlImage(imageView: ImageView, url: String?) {
    	GlideApp.with (imageView.context)
        	.load (url)
        	.into(imageView)
	}
```
Where `imageUrl ` will be the name of the method that we invoke from the XML:

```kotlin
 <ImageView
    app:imageUrl='@{beer.imageUrl"}'
    .../>
```

With the binding adapter we can also override methods of the Android framework naming in the _@BindingAdapter_ the method that we want to overwrite.
Ex: `@BindingAdapter("android:text")`

In a binding adapter we can receive multiple parameters. We can also decide whether or not they are required to be declared in the _XML_.

Former:

```kotlin
@BindingAdapter("text", "color", requireAll = False)
fun setTextWithColor(textView: TextView, text: String?, color: Int?) { ... }
```


## TL;DR And my conclusion 

Thanks to link data directly to our _XML_ we can lightweight our application by decoupling the logic of an activity or fragment from the view logic.

We should leave behind the `findViewById` or Kotlin synthetics ([they aren't a good practice anymore](https://android-review.googlesource.com/c/platform/frameworks/support/+/882241) and they are gonna be deprecated). Which were giving us problems when working with several flavors.

In our case it has been quite useful when it comes to reuse views between different flavors of our application. Decoupling the view logic of _activities_ and _fragments_ by moving it to the _XML_, allow us to modify this logic without having to create an activity or a fragment specific for each flavor.

And there is much more… Things like the use of observable, LiveData, Two-way binding, etc. But that's another story, and sooner or later I will write about them!

I hope you have found this article useful, and don't hesitate to contact me for any question or advice!

___

# Show me the Code :)

You can see the full code of the example I've been illustrating here:

https://github.com/danielceinos/BeerDataBindingExample



