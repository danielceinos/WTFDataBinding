# WTF is DataBinding

Buenos días o noches, hoy os traigo una breve explicación sobre que es "__Data binding__", porqué creo que deberías usarlo y como cambió mi día a día. Mi nombre es Daniel y actualmente pertenezco al departamento de desarrollo de apps de MásMovil. Recientemente hemos incorporado _data binding_ a nuestras apps de Yoigo y MásMovil, agilizando el proceso de maquetación de las vistas.

--

_Data binding_ es una biblioteca de vinculación de datos entre nuestro código nativo y nuestras vistas declaradas mediante _xml_. Puedes encontrar toda la documentación oficial aquí: [https://developer.android.com/topic/libraries/data-binding/?hl=es-419](https://developer.android.com/topic/libraries/data-binding).

Antes de explicarte porqué deberías añadir _data binding_ a tú proyecto, comencemos con un poco de historia. La forma nativa que nos brinda Android para acceder a nuestras vistas desde nuestro código nativo Java o Kotlin es mediante el tedioso “findViewById”. Haciendo que tengamos que escribir esta sentencia para acceder a nuestras vistas, ya sea para asignar _listener o cambiar un texto.
 
```kotlin
findViewById<TextView>().text = "Text"
```

¿Os suena el caso de tener un ViewHolder de un RecyclerView con 10 lineas repletas de “findViewById”? No estás solo, pero para tú suerte esto tiene una fácil solución.

Mi primera experiencia con el _data binding_ comenzó con el descubrimiento de ButterKnife (más info [aquí](http://jakewharton.github.io/butterknife/)).

_ButterKnife_ fue una de las primeras librerías de _data binding_ que surgió para hacer nuestras vidas un poco más fáciles. Gracias a que podíamos hacer referencia a nuestras vistas de una forma mucho más cómoda desde el código:

```kotlin
@BindView(R.id.title) TextView title;

// ...

title.text = "Title"
```

O implementar de una forma simple un `onClickListener`:

```kotlin
@OnClick(R.id.title)
public void titleClicked(View view) {
  // Do your acction
}

```

Con el uso de _ButterKnife_ conseguíamos un poco menos de _boilerplate_ y un código más limpio, pero aun así teníamos que seguir declarando las vistas como variables en nuestra clase.

A principios del 2017 llegó a mis manos una nueva librería de _data binding_ desarrollada por Google. Google liberó su propia librería de _data binding_ para Android, la cual nos permite escribir expresiones directamente en el _xml_ para realizar ciertas lógicas en la vista como mostrar un determinado texto en un _TextView_, hacer que al pulsar en un botón se ejecute un determinado método o mostrar/ocultar una vista.

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

## ¿Nos merece la pena usarlo?

Actualmente en MásMovil nos encontramos trabajando en una aplicación multimarca, donde nuestro objetivo es la mayor reutilización de código posible cambiando únicamente la parte visual de la aplicación. Pero a la hora de afrontar este cambio de apariencia nos hemos encontrado con que gran parte de la lógica de visualización esta acoplada a nuestras _activities_ y _fragments_. Esto nos lleva a tener que crear una nueva _activity_ o _fragment_ para cada _flavor_ del proyecto cada vez que la lógica de vista es diferente. Con _data binding_ movemos esta lógica a los _xml_, los cuales sí son específicos de cada _flavor_.

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

La cantidad de características de las que nos provee la librería de _data binding_ es extensa, en este articulo voy a tratar las más básicas y las que considero que solamente con ellas ya te compensa el cambio.

El _data binding_'s starter pack se compone de:

- Referencia de vistas para poder usar nuestras vistas en una _activity_, _fragment_ o vista customizada.
- Vinculación de datos en el _xml_.
- Manejo de eventos para poder realizar `onClickListeners` un poco más inteligentes.
- Binding Adapters con los que dotaremos de más lógica a nuestros _xml_.

### Referencia de vistas

Lo necesario para usar _data binding_ en una determinada vista es añadir la etiqueta `<layout>` en xml en el que lo queramos usar. Por cada xml que incluya la etiqueta `<layout>` se generara una clase con el nombre del _xml_ y el sufijo "Binding". Ex: `main_activity.xml` --> `MainActivityBinding`.
Desde esta clase tendremos acceso a todas las vistas declaradas en el _xml_ así como los métodos para vincular datos, si hemos declarado alguno en la vista.

Podemos obtener nuestra clase de binding mediante:

```kotlin
// En vez de usar el setContentView en el onCreate de una activity
val binding: MainActivityBinding = DataBindingUtil.setContentView(this, R.layout.main_activity);

// O podemos inflar una vista mediante
val binding: ItemBeerBinding = ItemBeerBinding.inflate(getLayoutInflater());
val binding: ItemBeerBinding = DataBindingUtil.inflate(layoutInflater, R.layout.detail_fragment, viewGroup, false);

```

Una vez obtenido nuestro objeto Binding ya tendremos accesible todos las vistas declaradas en el _xml_ desde él:

```kotlin
val title: TextView = binding.titleTv
```

### Vinculación de datos

De todas las cosas la que hizo que me enamorase de esta librería fue sin duda el poder pasarle directamente un objeto a un _xml_ , dejando al _xml_ que pinte la información que necesite de ese objeto.
En el _xml_ podemos añadir la etiqueta `<data>` para poder vincular variables, mediante la etiqueta `<variable>`, dentro del _xml_. Esta variable tendrá dos parámetros: 
__name__, que será el que usaremos para acceder a la variable desde el _xml_, y el __type__, que será la clase de la que es nuestra variable.

```xml
<data>
	 <variable name="beer" type="your.package.Beer"/>
</data>
```
y para vincular un objeto a la vista:

```kotlin
binding.beer = User("Black Cat", "British Ale")
```

Una vez vinculado nuestro objeto, podremos acceder a este en una vista del _xml_ usando la sintaxis `@{}`.
Por ejemplo:

```xml
<TextView
	android:text="@{beer.name}"
	.../>
```
            
### Manejo de eventos

En el _xml_ podemos vincular los eventos de las vistas, tales como `onClickListener`, con un objeto sobre el que invocar uno de sus métodos. Esta invocación puede ser mediante la referencia al método de la clase o una lambda.

Podemos pasarle a la vista una implementación de la interfaz:

```kotlin
interface OnClick {
    fun clicked(view: View)
}
```
Y la forma de invocar este método desde el _xml_ será:

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

La librería de DataBinding nos permite la creación de métodos referenciables desde el _xml_ para realizar una determinada lógica.
Por ejemplo podemos crear un binding adapter para cargar directamente una imagen desde una URL con glide:

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

Con los binding adapter también podemos sobrescribir métodos del framework de Android nombrando en el _@BindingAdapter_ el método que queramos sobrescribir.
ex: `@BindingAdapter("android:text")`

En un binding adapter podemos recibir múltiples parámetros. También podemos decidir si son necesarios o no que se declaren todos en el _xml_.

Ex:

```kotlin
@BindingAdapter("text", "color", requireAll = false)
fun setTextWithColor(textView: TextView, text: String?, color: Int?) { ... }
```


## TL;DR y mi conclusión 

Gracias a la vinculación de datos directamente en nuestros _xml_ podemos liberar de bastante lógica a nuestras actividades o fragments, desacoplando la lógica de una activity o fragment de la lógica de vista.

Olvidarnos de usar el `findViewById` o los sintéticos de Kotlin. los cuales nos estaban dando problemas a la hora de trabajar con varios _flavors_.

En nuestro caso nos ha sido bastante útil a la hora de reutilizar vistas entre distintos _flavors_ de nuestra aplicación. Desacoplando la lógica de vista de las _activities_ y _fragments_ llevándola a los _xml_, pudiendo modificar esta lógica sin necesidad de crear una _activity_ o _fragment_ especifico para cada _flavor_.

Y hay mucho más ...
Cosas como el uso de Observables, LiveData, Two-way binding, de los cuales intentaré hablar próximamente de ellos en otros artículos.

___

Puedes ver el código completo del ejemplo que he ido ilustrando en aquí:

https://github.com/danielceinos/BeerDataBindingExample



