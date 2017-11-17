# Kotlin Migration Guide

>  Practical Tips to migrate your Android App to Kotlin

## Steps to **Convert**

Once you learn basics syntax of **Kotlin**

1. Convert files, one by one, via **"⌥⇧⌘K",** make sure tests still pass
2. Go over the Kotlin files and make them more [idiomatic](https://blog.philipphauer.de/idiomatic-kotlin-best-practices/).
3. Repeat step 2 until you convert all the files.
4. Ship it.

---


## Comparing the type system between Java and Kotlin

---

### `Any` v/s `Object`

---

## Type inference

---

## What `nullability` means in Kotlin?

---

## Why `Optionals` exist?

---

## How `data` classes are based on Value Types

---

### `class` v/s `data-class`

---

## Removing `ButterKnife` to use `Kotlin-Extensions`

---

## Common Migration Gotchas

* **annotationProcessor** must be replaced by **kapt** in build.gradle
* Configure tests to mock final classes
* If you are using android **data-binding**, include:

```groovy
kapt com.android.databinding:compiler:3.0.0
```

* `@JvmField` to rescue while using ButterKnife `@InjectView` and Espresso `@Rule`

---
### `annotationProcessor` must be replaced by `kapt` in build.gradle

---

### Configure tests to mock `final` classes

---

### If you are using android `data-binding`, include:

```groovy
kapt com.android.databinding:compiler:3.0.0
```

---

### `@JvmField` to rescue while using ButterKnife `@InjectView` and Espresso `@Rule`

---

## Common Converter Gotchas

* TypeCasting for the sake of Interoperability.
* **Companion** will add extra layer.
* If java method starting with getX(), converter looks for property with the name X.
* **Generics** is hard to get it right on the first go.
* No argument captor.
* **git diff** If two developers are working on same java file and one guy converts it to Kotlin, it will be rework.

---

### `TypeCasting` for the sake of `Interoperability`

Kotlin is not Interoperable right away, but you need to do a lot of work around to make it Interoperable

Here is the **Java** class:

```java
public class DemoFragment extends BaseFragment implements DemoView {
  
    @Override
    public void displayMessageFromApi(String apiMessage) {
      ...
    }
}
```

---

```kotlin
// Kotlin class
class DemoResponse {
    @SerializedName("message") var message: String? = null
}

// Typecasting to String
mainView?.displayMessageFromApi(demoResponse.message as String)
```

---

### `Companion` will add extra layer

Here is **Java** class:

```java
public class DetailActivity extends BaseActivity implements DetailMvpView{
    public static final String EXTRA_POKEMON_NAME = "EXTRA_POKEMON_NAME";

    public static Intent getStartIntent(Context context, String pokemonName) {
        Intent intent = new Intent(context, DetailActivity.class);
        intent.putExtra(EXTRA_POKEMON_NAME, pokemonName);
        return intent;
    }
}
```
---

Converted  **Kotlin** class:

```kotlin
class DetailActivity : BaseActivity(), DetailMvpView {
    companion object {
        val EXTRA_POKEMON_NAME = "EXTRA_POKEMON_NAME"

        fun getStartIntent(context: Context, pokemonName: String): Intent {
            val intent = Intent(context, DetailActivity::class.java)
            intent.putExtra(EXTRA_POKEMON_NAME, pokemonName)
            return intent
        }
    }
}
```
---

```java
public class MainActivity extends BaseActivity implements MainMvpView {
  private void pokemonClicked(Pokemon pokemon) {
      startActivity(DetailActivity.Companion.getStartIntent(this, pokemon))    
  }
}
```

---

### Method names starting with `get`

Here is the Java class:

```java
public interface DemoService {
    @GET("posts")
    Observable<PostResponse> getDemoResponse();

    @GET("categories")
    Observable<CategoryResponse> getDemoResponse2();
}
```

---

Converted  **Kotlin** class:

```kotlin
interface DemoService {
    @get:GET("posts")
    val demoResponse: Observable<PostResponse>
    
    @get:GET("categories")
    val demoResponse2: Observable<CategotyResponse>
}
```

Expecting methods **demoResponse** and **demoResponse2**, They are being interpreted as getter methods, this will cause lots of issues.

---

## Eliminating `!!` from your Kotlin code

* Use **val** instead of **var**
* Use **lateinit**
* Use **let** function
* User **Elivis** operator

---

### Use `val` instead of `var`

* Kotlin makes you think about immutability on the language level and that’s great.
* **var** and **val** mean **"writable"** and **"read-only"**
* If you use them as immutables, you don’t have to care about nullability.

---

### No `ArgumentCaptor`

If you are using Mockito’s ArgumentCaptor, you will most probably get following error

```
java.lang.IllegalStateException: classCaptor.capture() must not be null
```

The return value of **classCaptor.capture()** is null, but the signature of **SomeClass#someMethod(Class, Boolean)** does not allow a *null* argument.

---


### Use `lateinit`

```kotlin
private var adapter: RecyclerAdapter<Droids>? = null

override fun onCreate(savedInstanceState: Bundle?) {
   super.onCreate(savedInstanceState)
   mAdapter = RecyclerAdapter(R.layout.item_droid)
}

fun updateTransactions() {
   adapter!!.notifyDataSetChanged()
}

```

---

```kotlin
private lateinit var adapter: RecyclerAdapter<Droids>

override fun onCreate(savedInstanceState: Bundle?) {
   super.onCreate(savedInstanceState)
   mAdapter = RecyclerAdapter(R.layout.item_droid)
}

fun updateTransactions() {
   adapter?.notifyDataSetChanged()
}

```

---

### Use `let` function

```kotlin
private var photoUrl: String? = null

fun uploadClicked() {
    if (photoUrl != null) {
        uploadPhoto(photoUrl!!)
    }
}
```

---

```kotlin
private var photoUrl: String? = null

fun uploadClicked() {
    photoUrl?.let { uploadPhoto(it) }
}

```

---

### Use `elvis` operator

Elvis operator is great when you have a fallback value for the null case. So you can replace this:

```kotlin
fun getUserName(): String {
   if (mUserName != null) {
       return mUserName!!
   } else {
       return "Anonymous"
   }
}
```

----

**elvis** operator is great when you have a fallback value for the null case. So you can replace this:

```kotlin
fun getUserName(): String {
   return mUserName ?: "Anonymous"
}
```

---

## Annotations to make Kotlin code interoperable with Java.

---

## Migrating Unit Tests to Use `Mockito-Kotlin`

---

## Power of Kotlin-Extensions

---

## Idiomatic Kotlin

---

## Acknowledgement 

[**Arun Sasidharan**](https://github.com/esoxjem) for Initial Idea



---

## References

---