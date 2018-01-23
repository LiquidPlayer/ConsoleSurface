# ConsoleSurface
A Node.js console UI for LiquidCore

Version
-------
[0.4.4](https://github.com/LiquidPlayer/ConsoleSurface/releases/tag/0.4.4) - Get it through [JitPack](https://jitpack.io/#LiquidPlayer/ConsoleSurface/0.4.4)

[![Release](https://jitpack.io/v/LiquidPlayer/ConsoleSurface.svg)](https://jitpack.io/#LiquidPlayer/ConsoleSurface)

A Node.js ANSI Console for Mobile Apps
--------------------------------------
A `ConsoleSurface` is a UI surface for use with [LiquidCore](https://github.com/LiquidPlayer/LiquidCore).  It provides an
ANSI-compatible Node.js console that can interact with a LiquidCore MicroService.  This surface is most useful for
debugging.

To see it in action, follow along with the tutorial below.  If you are looking for a more complex example, see [ExampleBitTorrentμService](https://github.com/LiquidPlayer/Examples/tree/master/ExampleBitTorrent%CE%BCService).

## "Hello, Console!" Example

(All of the code for this example can be found [here](https://github.com/LiquidPlayer/Examples/tree/master/HelloConsole).)

We will start by creating a very simple micro service.  This will be served from a machine on our network.  Start by creating a working directory somewhere.

```
$ mkdir ~/helloconsole
$ cd ~/helloconsole
```

Then, install the LiquidCore server (aptly named _LiquidServer_) from `npm`:
```
$ npm install -g liquidserver
```

Now let's create our micro service.  Create a file in the `~/helloconsole` directory called `console.js` and fill it with the following contents:

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

Okay, so here's what happening.  We are setting an indefinite timer so that the
process stays alive until we decide to kill it.  After that, we are requesting to attach to a
surface with `LiquidCore.attach(surface, callback)`.  The _surface_ parameter is the canonical
name of the surface (`org.liquidplayer.surface.console.ConsoleSurface`)
and _callback_ is a callback function that takes an error as its argument.  We are using
the arrow function (`=>`) notation here.  ES6 is for real and it's time to get used to it.  If you
want to use old-timey `function(error) { ... }` notation, that's ok too.

Once the surface has attached, which we'll know when our callback is called (with an undefined error),
then the surface is ready to use.  If _error_ is not undefined, then something went wrong and the _error_
parameter will be the thrown exception.

Next, let's set up a manifest file.  Don't worry too much about this right
now.  Basically, the manifest allows us to serve different versions based
on the capabilities/permissions given by the host.  But for our simple example,
we will serve the same file to any requestor.  Create a file in the same
directory named `console.manifest`:

```javascript
{
   "configs": [
       {
           "file": "console.js"
       }
   ]
}
```

You can now run your server.  Choose some port (say, 8080), or LiquidServer will create
one for you:

```
$ liquidserver 8080
```

You should now see the message, `Listening on port 8080`.  Congratulations, you just
created a micro service.  You can test that it is working correctly by navigating to
`http://localhost:8080/console.js` in your browser.  You should see the contents of
`console.js` that you just created with some wrapper code around it.  The wrapper is simply
to allow multiple Node.js modules to be packed into a single file.  If you were to
`require()` some other module, that module and its dependencies would get wrapped into this
single file.

You can leave that running or restart it later.  Now we need to create a host app.

1. In Android Studio, create a new project by selecting `File -> New Project ...`
2. Fill out the basics and press `Next` (Application Name: `HelloConsole`, Company Domain: `liquidplayer.org`, Package name: `org.liquidplayer.examples.helloconsole`)
3. Fill in the Target Devices information.  The defaults are fine.  Click `Next`
4. Select `Empty Activity` and then `Next`
5. The default options are ok.  Click `Finish`

Now link the `ConsoleSurface` library.  Go to your **root-level `build.grade`**
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

Then, add the ConsoleSurface library to your **app's `build.gradle`**:

```
dependencies {
    ...
	  implementation 'com.github.LiquidPlayer:ConsoleSurface:0.4.0'
	  implementation 'com.github.LiquidPlayer:LiquidCore:0.4.0'
}

```
(Note, replace `implementation` with `compile` if you are using older build tools)

Test it in the emulator (or on a device) to make sure all went to plan.  You should have a basic
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
  availableSurfaces: { 'org.liquidplayer.surface.console.ConsoleSurface': '0.4.0' },
  attach: [Function: attach_],
  detach: [Function: detach_] }
```

The `availableSurfaces` object will show the same set of surfaces that were enabled in the XML.  This way,
micro apps know what surfaces they can use.

Type `LiquidCore.detach()` in your console and Poof! Your surface is gone.  If you rotate the
device, a new instance will be launched and reattached.  Type `process.exit()`
and the same thing happens.

Of course it is possible to use a [`LiquidView`](https://liquidplayer.github.io/LiquidCoreAndroid/0.3.0/org/liquidplayer/service/LiquidView.html) programmatically and dynamically set its available surfaces
and URI.  The [ExampleBitTorrentμService](https://github.com/LiquidPlayer/Examples/tree/master/ExampleBitTorrent%CE%BCService) project uses LiquidView programmatically.

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
