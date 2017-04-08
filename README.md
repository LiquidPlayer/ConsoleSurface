# ConsoleSurface
A Node.js console UI for LiquidCore

### Hello, Console!

(All of the code for this example can be found [here](https://github.com/LiquidPlayer/Examples/tree/master/HelloConsole).)

Start by creating a new micro service, named `console.js` just like we did for the _Hallo, die Weld!_ example above, and fill it with the following:

```javascript
// This will keep the process alive until an explicit process.exit() call
setInterval(function(){},1000)

// This will attempt to attach to a ConsoleSurface
LiquidCore.attach('org.liquidplayer.surface.console.ConsoleSurface', (error) => {
    console.log("Hello, Console!")
    if (error) {
        console.error(error)
    }
})
```

Okay, so here's what happening.  We are once again setting in indefinite timer so that the
process stays alive until we decide to kill it.  After that, we are requesting to attach to a
surface with `LiquidCore.attach(surface, callback)`.  The _surface_ parameter is the canonical
name of the surface (`org.liquidplayer.surfaces.console.ConsoleSurface` is so far the only
choice) and _callback_ is a callback function that takes an error as its argument.  We are using
the arrow function (`=>`) notation here.  ES6 is for real and it's time to get used to it.  If you
want to use old-timey `function(error) { ... }` notation, that's ok too.

Once the surface has attached, which we'll know when our callback is called (with an undefined error),
then the surface is ready to use.  If _error_ is not undefined, then something went wrong and the _error_
parameter will be the thrown exception.

Don't forget our manifest file, `console.manifest`:

```javascript
{
   "configs": [
       {
           "file": "console.js"
       }
   ]
}
```

Run the server again and make sure you can navigate to the file correctly.

Now, let's create another host app.  We'll do exactly as we did above, only we will change the name:

1. In Android Studio, create a new project by selecting `File -> New Project ...`
2. Fill out the basics and press `Next` (Application Name: `HelloConsole`, Company Domain: `liquidplayer.org`, Package name: `org.liquidplayer.examples.helloconsole`)
3. Fill in the Target Devices information.  Select API 17 as the minimum and click `Next`
4. Select `Empty Activity` and then `Next`
5. The default options are ok.  Click `Finish`

As before, make sure you link the LiquidCore library.  Go to your **root-level `build.grade`**
file and add the `jitpack` dependency:

```
...

allprojects {
    repositories {
        jcenter()
        maven { url 'https://jitpack.io' }
    }
}

...
```

Then, add the LiquidCore library to your **app's `build.gradle`**:

```
dependencies {
    ...
	  compile 'com.github.LiquidPlayer:ConsoleSurface:0.2.1'
}

```

Test it in the emulator (or on a device) to make sure all went to plan.  You should have your
"Hello World!" screen.

We need to create a list of allowed surfaces.  For this, we will create an array resource.  Right-click
on `res/values` and click `New -> Values resource file`.  Set the filename as `arrays` and leave the rest
alone.  Click `OK`.  Fill that file with:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string-array name="surfaces">
        <item>org.liquidplayer.surface.console.ConsoleSurface</item>
    </string-array>
</resources>
```

This is a clunky Android thing.  We want to reference an array of surfaces, so we have to do it in
a separate resource file (eye roll).

Now, open the `res/layout/activity_main.xml` file.  We are going to make one simple change and then be done.  Change the file to the following, but replace the IP address with _your_ server:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:custom="http://schemas.android.com/apk/res-auto"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="org.liquidplayer.examples.helloconsole.MainActivity">

    <org.liquidplayer.service.LiquidView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/liquidview"
        custom:liquidcore.URI="http://192.168.1.152:8080/console.js"
        custom:liquidcore.surface="@array/surfaces"
        />
</RelativeLayout>
```

Now restart the app.  You should see a fully functional Node.js console.  Try typing `process.versions` and you will see the versions of the various libraries.

LiquidCore provides a `LiquidView` Android view widget.  This provides the container in which surfaces can
be attached.  You must specify which surfaces you want to make available to the micro app.  This gives hosts
ownership over what level of control they want to give to micro apps, which will be necessary as more
elaborate surfaces become available.

In your console, type `LiquidCore` and press enter.  You should see:
```javascript
LiquidCore_ {
  domain: null,
  _events: {},
  _eventsCount: 0,
  _maxListeners: undefined,
  availableSurfaces: { 'org.liquidplayer.surface.console.ConsoleSurface': '0.2.1' },
  attach: [Function: attach_],
  detach: [Function: detach_] }
```

The `availableSurfaces` object will show the same set of surfaces that were enabled in the XML.  This way,
micro apps know what surfaces they can use.

Type `LiquidCore.detach()` in your console and Poof! Your surface is gone.  If you rotate the
device, a new instance will be launched and reattached.  Type `process.exit()`
and the same thing happens.

Of course it is possible to use a [`LiquidView`](https://liquidplayer.github.io/LiquidCoreAndroid/v0.2.1/org/liquidplayer/service/LiquidView.html) programmatically and dynamically set its available surfaces
and URI.

License
-------

 Copyright (c) 2014-2017 Eric Lange. All rights reserved.

 Redistribution and use in source and binary forms, with or without
 modification, are permitted provided that the following conditions are met:

 - Redistributions of source code must retain the above copyright notice, this
 list of conditions and the following disclaimer.

 - Redistributions in binary form must reproduce the above copyright notice,
 this list of conditions and the following disclaimer in the documentation
 and/or other materials provided with the distribution.

 THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

[Node.js]:https://nodejs.org/
[Android Studio]:https://developer.android.com/studio/index.html
[BigNumber]:https://github.com/MikeMcl/bignumber.js/
