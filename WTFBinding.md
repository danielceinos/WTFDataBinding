# WTF is DataBinding

Buenos días o noches, hoy os traigo un breve resumen de que es databinding ,porqué creo que deberias usarlo y como cambió mi día a día. Actualmente pertenezco al departamento de desarrollo de apps de MásMovil y recientemente hemos incorporado databinding a nuestras apps de Yoigo y Másmovil, agilizando el proceso de maquetación de las vistas.

Databinding es una biblioteca de vinculación de datos entre nuestro código nativo y nuestras vistas declaradas en XML. Puedes encontrar toda la documentación oficial en [https://developer.android.com/topic/libraries/data-binding/?hl=es-419](https://developer.android.com/topic/libraries/data-binding/?hl=es-419).

Comencemos con un poco de historia. La forma nativa para acceder a nuestras vistas desde nuestro código es mediante el tedioso “findViewById”, haciendo que tengamos que escribir esta sentencia para acceder a nuestras vistas, ya sea para asignar listener o cambiar un texto.

```kotlin
findViewById<TextView>().text = "Text"
```

¿Os suena el caso de tener un ViewHolder de un RecyclerView con 10 lineas repletas de “findViewById”? No estás solo, pero para tú suerte esto tiene solución.

Mi primera expericia con el __data binding__ comenzó con el descubrimiento de ButterKnife (más info [aqui](http://jakewharton.github.io/butterknife/)).

ButterKnife fue una de las primeras librerías de "__data binding__" que surgió para hacer nuestras vidas un poco más faciles. Gracias a cosas como que podíamos hacer referencia a vistas des una forma mucho más cómoda desde el código y otra clase de anotaciones:

```kotlin
@BindView(R.id.title) TextView title;

// ...

title.text = "Title"
```

O implementar click listener:

```kotlin
@OnClick(R.id.title)
public void titleClicked(View view) {
  // Do your acction
}

```

Con el uso de ButterKnife conseguíamos un poco menos de boilerplate y un código más limpio, pero aun así tenemos que seguir declarando las vistas como variables en nuestra clase.

A principios del 2017 llegó a una nueva librería de __data binding__ desarrollada por Google.
En el 2015 Google liberó su propia librería de __data binding__ para Android, la cual nos permite escribir expresiones directamente en el xml para realizar ciertas lógicas en la vista como mostrar un determinado texto en un TextView, hacer que al pulsar en un botón se ejecute un determinado método o mostrar/ocultar una vista.

Y de esta forma podemos simplificar las cosas hasta este nivel:

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

![](blow_mind.gif)

## ¿Y cómo lo añado a mi proyecto?

Para habilitarlo es tan simple como añadir DataBinding en el build.gradle:

```
android {
    ...
    dataBinding {
        enabled = true
    }
}
```

## Vale, pero ¿Y cómo lo uso?

La cantidad de características de las que nos provee la libreria de databinding en este articulo voy a tratar las más básicas y que considero que solamente con ellas ya te compensa empezar a usarla.

Este pack básico se compone de:

- Referencia de vistas para poder usar nuestras vistas en una actividad, fragment o vista customizada.
- Vinculación de datos en el XML.
- Manejo de eventos para poder realizar unos clicks listener un poco más inteligentes.
- Binding Adapters con los que dotaremos de más logica a nuestros xml.

### Referencia de vistas

Lo necesario para usar _data binding_ en una determinada vista es añadir la etiqueta `<layout>` en xml en el que lo queramos usar. Por cada xml que incluya la etiqueta `<layout>` se generara una clase con el nombre del xml y el sufijo "Binding". Ex: `main_activity.xml` --> `MainActivityBinding`.
Desde esta clase tendremos acceso a todas las vistas declaradas en el xml así como los métodos para vincular datos, si hemos declarado alguno en la vista.

Podemos obtener nuestra clase de binding mediante:

```kotlin
// En vez de usar el setContentView en el onCreate de una activity
val binding: MainActivityBinding = DataBindingUtil.setContentView(this, R.layout.main_activity);

// O podemos inflar una vista mediante
val binding: ItemBeerBinding = ItemBeerBinding.inflate(getLayoutInflater());
val binding: ItemBeerBinding = DataBindingUtil.inflate(layoutInflater, R.layout.detail_fragment, viewGroup, false);

```

Una vez obtenido nuestro objeto Binding ya tendremos accesible todos las vistas declaradas en el xml desde él:

```kotlin
val title: TextView = binding.titleTv
```

### Vinculación de datos

De todas las cosas la que hizo que me enamorase de esta libreria fue sin duda el poder pasarle directamente un objeto a un _xml_ , dejando al xml que pinte la información que necesite de ese objeto.
En el xml podemos añadir la etiqueta `<data>` para poder vincular variables, mediante la etiqueta `<variable>`, dentro del xml. Esta variable tendrá dos parámetros: 
__name__, que será el que usaremos para acceder a la variable desde el xml, y el __type__, que será la clase de la que es nuestra variable.

```xml
<data>
	 <variable name="beer" type="your.package.Beer"/>
</data>
```
y para vincular un objeto a la vista:

```kotlin
binding.beer = User("Black Cat", "British Ale")
```

Una vez vinculado nuestro objeto, podremos acceder a este desde las vistas usando la sintaxis `@{}`.
Por ejemplo:

```xml
<TextView
	android:text="@{beer.name}"
	.../>
```
            
### Manejo de eventos

En el xml podemos vincular los eventos de las vistas, tales como `onClickListener`,  con un objeto sobre el que invocar uno de sus métodos. Esta invocación puede ser mediante la referencia al método de la clase o una lambda.

Podemos pasarle a la vista una implementación de la interfaz:

```kotlin
interface OnClick {
    fun clicked(view: View)
}
```
Y la forma de invocar este método desde el xml será:

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

### Binding adapters

La librería de DataBinding nos permite la creación de métodos referenciables desde el xml para realizar una determinada logica.
Por ejemplo podemos crear un binding adapter para cargar directamente una imagen desde una url con glide:

```kotlin
@BindingAdapter("imageUrl")
fun setUrlImage(imageView: ImageView, url: String?) {
    GlideApp.with(imageView.context)
        .load(url)
        .into(imageView)
}
```
Donde `imageUrl` será el nombre del método que invocaremos desde el xml:

```kotlin
 <ImageView
    app:imageUrl='@{beer.imageUrl"}'
    .../>
```

Con los binding adapter también podemos sobrescribir métodos del framework de android nombrando en el @BindingAdapter el método que queramos sobrescribir.
ex: `@BindingAdapter("android:text")`

En un binding adapter podemos recibir múltiples parámetros. Tambien podemos decidir si son necesarios o no que se declaren todos en el xml.

Ex:

```kotlin
@BindingAdapter("text", "color", requireAll = false)
fun setTextWithColor(textView: TextView, text: String?, color: Int?) { ... }
```


## TL;DR y conclusión 

Gracias a la vinculación de datos directamente en nuestros _xml_ podemos liberar de bastante lógica a nuestras actividades o fragments, desacomplando la lógica de una activity o fragment de la lógica de vista.

Liberarnos del `findViewById` o de los sintéticos de Kotlin, los cuales nos estaban dando problemas a la hora de trabajar con varios _dimension_.

En nuestro caso nos ha sido bastante útil a la hora de reutilizar vistas entre distintos flavors de nuestra applicación, pudiendo modificar la lógica de una vista sin necesidad de crear una actividad especifica para cada flavor.

Y hay mucho más ...
Cosas como el uso de Observables, LiveData, Two-way binding, de los cuales intentaré hablar próximamente de ellos.

___

Puedes ver el código completo del ejemplo que he ido ilustrando en aqui:

https://github.com/danielceinos/BeerDataBindingExample



