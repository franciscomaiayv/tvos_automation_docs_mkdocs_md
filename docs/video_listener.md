# Video Signal Listen

This is not part of the MediaRouter library but it is a useful tool for automation
of a lot of the tests in MediaRouter pack.

Usage, Imports:

```
from nextgen.video_signal import (
    wait_for_video,
    wait_for_no_video
)
```

Usage:
```
wait_for_video(20)

wait_for_no_video(20)

```

The argument passed *(int)* is the time we should wait for a video signal or no
video signal in **seconds**.
