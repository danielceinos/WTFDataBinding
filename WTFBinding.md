Databinding es una biblioteca de vinculación de datos entre nuestro codigo nativo y nuestras vistas declaradas en XML.

Un poco de historia, la forma nativa de acceder a nuestras vistas desde nuestro codigo es mediante el tedioso “findViewById”, provocandonos escribir hasta la saciedad esta sentencia para acceder a nuestras vistras para setear un click listener o cambiar un texto.

```kotlin
findViewById<TextView>().text = "Text"
```

esto se hace mas presente en vistas generadas por codigo como puede ser una celda de un recyclerview

```kotlin
holder.itemView.findViewById<TextView>(R.id.name_tv).text = "Name"
holder.containerView.setOnClickListener { clickListener(item) }
```

ButterKnife fue una de las primeras librerias que surgio para solvertar este problema. Con butterKnife podiamos hacer referencia a vistas des una forma mucho más cómoda desde el codigo

```kotlin
@BindView(R.id.title) TextView title;

// ...

title.text = "Title"

// ...

@OnClick(R.id.title)
public void titleClicked(View view) {
  // Do your acction
}

```

Mediante esta lireria conseguiamos un poco menos de boilerplate y un codigo más limpio.
Hasta que Google en el 2015 liberó su propia libreria de databinding para android.

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout>
    <data>
        <variable name="beer" type="***.Beer"/>
    </data>
    <android.support.constraint.ConstraintLayout
            ...>

        <TextView
                android:text="@{beer.name}"
                .../>

        <TextView
                android:text='@{beer.style ?? "Unkown style" }'
                .../>

        <TextView
                android:text='@{beer.brewer ?? "Unkown brewer"}'
                .../>

    </android.support.constraint.ConstraintLayout>

```
INSERTE GIF DE BLOW MIND

Como funciona?

Para habilitarlo tan solo tendremos que añadir databinding en el build.gradle:

```
android {
    ...
    dataBinding {
        enabled = true
    }
}
```

Por cada xml que incluya la etiqueta ```<layout></layout>``` se generara una clase con el nombre del xml y el sufijo "Binding". Ex: `main_activity.xml` --> `MainActivityBinding`.
Desde esta clase tendremos acceso a todas las variables declaradas en el xml asi como los metodos para vincular datos si hemos declarado algunos en la vista.

Para obtener esta clase tenemos varias opciones en funcion de que layout estemos intentando vincular.

```kotlin
// En vez de usar el setContentView en el onCreate de una activity
val binding: MainActivityBinding = DataBindingUtil.setContentView(this, R.layout.main_activity);

// O podemos inflar una vista mediante
val binding: ItemBeerBinding = ItemBeerBinding.inflate(getLayoutInflater());
val binding: ItemBeerBinding = DataBindingUtil.inflate(layoutInflater, R.layout.detail_fragment, viewGroup, false);

```

Desde aqui ya tendremos accesible todos las vistas declaradas en el xml.

```kotlin
val title: TextView = binding.titleTv
```

para vincular un objeto a la vista:

```kotlin
binding.beer = User("Black Cat", "British Ale")
```

Manejo de eventos:

En el xml podemos vincular los eventos de las vistas, tales como `onClickListener`,  con un objeto sobre el que invocar uno de sus metodos, esta invocación puede ser mediante la referencia al metodo o una lambda

Podemos pasarle a la vista una implementacion de la interfaz:

```kotlin
interface OnClick {
    fun clicked(view: View)
}
```
Y la forma de invocarla desde el xml será:

```kotlin

<data>
	<variable name="onBeerLogoClick" type="***.OnClick"/>
</data>

...

<ImageView
	android:onClick="@{(view) -> onBeerLogoClick.clicked(view)}"
		.../>
		
		O BIEN:
		
<ImageView
	android:onClick="@{(onBeerLogoClick::clicked}"
		.../>

```

Binding adapters

La libreria de DataBinding nos permite la creacion de metodos referenciables desde el xml para realizar una determinada logica.
Por ejemplo podemos crear un binding adapter para cargar directamente una imagen desde una url con glide:

```kotln
@BindingAdapter("imageUrl")
fun setUrlImage(imageView: ImageView, url: String?) {
    GlideApp.with(imageView.context)
        .load(url)
        .into(imageView)
}
```
Donde `imageUrl` sera el nombre del atributo que invocaremos desde el xml:

```kotlin
 <ImageView
    app:imageUrl='@{beer.imageUrl"}'
    .../>
```
Los bingind adaprter tambien podemos sobrescribir metodos del framework de android nombrando el bindingAdapter con el metodo que queramos sobrescribir.

Los bingind adapters tambien puedes recibir multiples parametros

```kotlin
@BindingAdapter("text", "color", requireAll = false)
fun setTextWithColor(textView: TextView, text: String, color: Int) { ... }
```

