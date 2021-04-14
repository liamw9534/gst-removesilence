# gst-removesilence

A standalone repo for the `removesilence` plugin for gstreamer 1.14.
The standard 1.14 `removesilence` plugin (part of gst-bad-plugins repo) lacks many of
the features of the plugin shipped with 1.19, so this is a back port so that
those features can be enjoyed with gstreamer 1.14, which is the de-facto release in
Ubuntu 18.04 and Raspbian Buster.

There is also an additional feature added which introduces a new boolean property called
`gate`.  This is useful in pipelines where you want to dynamically gate the output on/off
irrespective of the input stream e.g., using it as a VAD during speech processing.

## Compiling

Configure and build as such:

    meson builddir
    ninja -C builddir

See <https://mesonbuild.com/Quick-guide.html> on how to install the Meson
build system and ninja.

Once the plugin is built you can either install it system-wide with `sudo ninja
-C builddir install` (however, this will by default go into the `/usr/local`
prefix where it won't be picked up by a `GStreamer` installed from packages, so
you would need to set the `GST_PLUGIN_PATH` environment variable to include or
point to `/usr/local/lib/gstreamer-1.0/` for your plugin to be found by a
from-package `GStreamer`).

Alternatively, you will find your plugin binary in `builddir/gst-plugins/src/`
as `libgstplugin.so` or similar (the extension may vary), so you can also set
the `GST_PLUGIN_PATH` environment variable to the `builddir/gst-plugins/src/`
directory (best to specify an absolute path though).

You can also check if it has been built correctly with:

    gst-inspect-1.0 builddir/gst-plugins/src/libgstremovesilence.so

[MIT]: http://www.opensource.org/licenses/mit-license.php or COPYING.MIT
[LGPL]: http://www.opensource.org/licenses/lgpl-license.php or COPYING.LIB
[Licensing]: https://gstreamer.freedesktop.org/documentation/application-development/appendix/licensing.html

## Usage

I personally use this plugin in conjunction with my `snowboy` gst plugin.  Refer to
https://github.com/liamw9534/snowboy/tree/gst-plugin/examples/C%2B%2B/gstreamer/gst-snowboy
for more information on this gst plugin.

The idea is that we use `snowboy` as a hotword detector and then grab subsequent samples
from utterences _after_ the hotword has been spoken via `removesilence`.  The `removesilence`
plugin can trigger bus messages to tell us when the user has stopped speaking through the
bus event `silence_detected`.  We have to be careful to tune the `minimum_silence_time` property
correctly to ensure normal speaking pauses are not stripped out of the audio stream.

Example pipeline with 2 seconds `minimum_silence_time` and starts up gated:

    pulsesrc ! snowboy resource=... models=... sensitivity=... ! removesilence gate=true minimum_silence_time=2000000000 silent=0 ! appsink

Note that an application would then dynamically set the property `gate` to `false` after the hotword is detected
(refer to `hotword-detect` signal in `snowboy` plugin) and then `gate` to `true` after the `silence_detected` bus event
is emitted by `removesilence`.

## Why do it this way?

I've seen lots of code on the internet written in python to process audio samples in real-time to detect
silence periods.  This is simply not a good idea.  GStreamer excels at creating audio processing pipelines
and can be wrapped-up in a python application using its gst bindings.
