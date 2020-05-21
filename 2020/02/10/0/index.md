# Code samples from Go-related languages

This post intends to showcase programming patterns which were in common from Newsqueak, Alef, Plan9 C, Limbo, to Go.

(??) ­ Include different Plan9 edition releases to see the libc evolve?

All of these code snippets should be complete as shown and compilable/runnable in the state presented.

## Building and running examples

### Newsqueak

The unix port of squint is probably the most straightforward method, found [here](https://github.com/rwos/newsqueak). The papers describing Newsqueak are [here](https://swtch.com/~rsc/thread/newsqueak.pdf) and [here](https://swtch.com/~rsc/thread/newsquimpl.pdf).

To run a program from a prompt:

	$ squint foo.nq
	# output, if any
	$

### Alef

Your best bet is probably installing a Plan9 2nd edition VM. A text guide for this process is in [a prior blog post](https://seh.dev/2018/03/19/0/) and a video guide to installation is [here](https://www.youtube.com/watch?v=W00TnQ91nj8).

There's a work-in-progress port of Alef from 2e to 9front/386 which can be found on the [public grid](http://wiki.9gridchan.org/public_grid/index.html) griddisk at `/burnzez/rep/alef/root` and maybe `/burnzez/alef`. Griddisk is accessible over 9p via `tcp!45.63.75.148!9564`. You can more easily access the grid from unix via the [gridnix scripts](https://github.com/henesy/grid-unix).

Papers on Alef are [here](http://doc.cat-v.org/plan_9/2nd_edition/papers/alef/).

There are more resources on Plan9 2nd edition indexed [here](http://9.postnix.pw/hist/2e/).

From a prompt on a complete Plan9 2e installation:

	term% 8al foo.l
	term% 8l co.8
	term% 8.out
	# output, if any
	term%

### Plan9 C

The most maintained Plan9 fork, 9front, is found [here](http://9front.org/).

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

Either the official Inferno [here](https://bitbucket.org/inferno-os/inferno-os/) or the purgatorio fork [here](https://code.9front.org/hg/purgatorio).

From a prompt inside the Inferno VM:

	; limbo foo.b
	; foo
	# output, if any
	;

### Go

Go can be acquired from https://golang.org

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

#### Output

```
abc » 'test 1 2 3' !
abc
»
'test
1
2
3'
!
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

#### Output

```
abc ☺ 'test 1 2 3' !
abc
☺
test 1 2 3
!
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

#### Output

```
abc ☺ 'test 1 2 3' !
abc
☺
'test
1
2
3'
!
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

#### Output

```
abc ☺ 'test 1 2 3' !
abc
☺
'test
1
2
3'
!
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

#### Output

```
14
18
22
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

#### Output

```
18
26
22
14
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

#### Output

```
14
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

#### Output

```
14
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

#### Output

```
0 1 4 9 16 25 36 49 64 81
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
	i : int;

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

#### Output

```
pushed		→ 123
pushed		→ 123
case recv	← 123
case send	→ 456
received	→ 456
case send	→ 456
received	→ 456
case recv	← 123
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

#### Output

```
pushed		→ 123
received	→ 456
case send	→ 456
pushed		→ 123
received	→ 456
case send	→ 456
case recv	← 123
case send	→ 456
case recv	← 123
```

### Limbo

[select.b](./select.b)

```
implement Select;

include "sys.m";
	sys: Sys;
	print: import sys;

include "draw.m";

Select: module {
	init: fn(nil: ref Draw->Context, nil: list of string);
};

max : con 2;

selector(prodChan: chan of int, recChan: chan of int, n: int) {
	for(;;)
		alt {
		i := <- prodChan =>
			print("case recv	← %d\n", i);

		recChan <-= n =>
			print("case send	→ %d\n", n);

		* =>
			break;
		}
}

producer(n: int, prodChan: chan of int) {
	for(i := 0; i < max; i++){
		print("pushed		→ %d\n", n);
		prodChan <-= n;
	}
}

receiver(recChan: chan of int) {
	for(i := 0; i < max; i++)
		print("received	→ %d\n", <- recChan);
}

init(nil: ref Draw->Context, nil: list of string) {
	sys = load Sys Sys->PATH;

	prodChan	:= chan of int;
	recChan	:= chan of int;

	spawn producer(123, prodChan);
	spawn receiver(recChan);
	spawn selector(prodChan, recChan, 456);

	sys->sleep(1000);

	exit;
}

```

#### Output

```
pushed		→ 123
case send	→ 456
received	→ 456
case recv	← 123
pushed		→ 123
case recv	← 123
case send	→ 456
received	→ 456
```

### Go

show `select{}`

## Multiple returns

### Newsqueak

show using tuples

### Alef

??

### Plan9 C

Nope.

### Limbo

[multret.b](./multret.b)

```
implement MultRet;

include "sys.m";
	sys: Sys;

include "draw.m";

MultRet: module {
	init: fn(nil: ref Draw->Context, nil: list of string);
};

swap(a, b: int): (int, int) {
	return (b, a);
}

init(nil: ref Draw->Context, nil: list of string) {
	sys = load Sys Sys->PATH;

	(x, y) := swap(3, 7);

	sys->print("3, 7 → %d, %d\n", x, y);

	exit;
}
```

#### Output

```
3, 7 → 7, 3
```

### Go

show normal multiple returns from function signature

## Lists

### Newsqueak

???

### Alef

show `for(each X in L){}` format

### Plan9 C



### Limbo

show list :: operation and iteration style

### Go

show `for p, v := range X`
