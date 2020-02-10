# Go-like programming patterns through the ages

This post intends to showcase programming patterns from Alef, to Limbo, to Go.

All of these code snippets should be complete in their own forms and compilable/runnable.

If you want to play with Alef in 2020, your best bet is probably installing a Plan9 2nd edition VM. A text guide for this process is [here](https://seh.dev/2018/03/19/0/) and a video guide is [here](https://www.youtube.com/watch?v=W00TnQ91nj8).

There's a work-in-progress port of Alef from 2e to 9front/386 which can be found on the [public grid](http://9gridchan.org) griddisk at `/burnzez/rep/alef/root` and maybe `/burnzez/alef`.

If you want to play with Limbo in 2020, your best bet is either the official Inferno [here](https://bitbucket.org/inferno-os/inferno-os/) or the purgatorio fork [here](https://code.9front.org/hg/purgatorio).

## Coroutine spawning

### Alef



### Limbo

[co.b](./co.b)

```
implement Coroutines;

include "sys.m";
	sys: Sys;

include "draw.m";

Coroutines: module {
	init: fn(nil: ref Draw->Context, nil: list of string);
};

double(n: int) {
	sys->print("%d\n", 2*n);
}

init(nil: ref Draw->Context, nil: list of string) {
	sys = load Sys Sys->PATH;

	spawn double(7);

	sys->sleep(5);

	exit;
}
```

### Go

[co.go](./co.go)

```
package main

import (
	"fmt"
	"time"
)

func double(n int) {
	fmt.Println(2*n)
}

func main() {
	go double(7)
	time.Sleep(5 * time.Millisecond)
}
```

## Unnamed struct members



## CSP elements



## Multiple returns (??)



##



