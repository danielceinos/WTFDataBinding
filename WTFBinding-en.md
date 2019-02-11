# WTF is DataBinding

Good morning or nights, today I bring a brief explanation of what is "__Data binding__", why I think you should use it and how it changed my day to day. My name is Daniel and I currently belong to the development Department of Apps MásMovil. We have recently incorporated _data binding_ into our Apps Yoigo and MásMovil, streamlining the process of layout of the views.

--

_Data Binding_ is a data link library between our native code and our views declared by _xml_. You can find all the official documentation here: [https://developer.android.com/topic/libraries/data-binding/?hl=en-419] (https://developer.android.com/topic/libraries/data-binding).

Before explaining why you should add _data binding_ to your project, let's start with a little bit of history. The native form that gives us Android to access our views from our native code Java or Kotlin is through the tedious "findViewById". Having to write this statement to access our views, either to assign listener or change a text.
 
```kotlin
findViewById <TextView> ().text = "Text"
```

Do you sound the case of having a ViewHolder of a RecyclerView with 10 lines full of "findViewById"? You're not alone, but for your luck this has an easy solution.

My first experience with the _ data binding began with the discovery of ButterKnife (more info [here] (http://jakewharton.github.io/butterknife/)).

_ButterKnife_ was one of the first _data binding_ libraries that emerged to make our lives a little easier. Because we could refer to our views in a much more comfortable way from the code:

``` Kotlin
@BindView (R.id.title) TextView title;

// ...

title.text = "Title"
```

Or to implement in a simple way a ' onClickListener ':

```kotlin
@OnClick (R.id.title)
public void titleClicked (View view) {
  // Do your Acction
}
```

With the use of _ButterKnife_ we got a little less than _boilerplate_ and a cleaner code, but still we had to continue declaring the views as variables in our class.

At the beginning of 2017 came to my hands a new library of _data binding_ developed by Google. Google freed its own library of _data binding_ for Android, which allows us to write expressions directly in the _XML_ to make certain logics in the view as to show a certain text in a _TextView_, to make that when pressing a button it executes a A certain method or show/hide a view.

And in this way we can simplify things up to this level:

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

```

! [] (blow_mind. gif)

## Are we worth using it?

Currently in MásMovil we are working on a multibrand application, where our goal is the greatest reuse of code possible by changing only the visual part of the application. But when faced with this change of appearance we have found that much of the logic of visualization is coupled to our activities and fragments. This leads us to have to create a new activity or a fragment for each flavor of the project every time the view logic is different. With _data binding_ We move this logic to _XML_, which yes are specific to each flavor.

## and how do I add it to my project?

To enable it is as simple as adding DataBinding in the Build. Grade:

```
android {
    ...
    dataBinding {
        enabled = true
    }
}
```

## Okay, but how do I use it?

The number of features provided by the Library of _ data binding _ is extensive, in this article I will try the most basic and I think that only with them already compensates for the change.

The _data binding_'s starter pack consists of:

- View reference in order to use our views in an activity, fragment or custom view.
- Link Data in _XML_.
- Event management to be able to make `onClickListeners` a little more intelligent.
- Binding adapters with which we will provide more logic to our _XML_.

### View Reference

The necessary to use _ data binding in a given view is to add the label ' <layout> ' in XML in which we want to use it. For each XML that includes the ' <layout> ' tag, a class is generated with the name of the _ xml _ and the suffix "Binding". Ex: ' main_activity. xml '--> ' MainActivityBinding '.
From this class we will have access to all the views declared in the _ XML as well as the methods to link data, if we have declared one in the view.

We can get our binding class by:

```kotlin
//Instead of using the SetContentView in the onCreate of an activity
val binding: MainActivityBinding = DataBindingUtil.setContentView (this, R.layout.main_activity);

Or we can inflate a view using
val binding: ItemBeerBinding = ItemBeerBinding.inflate (getLayoutInflater());
val binding: ItemBeerBinding = DataBindingUtil.inflate (layoutInflater, R.layout.detail_fragment, viewGroup, false);
```

Once we have obtained our Binding object we will have access All declared views In the _XML_ from it:

```kotlin
val title: TextView = binding.titleTv
```

### Data Link

Of all the things that made me fall in love with this library was without a doubt the power to pass an object directly to an _XML_ , leaving the _XML_ to paint the information you need from that object.
In the _XML_ we can add the label `<data>` to be able to link variables, by means of the label `<variable>`, within the _XML_. This variable will have two parameters: 
__name__, which will be used to access the variable from the _XML_, and the __type__, which will be the class of which is our variable.

```xml
<data>
	 < Variable name = "Beer" type = "your. Package. Beer"/>
</data>
```
and to link an object to the view:

```kotlin
binding.beer = User("Black Cat", "British Ale")
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

...

<ImageView
	android:onClick="@{(view)-> onBeerLogoClick.clicked(view)}"
		.../>
		
		Or:
		
<ImageView
	android:onClick="@{onBeerLogoClick::clicked}"
		.../>

```

### Binding Adapters

The DataBinding library allows us to create referential methods from the _XML_ to make a certain logic.
By Example We can create a binding adapter to directly load an image from a glide URL:

```kotlin
@BindingAdapter("imageUrl")
fun setUrlImage(imageView: ImageView, url: String?) {
    GlideApp.with (imageView.context)
        .load (url)
        .into(imageView)
}
```
Where `IMAGEURL` will be the name of the method that we invoke from the XML:

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


## TL; DR And my conclusion 

Thanks to the link of data directly in our _XML_ we can release from enough logic to our activities or fragments, uncoupling the logic of an activity or fragment of the logic of view.

Forget about using the `findViewById` or Kotlin synthetics. Which were giving us problems when working with several flavors.

In our case it has been quite useful when it comes to reuse views between different flavors of our application. Undocking The view logic of activities and fragments taking it to the _XML_, being able to modify this logic without having to create an activity or a fragment specific for each flavor.

And there is much more...
Things like the use of observable, LiveData, Two-way binding, of which I will try to speak soon of them in other articles.

___

You can see the full code of the example I've been illustrating here:

Https://github.com/danielceinos/BeerDataBindingExample



