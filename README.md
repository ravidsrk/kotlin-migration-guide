# Kotlin Migration Guide

>  Practical Tips to migrate your Android App to Kotlin

---

## Contents

- [Configure Kotlin](#configure-kotlin)
- [Steps to **Convert**](#steps-to-convert)
- [Comparing the type system between `Java` and `Kotlin`](#comparing-the-type-system-between-java-and-kotlin)
  - [`Any` v/s `Object`](#any-vs-object)
- [Type inference](#type-inference)
- [What `nullability` means in Kotlin?](#what-nullability-means-in-kotlin)
- [Why `Optionals` exist?](#why-optionals-exist)
- [How `data` classes are based on Value Types](#how-data-classes-are-based-on-value-types)
  - [`class` v/s `data-class`](#class-vs-data-class)
- [Using `Parcelize` annotation for `Parcelable`](#using-parcelize-annotation-for-parcelable)
- [String templates](#string-templates)
- [Removing `ButterKnife` and `findViewById` to use `Kotlin-Android-Extensions`](#removing-butterknife-and-findviewbyid-to-use-kotlin-android-extensions)
- [`apply`, `let`, `with` and `also`](#apply-let-with-and-also)
- [Common Migration Gotchas](#common-migration-gotchas)
  - [`annotationProcessor` must be replaced by `kapt` in build.gradle](#annotationprocessor-must-be-replaced-by-kapt-in-buildgradle)
  - [Configure tests to mock `final` classes](#configure-tests-to-mock-final-classes)
  - [If you are using android `data-binding`, include:](#if-you-are-using-android-data-binding-include)
  - [`@JvmField` to rescue while using ButterKnife `@InjectView` and Espresso `@Rule`](#jvmfield-to-rescue-while-using-butterknife-injectview-and-espresso-rule)
- [Common Converter Gotchas](##common-converter-gotchas)
  - [`TypeCasting` for the sake of `Interoperability`](#typecasting-for-the-sake-of-interoperability)
  - [`Companion` will add extra layer](#companion-will-add-extra-layer)
  - [Method names starting with `get`](#method-names-starting-with-get)
- [Eliminating `!!` from your Kotlin code](#eliminating--from-your-kotlin-code)
  - [Use `val` instead of `var`](#use-val-instead-of-var)
  - [No `ArgumentCaptor`](#no-argumentcaptor)
  - [Use `lateinit`](#use-lateinit)
  - [Use `let` function](#use-let-function)
  - [Use `elvis` operator](#use-elvis-operator)
- [Annotations to make Kotlin interoperable with Java.](#annotations-to-make-kotlin-interoperable-with-java)
- [Migrating Unit Tests to Use `Mockito-Kotlin`](#migrating-unit-tests-to-use-mockito-kotlin)
- [Utils -> Kotlin-Extensions](#utils---kotlin-extensions)
- [Migrating Room to Kotlin](#migrating-room-to-kotlin)
- [Idiomatic Kotlin](#idiomatic-kotlin)
- [Acknowledgement ](#acknowledgement)
- [References](#references)


## Configure Kotlin

1. In Android Studio, select **Tools > Kotlin > Configure Kotlin in Project**. If a window titled **Choose Configurator** appears, select **Android with Gradle**, make sure **All modules** is selected, and click **OK**.
2. You will be prompted to sync the project with Gradle as the build.gradle files have changed. Once the sync is complete, move to the next step and write some Kotlin code.

**Note**: When using Android Studio 3.0 or higher and creating a new project, you can have Kotlin configured by default by selecting the **Include Kotlin support** checkbox on the Create Android Project dialog.

---

## Steps to **Convert**

Once you learn basics syntax of **Kotlin**

1. Convert files, one by one, via **"⌥⇧⌘K",** make sure tests still pass
2. Go over the Kotlin files and make them more [idiomatic](https://blog.philipphauer.de/idiomatic-kotlin-best-practices/).
3. Repeat step 2 until you convert all the files.
4. Ship it.

---

## Comparing the type system between `Java` and `Kotlin`



---

### `Any` v/s `Object`

All classes in Kotlin have a common superclass Any, that is a default super for a class with no supertypes declared:

```kotlin
class Example // Implicitly inherits from Any
```

> **Any is not java.lang.Object**; in particular, it does not have any members other than `equals()`, `hashCode()` and `toString()`. Please consult the Java interoperability section for more details.

Further, from the section on mapped types we find:

Kotlin treats some Java types specially. Such types are not loaded from Java “as is”, but are mapped to corresponding Kotlin types. The mapping only matters at compile time, the runtime representation remains unchanged. Java’s primitive types are mapped to corresponding Kotlin types (keeping platform types in mind):

> `java.lang.Object` `kotlin.Any!`

This says that at **runtime** `java.lang.Object` and `kotlin.Any!` are treated the same. But the `!`also means that the type is a platform type, which has implication with respect to disabling null checks etc.

> Any reference in Java may be null, which makes Kotlin’s requirements of strict null-safety impractical for objects coming from Java. Types of Java declarations are treated specially in Kotlin and called platform types. Null-checks are relaxed for such types, so that safety guarantees for them are the same as in Java (see more below).

> When we call methods on variables of platform types, Kotlin does not issue nullability errors at compile time, but the call may fail at runtime, because of a null-pointer exception or an assertion that Kotlin generates to prevent nulls from propagating:

---

## Type inference

```kotlin
// val abc: Map<String,Int> = mapOf("a" to 1, "b" to 2, "c" to 3)
// the actual type of the Map is obvious
// thus we can skip Map<String,Int> and let kotlin compiler to make type inference
val abc = mapOf("a" to 1, "b" to 2, "c" to 3)

val abcd = abc + ("d" to 4) // ok!
val abce = abc + ("e" to "5") // compile error: Type mismatch

// val c: Int? = abc["c"]
var c = abc["c"] 

c = "2" // compile error: Type mismatch

// Same Question: 
// If you are asked to refactor from Map<String,Integer> to Map<String,String>,
// try to estimate the efforts!
```



---

## What `nullability` means in Kotlin?
[Source](https://kotlinlang.org/docs/reference/null-safety.html)



---

## Why `Optionals` exist?



---

## How `data` classes are based on Value Types



---

### `class` v/s `data-class`



---

## Using `Parcelize` annotation for `Parcelable`
[Source](https://github.com/Kotlin/KEEP/blob/master/proposals/extensions/android-parcelable.md)

Here is the Java Class:

```java
public class MyParcelable implements Parcelable {
     private int mData;

     public int describeContents() {
         return 0;
     }

     public void writeToParcel(Parcel out, int flags) {
         out.writeInt(mData);
     }

     public static final Parcelable.Creator<MyParcelable> CREATOR
             = new Parcelable.Creator<MyParcelable>() {
         public MyParcelable createFromParcel(Parcel in) {
             return new MyParcelable(in);
         }

         public MyParcelable[] newArray(int size) {
             return new MyParcelable[size];
         }
     };
     
     private MyParcelable(Parcel in) {
         mData = in.readInt();
     }
 }
```

Converted Kotlin Class:

```kotlin
data class MyParcelable(var data: Int): Parcelable {

    override fun describeContents() = 1

    override fun writeToParcel(dest: Parcel, flags: Int) {
        dest.writeInt(data)
    }

    companion object {
        @JvmField 
        val CREATOR = object : Parcelable.Creator<MyParcelable> {
            override fun createFromParcel(source: Parcel): MyParcelable {
                val data = source.readInt()
                return MyParcelable(data)
            }

            override fun newArray(size: Int) = arrayOfNulls<MyParcelable>(size)
        }
    }
}
```

To use `@Parcelize` we need to set experimental flag in `build.gradle`

```groovy
androidExtensions {
    experimental = true
}
```

Android Extensions plugin now includes an automatic [Parcelable](https://developer.android.com/reference/android/os/Parcelable.html) implementation generator. Declare the serialized properties in a primary constructor and add a `@Parcelize` annotation, and `writeToParcel()`/`createFromParcel()` methods will be created automatically:

```kotlin
@Parcelize
class MyParcelable(val data: Int): Parcelable
```

---

## String templates

Kotlin has string templates, which is awesome. e.g. "$firstName $lastName" for simple variable name or "${person.name} is ${1 * 2}" for any expressions. You can still do the string concatenation if you like e.g. "hello " + "world", but that means being stupid.

---

## Removing `ButterKnife` and `findViewById` to use `Kotlin-Android-Extensions`





---

## `apply`, `let`, `with` and `also`



---

## Common Migration Gotchas

* **annotationProcessor** must be replaced by **kapt** in build.gradle
* Configure tests to mock final classes
* `@JvmField` to rescue while using ButterKnife `@InjectView` and Espresso `@Rule`

---

## If you are using android **data-binding**, include:

```groovy
kapt "com.android.databinding:compiler:${compiler_version}"
```

Fixes following the Gradle warning:
```
Warning:warning: The following options were not recognized by any processor: '[android.databinding.artifactType,
android.databinding.printEncodedErrors, android.databinding.minApi, android.databinding.isTestVariant,
android.databinding.enableDebugLogs, android.databinding.sdkDir, android.databinding.bindingBuildFolder,
android.databinding.enableForTests, android.databinding.modulePackage, kapt.kotlin.generated, 
android.databinding.generationalFileOutDir, android.databinding.xmlOutDir]'
```

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
    @Override public void displayMessageFromApi(String apiMessage) {
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
[Source](https://android.jlelse.eu/how-to-remove-all-from-your-kotlin-code-87dc2c9767fb)

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
   adapter = RecyclerAdapter(R.layout.item_droid)
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
   adapter = RecyclerAdapter(R.layout.item_droid)
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
   if (username != null) {
       return username!!
   } else {
       return "Anonymous"
   }
}
```

----

**elvis** operator is great when you have a fallback value for the null case. So you can replace this:

```kotlin
fun getUserName(): String {
   return username ?: "Anonymous"
}
```

---

## Annotations to make Kotlin interoperable with Java.



---

## Migrating Unit Tests to Use [`Mockito-Kotlin`](https://github.com/nhaarman/mockito-kotlin)



---

## Utils -> Kotlin-Extensions



---

## Migrating Room to Kotlin



---

## Idiomatic Kotlin



---

## Acknowledgement 

[**Arun Sasidharan**](https://github.com/esoxjem) for Initial Idea

[**Ritesh Gupta**](https://github.com/riteshhgupta) for More Ideas

---

## References

https://medium.com/fueled-android/practical-tips-to-migrate-your-android-app-to-kotlin-4d331e5256dc
https://codelabs.developers.google.com/codelabs/taking-advantage-of-kotlin/index.html
https://android.jlelse.eu/how-to-remove-all-from-your-kotlin-code-87dc2c9767fb
https://medium.com/google-developers/migrating-an-android-project-to-kotlin-f93ecaa329b7
https://blog.philipphauer.de/idiomatic-kotlin-best-practices/
https://stackoverflow.com/a/38761552/1257042
https://gist.github.com/gaplo917/c3b70249b660bff9cdc0b5453c34f98b

---
