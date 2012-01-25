!SLIDE
# Scala on Android

<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
Jan Berkel (@jberkel)

!SLIDE

# Quick intro

* From 1.0 to 4.0 in < 4 years
* 700k activations *every* day
* 10 billion app downloads (both Dec 2011)


![](hello/trend.png)
![](hello/market.jpg)

!SLIDE

# Basics

* at the core: Linux + DalvikVM (not JVM!)
* Officially supported: Java + C/C++ (NDK)

<a
href="https://docs.google.com/drawings/d/1LyNAij06eaHNxHpGczNApUdU84-8mVC8uEAkNGfj5hM/edit?pli=1&hl=en_GB">
<embed src="hello/jvm-dex-empty.svg" height="90%" type="image/svg+xml"/>
</a>

!SLIDE

# What's this Dalvik thing?

<br/>

 * Dalvik bytecode != JVM bytecode
 * optimised for smaller devices
 * one VM per process (forked)
 * used to be very slow, JIT since 2.2

!SLIDE

# Not just Java
<a
href="https://docs.google.com/drawings/d/11ccszWUtTul1DWpvbFBBhZlv_NTLaoJSrxPb3cd4U3Y/edit">
<embed src="hello/jvm-dex.svg" height="90%" type="image/svg+xml"/>
</a>

!SLIDE

# Why Scala on Android?

<br/>

### Android is Java framework design, ca. 2001

<br>

    public class MyActivity extends android.app.Activity {

      @Override public void onCreate(Bundle b) {
        super.onCreate(b);
        BatteryHelper.initialize(this);
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

    class MyActivity extends Activity
      with BatteryAware
      with Logger {
    }

!SLIDE

# Android & APIs

![](hello/api.png)

## actually not so bad.

!SLIDE
# But sometimes awkward

<br/>

    Cursor  c = resolver.query(...);
    List<MyModel> l = new ArrayList<MyModel>();

    while (c != null && c.moveToNext()) {
        l.add(new MyModel.fromCursor(c));
    }

    if (c != null) c.close();

## ugh.

!SLIDE

# Let's make a nice functional Cursor

<br/>

    class NiceCursor(c: Cursor) extends Iterable[Cursor] {
      def iterator = new Iterator[Cursor] {
        def hasNext = c.getCount > 0 && !c.isLast
        def next() = { c.moveToNext(); c }
      }
    }
    implicit def NiceCursor(c: Cursor) = new NiceCursor(c)

!SLIDE

# and a better query interface

    def query[T](uri: Uri)(fun: Cursor => T) = {
        val cursor = resolver.query(uri)
        try {
          fun(cursor)
        } finally {
          cursor.close()
        }
    }
    // one liner
    val l = query(...)(_.map(MyModel.fromCursor(_)))

!SLIDE

# shorter callbacks

## pimp my library
<br/>

    locationManager.addGpsStatusListener(
      new GpsStatusListener() {
        public void onGpsStatusChanged(int evt) {
          System.out.printn(evt)
        }
      }
    }
    // vs.
    locationManager.addGpsStatusListener(println(_))

!SLIDE

# Testing on Android
## ...sucks.

<br/>

  * slow
  * verbose test code

!SLIDE

# Scala for testing Android apps

## ScalaMock

 * Uses a compiler plugin to generate mocks
 * Currently not able to mock Android core classes

<br/>
[https://github.com/paulbutcher/scalamock](https://github.com/paulbutcher/scalamock)

!SLIDE

# robolectric / robospecs

<br>

  * Testable reimplementation of the SDK
  * classloading tricks allow test execution on development machine
  * works with specs2 and ScalaTest

<br>
[https://github.com/pivotal/robolectric](https://github.com/pivotal/robolectric)
[https://github.com/jbrechtel/robospecs](https://github.com/jbrechtel/robospecs)

!SLIDE

# ScalaTest Example

    lazy val track = ...

    it should "insert an read back a track" in {
      val uri = provider.insert(Content.TRACKS.uri,
        track.buildContentValues())

      val read = query(uri, 1)(_.map(new Track(_))).head
      read.id should equal(track.id)
      read.title should equal(track.title)
      // etc.
    }

!SLIDE
# problems with Scala

### many runtime assumptions:

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

# problems #2

## Scala programs need a runtime lib


!SLIDE

# solution

  * use proguard / treeshaker before dexing
  * preinstall Scala libs on the phone

<br/>

### Proguard is slow, long dev cycles

### Only run it before releasing

!SLIDE

# problems #3

Scala / Android interop problems


![](hello/scala-bug.png)

[https://issues.scala-lang.org/browse/SI-4620](https://issues.scala-lang.org/browse/SI-4620)

!SLIDE

## Android is open source (sometimes)

![](hello/android-bug.png)

[https://android-review.googlesource.com/#/c/30900/](https://android-review.googlesource.com/#/c/30900/)

!SLIDE

# When to use Scala

<br/>

  * Not good for realtime apps (i.e. games)
  * if unsure, start with Scala tests

!SLIDE
# Tools

  * sbt-android-plugin
  * Intellij IDEA (fsc + Scala / Android facets)
  * positronic lib

<br/>

[https://github.com/jberkel/android-plugin/](https://github.com/jberkel/android-plugin/)
[https://github.com/rst/positronic_net](https://github.com/rst/positronic_net)


!SLIDE
# How to get started

<br/>

## Demo time...

!SLIDE

# Thanks!

## scala-on-android

![](hello/group.png)

[https://groups.google.com/forum/#!forum/scala-on-android](https://groups.google.com/forum/#!forum/scala-on-android)
