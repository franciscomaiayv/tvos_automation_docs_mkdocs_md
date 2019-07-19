# Media Router YV

Aforementioned is the official documentation for TVOS automation with the MediaRouter.

Meant to be a concise representation of all the available methods available.

## start_daemon()

Starting the dbusdaemon.

As mentioned above, this is necessary for out MR object to be store
and not recycled.

```
dbussenddaemon &
```

## kill_daemon():

Kills daemon.

Send a dbus-message to the void send method. this does not care
about any of the ssh output

```
killall dbussenddaemon
```


## get_media_factory_address()

Gets the MR address

this sends a message through the dbus and returns the generated MR
address. this will also be stored in the dbus daemon.

Returns:
```
    string: generated MR address -> 'Zinc/Media/MediaRouters/X'
```

Same as executing in UNIX:

```
dbus-send --session --print-reply --dest='Zinc.DBusSendDaemon' \
/Zinc/Media/MediaRouterFactory \
Zinc.Media.MediaRouterFactory.createMediaRouter string:Zinc.Media \
string:"Zinc/Media/MediaRouters/ \

```

## deactivate_tuner()

Deactivate tuner

Sends a dbus message that soft deactivates the inboard DTT tuner.

Returns:
```
    string: the output of the dbus send command, might throw error
    but it must be caught by the automation script
```


# MediaRouterErrors(Enum)

This is a class that lists known errors, extremely useful for creating human readable
test. You can add new errors in this format:

```
MediaRouterErrors.InvalidLocator = "Zinc.Media.Error.InvalidLocator"
```

A example usage for this is as such:

```python
with then(""" I call MediaRouter.setSink() and THEN MediaRouter.setSink()
returns an error:"Zinc.Media.Error.IllegalReconfiguration"  """):

    if not MR.set_sink("string:decoder://0") == MediaRouterErrors.IllegalReconfiguration:
        fail("Zinc.Media.Error.IllegalReconfiguration not returned")

```


#class MediaRouter

Media router class for TVOS automation.

Can be initialised from a test. all sorts of interfaces are available.
We can generate, and modify a media router on demand.



*please note that we will often abbreviate Media router as MR.*

##MediaRouter.__init__()

Construsctor for the MediaRouter class
A bit longer description.

We start a deamon. this daemon will store our new
MR adress. dbus tends to recycle any generated object.
So for us to store the reference to out generated MR we
must start this dbus daemon.

*We also store the MR address locally.*


##MediaRouter.get_mr(self)

Returns the MR address **in use**

Returns:
```
    string: generated MR address -> 'Zinc/Media/MediaRouters/X'
```

##MediaRouter.recycle(self)

This will send a dbus command with the current MR and request a recycle.
we might timeout but this error must be handled by the automation
script.

```
Args:
    variable (type): description

Returns:
    str(stdout): the output of the dbus send command.
```

Same as executing:
```
dbus-send --session --print-reply --type=method_call --dest='Zinc.Media' $MR Zinc.Media.MediaRouter.recycle
```

##MediaRouter.start(self):

Sends a dbus message that starts the MR we generated
```
Returns:
    string: the dbus send output. this can throw an error but it
    must be caught by automation script
```

##MediaRouter.set_sink(self, string):

Sends a dbus message that sets the sink

Can throw an error but it
must be caught by automation script
Args:
    sink (string): MR sink variable
Return:
    classified error, as a MediaRouterErrors object.

Same as executing:

```
dbus-send --session --print-reply --type=method_call --dest='Zinc.Media' $MR Zinc.Media.MediaRouter.setSink string:decoder://0
```

###MediaRouter.get_sink(self)
Returns the current sink set in the MR

Can throw an error but it
must be caught by automation script

Returns:
    string: dbus send response

Same as executing:
```
dbus-send --session --print-reply --type=method_call --dest='Zinc.Media' $MR Zinc.Media.MediaRouter.getSink
```

##MediaRouter.stop(self)

Sends a stop signal to the current MR

Can throw an error but it
must be caught by automation script

Returns:
    string: dbus response

Same as executing:
```
dbus-send --session --print-reply --type=method_call --dest='Zinc.Media' '/Zinc/Media/MediaRouters/X' Zinc.Media.MediaRouter.start
```

##MediaRouter.set_source_and_play(self, uri)
Set source and play

Executes a compiled list of arguments that prepares a MR, sets a custom
URI as its source and plays it.

Args:
    URI (string): a uri for the media we should play

##MediaRouter.get_source(self)
Get source

Sends a dbus message that will return the source of the current MR

Returns:
    source (string): uri for the current MR

##MediaRouter.get_playspeed(self)

Sends a dbus message that will return the play speed of the current MR

Returns:
    playSpeed (string): play speed for the current MR

Same as executing:
```
dbus-send --session --print-reply --type=method_call --dest='Zinc.Media' $MR Zinc.Media.MediaRouter.getPlaySpeed
```

##MediaRouter.call_custom_method(self, method, _*args_)
Call a custom method

Sends a dbus message that will call on a methods specified

Arguments can be trailed to that argument.

Do note that arguments have to be "pre parsed" so:

if you want to send,
```
string:'a/string/to/send' or int32:100
```
you must,
```python
call_custom_method("methodToCall", "string:'a/string/to/send'","int32:100")
```
which will result in,
```
dbus-send --session --print-reply --type=method_call --dest='Zinc.Media' $MR Zinc.Media.MediaRouter.methodToCall string:'a/string/to/send' int32:100
```

Returns:
    method response (string): method's response

##MediaRouter.set_source(self, uri)

Sets the source for the current MR. we can use this for overiding the
current source too, although this will likely break the MR

It will returned a classified error or a response string.

Args:
    URI (string): uri for the media we wish to set for the MR

```
dbus-send --session --print-reply --type=method_call --dest='Zinc.Media' $MR Zinc.Media.MediaRouter.setSource string:"http://filegateway.youview.co.uk/test_assets/restricted/VA-237.mpg" int32:0
```

##MediaRouter.set_source_with_int(self, uri, integer)

This will set the source for the current MR and append a int to the
dbus arguments, this is useful for some tests that require this.

Args:
    uri (string): uri for the media to set this MR for

    integer (string): additional argument for the dbus send
```
dbus-send --session --print-reply --type=method_call --dest='Zinc.Media' $MR Zinc.Media.MediaRouter.setSource string:"http://filegateway.youview.co.uk/test_assets/restricted/VA-237.mpg" int32:0
```

##MediaRouter.seek_position(self, arg)

Send a dbus message with a sink argument. this will call on the seek
position method.

It will returned a classified error or a response string.

Args:
    arg (string): seek arguments

```
dbus-send --session --print-reply --type=method_call --dest='Zinc.Media' $MR Zinc.Media.MediaRouter.seekPosition int32:1 int32:600000 int32:4
```

##MediaRouter.get_buffer_status(self)

Sends a dbus message that will return the buffer status
of the current MR.

Returns:
    buffer status (string): buffer status for the current MR

```
dbus-send --session --print-reply --type=method_call --dest='Zinc.Media' $MR Zinc.Media.MediaRouter.getBufferStatus
```

##MediaRouter.get_position(self)

sends a dbus message that will return the position
of the current MR

Returns:
    position (string): media position for the current MR

```
dbus-send --session --print-reply --type=method_call --dest='Zinc.Media' $MR Zinc.Media.MediaRouter.getPosition
```

##MediaRouter.start_buffering(self)

Sends a dbus message that will start the buffering
of the current MR

Returns:
    buffer start buerring (string): buffering start response

```
dbus-send --session --print-reply --type=method_call --dest='Zinc.Media' $MR Zinc.Media.MediaRouter.startBuffering
```

##MediaRouter.set_media_duration(self, arg)

This will send a message through the dbus that sets a MR duration

Args:
    arg (string): a set of arguments to add onto them setMediaDuration
    call

```
dbus-send --session --print-reply --type=method_call --dest='Zinc.Media' $MR Zinc.Media.MediaRouter.setMediaDuration int32:-1
```

##MediaRouter.cleanup(self)

This will perform some internal methods that clean the generated MR.
this is useful in between tests or on thrown errors.

This does,
```
self.stop()
self.recycle()
```

# Tests Example

Here you will find two tests. Please Have a look through these. They were selected
because they are great showcases of the capabilities of the MediaRouter library.

These tests inlcude:
```python
import logging
import time
from utils.testing import (
    fail,
    given,
    then,
    when
)
from tvos.mediarouter import (
    MediaRouter,
    MediaRouterErrors
)
from nextgen.video_signal import (
    wait_for_video,
    wait_for_no_video
)
import recover
logger = logging.getLogger(__name__)
```

##Test #1

```python
@attr(trid="C496887", labels=(TL.YOUVIEW_SPECIFICATION_TESTS, TL.SSH))
def run_test_recycle_active_mr():
    try:
        with given(""" Spec Paragraph/Functional Area:
            MediaRouter Specification (1125-S) Version 1.3E, Section 4.5,
            GIVEN STB is connected to an
            display device through HDMI
            AND I create a PDL MediaRouter and start it """):
            MR = MediaRouter()
            MR.set_source_and_play(
                "http://filegateway.youview.co.uk/test_assets/licence_free/VA-503.ts"
            )
            wait_for_video(20)
            time.sleep(20)
        with when(""" WHEN I call MediaRouter.recycle() """):
            MR.recycle()
        with then(""" THEN the video output on the connected
            display device will be blanked  """):
            wait_for_no_video(20)
            MR.cleanup()
    except:
        MR.cleanup()

```

##Test #2
```python
@attr(trid="C496887", labels=(TL.YOUVIEW_SPECIFICATION_TESTS, TL.SSH))
def run_test_recycle_active_mr():
    try:
        with given(""" Spec Paragraph/Functional Area:
            MediaRouter Specification (1125-S) Version 1.3E, Section 4.5,
            GIVEN STB is connected to an
            display device through HDMI
            AND I create a PDL MediaRouter and start it """):
            MR = MediaRouter()
            MR.set_source_and_play(
                "http://filegateway.youview.co.uk/test_assets/licence_free/VA-503.ts"
            )
            wait_for_video(20)
            time.sleep(20)
        with when(""" WHEN I call MediaRouter.recycle() """):
            MR.recycle()
        with then(""" THEN the video output on the connected
            display device will be blanked  """):
            wait_for_no_video(20)
            MR.cleanup()
    except:
        MR.cleanup()

```

##Test #3

Error detection is a big part of TVOS testing. In the following test we will showcase how
we can detect a non included error.

The test goes as follows:
```
@attr(trid="C496874", labels=(TL.YOUVIEW_SPECIFICATION_TESTS, TL.SSH))
def run_test_invalid_enumeration():
    with given(""" Spec Paragraph/Functional Area: MediaRouter Specification (1125-S) Version 1.3E, Annex A MediaRouter API, A.1.19,
    I create a PDL MediaRouter: """):
        MR = MediaRouter()
        MR.set_source("http://filegateway.youview.co.uk/test_assets/restricted/VA-237.mpg")
        MR.set_sink("string:decoder://0")
        pass
    with then(""" THEN MediaRouter.seekPosition() with invalid arguments returns an error:"Zinc.Media.Error.InvalidEnumeration"  """):
        if not MR.seek_position("int32:1", "int32:600000", "int32:4") == MediaRouterErrors.InvalidEnumeration:
            fail("No error returned")
        pass
    with when(""" AND I call MediaRouter.start()"""):
        MR.start()
        pass
    with when(""" AND MediaRouter.seekPosition() with invalid arguments returns an error:"Zinc.Media.Error.InvalidEnumeration" """):
        if not MR.seek_position("int32:7", "int32:600000", "int32:1") == MediaRouterErrors.InvalidEnumeration:
            fail("No error returned")
        pass
```

The above test gets a dbus resoonse like this:

```
Error org.freedesktop.DBus.Error.InvalidArgs: Could not convert int->enum\c Invalid enum value
```

This error is not known, so we can listen to it as follows:


```
@attr(trid="C496874", labels=(TL.YOUVIEW_SPECIFICATION_TESTS, TL.SSH))
def run_test_invalid_enumeration_force_error():
    with given(""" Spec Paragraph/Functional Area: MediaRouter Specification (1125-S) Version 1.3E, Annex A MediaRouter API, A.1.19,
    I create a PDL MediaRouter: """):
        MR = MediaRouter()
        MR.set_source("http://filegateway.youview.co.uk/test_assets/restricted/VA-237.mpg")
        MR.set_sink("string:decoder://0")
        pass
    with then(""" THEN MediaRouter.seekPosition() with invalid arguments returns an error:"Zinc.Media.Error.InvalidEnumeration"  """):
        if not "org.freedesktop.DBus.Error.InvalidArgs" in MR.seek_position("int32:1", "int32:600000", "int32:4"):
            fail("No error returned")
        pass
    with when(""" AND I call MediaRouter.start()"""):
        MR.start()
        pass
    with when(""" AND MediaRouter.seekPosition() with invalid arguments returns an error:"Zinc.Media.Error.InvalidEnumeration" """):
        if not "org.freedesktop.DBus.Error.InvalidArgs" in MR.seek_position("int32:7", "int32:600000", "int32:1"):
            fail("No error returned")
        pass
```
