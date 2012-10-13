!SLIDE
# Scala on Android

<img src="hello/devfest.png"/>
<img src="hello/droid-scala.png" class="centered"/>

<br/>
<br/>
@jberkel

!SLIDE

# /me

<br/>

  * From Java to Ruby and back
  * Work on the official SoundCloud native apps
  * Would love to see more people use Scala on Android, build community

!SLIDE

# Why not Java ?

<img src="hello/java_works.jpg" height="100%" class="centered"/>

!SLIDE

# It works ?

* Java is mature, stable ... and also a bit boring.
* Java 1.0 released in 1996 (16 years ago!)
* Slow Java Community Process, even slower with Oracle
* Java 8 won't ship until Summer 2013 (earliest)
* Oracle decided on 2 year release cycle - Java 9 in 2015
* Google/Oracle lawsuit



!SLIDE

# Java roadmap

## Java 7

* Type inference for generics // List&lt;String&gt; list = new ArrayList<>();
* Strings in switch // switch(foo) { case "bar": ... }
* Catching multiple exception types // try { } catch (IOException | SQLException)

## Java 8

* Closures // { String x -> x.length() == 0 }
* Multiple inheritance // "virtual extension methods"

!SLIDE

# But

* Android doesn't even support Java 7 at the moment
* Java is a conservative language
* Innovation happens elsewhere

!SLIDE

# Scala

* Modern, multi-paradigm language compiling to JVM bytecode
* Powerful type system (+ inference)
* Stronger object orientation, multiple inheritance
* Focus on functional (closures, immutability)
* Compatible w/ existing Java code
* Has already a lot of the features found in Java 7 + 8

!SLIDE

# JVM vs Dalvik

* at the core: Linux + DalvikVM (not JVM!)
* Officially supported: Java + C/C++ (NDK)

<a
href="https://docs.google.com/drawings/d/1LyNAij06eaHNxHpGczNApUdU84-8mVC8uEAkNGfj5hM/edit?pli=1&hl=en_GB">
<embed src="hello/jvm-dex-empty.svg" height="90%" type="image/svg+xml"/>
</a>

!SLIDE

# Not just Java
<a
href="https://docs.google.com/drawings/d/11ccszWUtTul1DWpvbFBBhZlv_NTLaoJSrxPb3cd4U3Y/edit">
<embed src="hello/jvm-dex.svg" height="90%" type="image/svg+xml"/>
</a>

!SLIDE

# Some useful Scala features

<br/>

## * Traits

!SLIDE


### Android is old-school framework design

<br>

    public class MyActivity extends Activity {
      @Override public void onCreate(Bundle b) {
        super.onCreate(b);
      }
    }

<br/>

!SLIDE

## Move everything to a base class

<br>

    public class BaseActivity extends Activity {
      @Override public void onCreate(Bundle b) {
        // shared behaviour
      }
    }

    public class MyActivity extends BaseActivity {
      @Override public void onCreate(Bundle b) {
        super.onCreate(b);
      }
    }

!SLIDE

# Traits to split concerns

<br/>

    trait BatteryAware extends Activity {
      override def onCreate(b: Bundle) {
        // register battery handler
      }
      override def onPause() { ... }
    }
    trait Logger extends Activity { ... }

    class MyActivity extends Activity
      with BatteryAware
      with Logger {
    }

!SLIDE

# Some useful Scala features #2

<br/>

## * Closures

!SLIDE

# useful for callbacks

<br/>

    locationManager.addGpsStatusListener(
      new GpsStatusListener() {
        public void onGpsStatusChanged(int evt) {
          System.out.printn(evt)
        }
      }
    }
    // vs.
    locationManager.addGpsStatusListener(evt => println(evt))

!SLIDE

# or to create nicer APIs

<br/>

    // query content provider
    Cursor c = resolver.query(...);
    List<MyModel> list = new ArrayList<MyModel>();
    // iterate over rows and create objects
    while (c != null && c.moveToNext()) {
        list.add(MyModel.fromCursor(c));
    }
    if (c != null) c.close(); // release resource
    return list;

<br/>


!SLIDE

# vs

<br/>

    return resolver.query(...) { cursor =>
      cursor.map(MyModel.fromCursor(_))
    }

!SLIDE

# making the cursor iterable

<br/>

    class BetterCursor(c: Cursor) extends Iterable[Cursor] {
      def iterator = new Iterator[Cursor] {
        def hasNext = c.getCount > 0 && !c.isLast
        def next() = { c.moveToNext(); c }
      }
    }

!SLIDE

# managing resources

    def query[T](uri: Uri)(fun: Cursor => T) = {
        val cursor = resolver.query(uri)
        try {
          fun(cursor)
        } finally {
          cursor.close()
        }
    }

!SLIDE

# Why you shouldn't use Scala (yet)

!SLIDE

# Problems #1 memory

<br/>

### runtime assumptions based on JVM:

* object allocation is cheap
* efficient and fast GC

<br/>
However, Android developer guidelines say:

<blockquote>
<p>
Avoid Creating Unnecessary Objects
</p>
</blockquote>

But performance is improving (parallel GC, JIT etc)

!SLIDE

# Problems #2

## Scala programs need a runtime lib

<br/>

    $ ls -lh scala-library.jar
    rw-r--r--  jan  wheel  8.5M 24 May  2011 scala-library.jar

    $ jar tfv scala-library.jar | wc -l
      5480

!SLIDE

# Solution

  * use proguard / treeshaker before dexing
  * preinstall Scala libs on the phone

<br/>

### Proguard is slow, long dev cycles

### Only run it before releasing

!SLIDE

# Problems #3

Scala / Java / Android interop problems


![](hello/scala-bug.png)

[issues.scala-lang.org/browse/SI-4620](https://issues.scala-lang.org/browse/SI-4620)

!SLIDE

# Problems #4 
## scalac is slow. Like 4x slower.

    public class Test { }

<br/>

    $ time javac Test.java
    real  0m0.630s
    user  0m1.091s
    sys 0m0.070s

    $ time scalac Test.scala
    real  0m2.340s
    user  0m3.908s
    sys 0m0.389s

!SLIDE

# When to use Scala

<br/>

  * Suitable for most types of apps
  * Not for realtime (i.e. games)
  * Lack of experience / libraries
  * If unsure, start with Scala tests

!SLIDE
# Tools

  * android-maven-plugin
  * sbt-android-plugin
  * Intellij IDEA CE (fsc + Scala / Android facets)

<br/>

 * [code.google.com/p/maven-android-plugin](http://code.google.com/p/maven-android-plugin/)
 * [github.com/jberkel/android-plugin](https://github.com/jberkel/android-plugin)
 * [www.jetbrains.com/idea/download/](http://www.jetbrains.com/idea/download/)

!SLIDE

# Thanks!

## need help? ask scala-on-android

![](hello/group.png)

[groups.google.com/forum/#!forum/scala-on-android](https://groups.google.com/forum/#!forum/scala-on-android)
[jberkel.github.com/presentation-scala-on-android](http://jberkel.github.com/devfest-scala)
