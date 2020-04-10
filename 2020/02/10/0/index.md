# Legacy of Go - CSP and otherwise

This post intends to showcase programming patterns from Newsqueak, Alef, Plan9 C, Limbo, to Go.

(??) ­ Include different Plan9 edition releases to see the libc evolve?

(???) ­ Include unix examples for shade?

All of these code snippets should be complete as shown and compilable/runnable in the state presented.

If you want to play with Newsqueak, the unix port of squint is probably the most straightforward, found [here](https://github.com/rwos/newsqueak). The papers describing Newsqueak is [here](https://swtch.com/~rsc/thread/newsqueak.pdf) and [here](https://swtch.com/~rsc/thread/newsquimpl.pdf).

If you want to play with Alef in 2020, your best bet is probably installing a Plan9 2nd edition VM. A text guide for this process is in a prior blog post [here](https://seh.dev/2018/03/19/0/) and a video guide is [here](https://www.youtube.com/watch?v=W00TnQ91nj8).

There are more resources on Plan9 2nd edition indexed [here](http://9.postnix.pw/hist/2e/).

There's a work-in-progress port of Alef from 2e to 9front/386 which can be found on the [public grid](http://wiki.9gridchan.org/public_grid/index.html) griddisk at `/burnzez/rep/alef/root` and maybe `/burnzez/alef`. Griddisk is accessible over 9p via `tcp!45.63.75.148!9564`. You can more easily access the grid from unix via the [gridnix scripts](https://github.com/henesy/grid-unix).

Papers on Alef are [here](http://doc.cat-v.org/plan_9/2nd_edition/papers/alef/).

If you want to play with Limbo in 2020, your best bet is either the official Inferno [here](https://bitbucket.org/inferno-os/inferno-os/) or the purgatorio fork [here](https://code.9front.org/hg/purgatorio).

If you want to play with Plan9 in 2020, your best bet is 9front, found [here](http://9front.org/).

## Building and running examples

### Newsqueak

To run a program from a prompt:

	$ squint foo.nq
	# output, if any
	$

### Alef

From a prompt on a complete Plan9 2e installation:

	term% 8al foo.l
	term% 8l co.8
	term% 8.out
	# output, if any
	term%

### Plan9 C

From a 386 system:

	term% 8c foo.c
	term% 8l foo.8
	term% 8.out
	# output, if any
	term%

From an amd64 system:

	term% 6c foo.c
	term% 6l foo.6
	term% 6.out
	# output, if any
	term%

Arm uses `5?` in its commands, etc. as per the manuals 2c(1) and 2l(1).

### Limbo

From a prompt inside the Inferno VM:

	; limbo foo.b
	; foo
	# output, if any
	;

### Go

To run a single file program:

	$ go run foo.go
	# output, if any
	$

## Intro ­ tokenizing

### Newsqueak

Nope.

### Alef

[tok.l](./tok.l)

```
#include <alef.h>

#define NTOKS 9
#define MAXTOK 512
#define str "abc » 'test 1 2 3' !"

void
main(void)
{
	int n, i;
	byte *toks[MAXTOK];

	print("%s\n", str);

	n = tokenize(str, toks, NTOKS);

	for(i = 0; i < n; i++)
		print("%s\n", toks[i]);

	exits(nil);
}
```

### Plan9 C

[tok.c](./tok.c)

```
#include <u.h>
#include <libc.h>

#define NTOKS 9
#define MAXTOK 512
char *str = "abc ☺ 'test 1 2 3' !";

void
main(int, char*[])
{
	int n, i;
	char *toks[MAXTOK];

	print("%s\n", str);

	n = tokenize(str, (char**)toks, NTOKS);

	for(i = 0; i < n; i++)
		print("%s\n", toks[i]);

	exits(nil);
}
```

### Limbo

[tok.b](./tok.b)

```
implement Tokenizing;

include "sys.m";
	sys: Sys;

include "draw.m";

Tokenizing: module {
	init: fn(nil: ref Draw->Context, nil: list of string);
};

str: con "abc ☺ 'test 1 2 3' !";

init(nil: ref Draw->Context, nil: list of string) {
	sys = load Sys Sys->PATH;

	sys->print("%s\n", str);

	(n, toks) := sys->tokenize(str, "\n\t ");

	for(; toks != nil; toks = tl toks) {
		sys->print("%s\n", hd toks);
	}

	exit;
}
```

### Go

[tok.go](./tok.go)

```
package main

import (
	"fmt"
	"strings"
)

const str = "abc ☺ 'test 1 2 3' !"

func main() {
	fmt.Println(str)

	fields := strings.Fields(str)

	for _, f := range fields {
		fmt.Println(f)
	}
}
```

## Asynchronous spawning

### Newsqueak

[co.nq](./co.nq)

```
double := prog(n : int) {
	print(2*n, "\n");
};

# Begin main logic
begin double(7);
begin double(9);
begin double(11);
```

### Alef

[co.l](./co.l)

```
#include <alef.h>

void
double(int n)
{
	print("%d\n", 2*n);
}

void
main(void)
{
	task double(7);		/* A coroutine	*/
	proc double(9);		/* A process	*/
	par {
		double(11);		/* A process	*/
		double(13);		/* A process	*/
	}
	sleep(5);
}

```

### Plan9 C

[co.c](./co.c)

```
#include <u.h>
#include <libc.h>
#include <thread.h>

void
doubleN(void *n)
{
	print("%d\n", 2*(*(int*)n));
}

void
threadmain(int, char*[])
{
	int s₀ = 7, s₁ = 9;
	threadcreate(doubleN, &s₁, 4096);	// A thread
	proccreate(doubleN, &s₀, 4096);		// A process
	sleep(5);

	threadexitsall(nil);
}
```

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

### Newsqueak

Nope.

### Alef



### Plan9 C



### Limbo



### Go



## Sending and receiving on channels

### Newsqueak

[chans.nq](./chans.nq)

```
max := 10;

# Prints out numbers as they're received
printer := prog(c: chan of int)
{
	i : int;
	for(i = 0; i < max; i++){
		n := <- c;
		print(n, " ");
	}
	print("\n");
};

# Pushes values into the channel
pusher := prog(c: chan of int)
{
	i : int;
	for(i = 0; i < max; i++){
		c <-= i * i;
	}
};

# Begin main logic
printChan := mk(chan of int);

begin printer(printChan);
begin pusher(printChan);
```

### Alef



### Plan9 C



### Limbo



### Go



## Selecting on multiple channels

### Newsqueak

[select.nq](./select.nq)

```
max := 2;

# Selects on two channels for both being able to receive and send
selector := prog(prodChan : chan of int, recChan : chan of int, n : int){
	i, j : int;

	for(;;)
		select{
		case i =<- prodChan:
			print("case recv	← ", i, "\n");

		case recChan <-= n:
			print("case send	→ ", n, "\n");
		}
};

# Pushes `max` values into `prodChan`
producer := prog(n : int, prodChan : chan of int){
	i : int;

	for(i = 0; i < max; i++){
		print("pushed		→ ", n, "\n");
		prodChan <-= n;
	}
};

# Reads `max` values out of `recChan`
receiver := prog(recChan : chan of int){
	i : int;

	# Stop receiving, manually
	for(i = 0; i < max; i++)
		print("received	→ ", <- recChan, "\n");
};

# Begin main logic
prodChan := mk(chan of int);

recChan := mk(chan of int);

begin producer(123, prodChan);
begin receiver(recChan);
begin selector(prodChan, recChan, 456);
```

### Alef



### Plan9 C

[select.c](./select.c)

```
#include <u.h>
#include <libc.h>
#include <thread.h>

const int max = 2;

typedef struct Tuple Tuple;
struct Tuple {
	Channel *a;
	Channel *b;
};

void
selector(void *v)
{
	Tuple *t = (Tuple*)v;
	Channel *prodchan = t->a;
	Channel *recchan = t->b;

	// Set up vars for alt
	int pn;
	int *rn = malloc(1 * sizeof (int));
	*rn = 456;

	// Set up alt
	Alt alts[] = {
		{prodchan,	&pn,	CHANRCV},
		{recchan,	rn,		CHANSND},
		{nil,		nil,	CHANEND},
	};

	for(;;)
		switch(alt(alts)){
		case 0:
			// prodchan open for reading
			recv(prodchan, &pn);
			print("case recv	← %d\n", pn);
			break;

		case 1:
			// recchan open for writing
			send(recchan, rn);
			print("case send	→ %d\n", *rn);
			break;

		default:
			break;
		}
}

void
producer(void *v)
{
	Channel *prodchan = (Channel*)v;
	int *n = malloc(1 * sizeof (int));
	*n = 123;

	int i;
	for(i = 0; i < max; i++){
		print("pushed		→ %d\n", *n);
		send(prodchan, n);
	}

	chanclose(prodchan);
}

void
receiver(void *v)
{
	Channel *recchan = (Channel*)v;

	int i;
	int n;
	for(i = 0; i < max; i++){
		recv(recchan, &n);
		print("received	→ %d\n", n);
	}

	chanclose(recchan);
}

void
threadmain(int, char*[])
{
	// Set up channels
	Channel *prodchan	= chancreate(sizeof (int), max);
	Channel *recchan	= chancreate(sizeof (int), max);

	Tuple *chans = malloc(1 * sizeof (Tuple));
	chans->a = prodchan;
	chans->b = recchan;

	// Start processes
	proccreate(producer, prodchan,	4096);
	proccreate(receiver, recchan,	4096);
	proccreate(selector, chans,		4096);

	sleep(1000);

	threadexitsall(nil);
}
```

### Limbo

show `alt{}`

### Go

show `select{}`

## Multiple returns (??)

### Alef

??

### Plan9 C

Nope.

### Limbo

show using tuples

### Go

show normal multiple returns from function signature

## Lists

### Alef

show `for(each X in L){}` format

### Plan9 C



### Limbo

show list :: operation and iteration style

### Go

show `for p, v := range X`
