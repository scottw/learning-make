# Make

Make[1] is the developer's equivalent of Batman's grappling hook. It always works, and gets you from A to B really fast. It is also used to take down bad guys and is often used for things well outside of its original use cases.

## What is Make For?

Make was written for building C programs originally, but is general purpose enough to abuse for many other uses.

You can download virtually any software system these days and find a Makefile. Some software systems will generate the Makefile and then rely on Make's strengths to do the heavy lifting:

	$ perl Makefile.PL
	$ make && make test && make install && make distclean

or:

	$ ./configure.sh
	$ make && make test && make install && make clean

## How Does Make Work?

This is a rule:

    target: prerequisite-1 prerequisite-2
    	command-1
    	command-2

The two command lines are called the "recipe". They *must* start with a tab character. If you use spaces, you will not get good results.

Make reads rules like this[2]:

> I want to ensure `target` is up to date. To ensure it is up to date, I will look at the timestamps on the prerequisites. If the timestamps on the prerequisites are newer than the timestamp on `target`, or if there are no prerequisites, or if `target` does not exist—then I will execute the recipe.

Each logical line in the recipe is processed by Make, which has its own substitutions and so forth, after which it is passed to a shell. Each line gets its own shell. This is important to remember if you're expecting state changes before running a command. Behold:

    where-am-i:
    	-mkdir zzz && cd zzz && pwd
    	pwd
    	rm -r zzz

Which illustrates that each command runs by itself:

    $ make where-am-i
    mkdir zzz && cd zzz && pwd
    /Users/scott/zzz
    pwd
    /Users/scott
    rm -r zzz

Here's another illustration:

    shell:
    	echo $$$$;
    	echo $$$$;

This one is interesting. In Make, the dollar sign indicates a Make variable, but that can be escaped with another dollar sign before it's passed to the shell.

This runs:

    $ make shell
    echo $$;
    63352
    echo $$;
    63353

What do you do to keep things in one shell? You have two options. `.ONESHELL` or newline continuations:

	say-something:
		@echo Here is a thing and \
			another thing in the same shell.

## How Can I Abuse Make to Suit My Own Needs?

Make, by default, lives in the world of c, with object files and binary targets:

    bar.o: bar.c bar.h
    	gcc -c bar.c

    baz.o: baz.c baz.h
    	gcc -c baz.c

    foo: bar.o baz.o
    	gcc -o foo bar.o baz.o -lcrypto

Let's read this in English:

> `bar.o` should be rebuilt if `bar.o` doesn't exist, or if `bar.o` exists and is older than its dependencies `bar.c` and `bar.h`. We rebuild it by running its recipe `gcc -c bar.c`.

> `baz.o` works in the same way.

> `foo` is our binary executable that depends on `bar.o` and `baz.o`. We build it and link with `libcrypto`.

What if we're not writing c, though? Can Makefiles be useful for other tasks?

Let's start with something simple, say, running some unit tests. It's good software design to start with the end in mind: how do we want to use this? Simple is usually better, so let's just try for:

    $ make test

What would that look like in the Makefile?:

	.PHONY: test
	test:
		prove -lvr t

Pretty easy, but what's this `.PHONY` business? `.PHONY` tells Make not to look for a file called `test`, that this target doesn't actually create any files. Also, it tells Make that if a file or directory `test` existed, we should ignore it. I'm not going to refer to `.PHONY` anymore, but it's still necessary for the kinds of targets we'll be using.

What if we only want to run the tests if source has changed?

(see 03-depends.mk)

Let's say that we want to run tests against the alpha database or a beta database. We could make two rules for this:

	test-alpha:
		DB=alpha prove -lvr t
	
	test-beta:
		DB=beta prove -lvr t

This is good, but we see some redundancy. We can fix this by introducing wildcards:

	%-alpha: DB=alpha
	%-beta:  DB=beta
	
	test-%:
		DB=$(DB) prove -lvr t

This lets us add other DB hosts by adding one line:

	%-staging: DB=staging

It also lets us add more targets that cover all of the DB hosts by adding another wildcard rule:

	%-alpha: TAG=alpha
	%-beta:  TAG=beta
	%-staging: TAG=staging

    docker-build-%:
    	docker build -t cas:$(TAG) . -e="DB=$(DB)"

The wildcard matches any prefix or postfix or even infix.

## Make Variables

Make has the notion of variables. We were using them above with DB and TAG. You can also set Make variables on Make's command-line:

    $ make test DB=local

Make creates Make variables from any environment variables set at the time Make runs, so this is equivalent to the previous call:

	$ DB=local make test

You don't access environment variables inside Make, you access Make variables, which were initialized from the environment.

You use Make variables in rules by wrapping them with parentheses:

	test: FILE=domains
	test:
		prove -lv t/$(FILE).t

Another interesting thing is that you can set variable defaults this way, but can then be overridden when Make is invoked:

	$ make test FILE=email

## Functions

Make also has the notion of functions. For most work you do with Makefiles, you'll never use Make's functions, but it's good to know they're there for you if you need them.

### Shell

Sometimes in a Makefile, you just need to shell out and grab something. How is this done? The best way is to use Make's `shell` function:

	MODE = $(shell hostname | grep -q beta && echo 'beta' || echo 'production')

Here is a poor man's ternary operator. We run the `hostname` command, which we pipe to `grep` looking for the word `beta`. If we find it, we echo `beta` otherwise we echo `production`. The result of all this stays in the parentheses and is assigned to the Make variable `MODE`.

Maybe we need to pull some value from a JSON file:

	AWS_ACCESS_KEY_ID := $(shell jq -r .Credentials.AccessKeyId file.json)

Here we invoke the excellent `jq` command line utility for parsing JSON files and picking out the raw value of the `AccessKeyId` node, which we assign to `AWS_ACCESS_KEY_ID`.

Note that we assign here using `:=`, rather than `=`. Make has two kinds of assignment: deferred or lazy and immediate. A lazily assign variable is done using `=`; it's called lazy because the right-hand side of the assignment isn't done until the variable is used in a rule. This allows you to change maybe some of the values of the variable until the last minute.

Immediate variable assignment is done with `:=`. The right-hand side of the assignment is done at the time the variable is created and keeps its value through other rules.

The `shell` function is one of only a tiny handful of Make functions that interact with things outside of the Make world.

### Other Functions

* **subst**

		$(subst from,to,text)

* **strip**

		$(strip string)

* **filter**, **filter-out**

		$(filter pattern..., text)

* **sort**

* **word**

* **firstword**, **lastword**

* **dir**

* **suffix**

* **basename**

* **join**

* **realpath**, **abspath**

* **call**

## Final Example

(BHaaS)?

## Notes

I consulted the Wikipedia article on Make, looking for a little bit of historical information and found that it was on the whole a pretty good introduction to Make as well. I recommend the manual, of course.

## When Should I Use Make?

Preface: a seasoned software developer is familiar with many tools, not just those in his or her preferred programming language and ecosystem. All tools are designed with a variety of use cases in mind and optimized for those purpose. Using a screwdriver to drive nails will work, but not as fast as a hammer.

Practically anything that needs to be automated can and probably should be automated. Make excels at simple automation. Make is a declarative wrapper around another language optimized for dependency graphing and automation.

Make fits into the build tools automation continuum with shell commands at one end and tools like Ansible closer to the other end. Make has powerful recursive capabilities but definitely has declarative biases. For example, you can have an `if` statement in Make, but if have more than one or two of them, it may indicate a problem in your thinking of the problem.

Make scales fairly well for some domains, but it gets complex once your build work has a lot of conditionals. Again, conditional are often a code smell and a build environment is no exception. There's nearly always a way to clean up conditionals into declarative primitives.

Once you start hitting multiple Makefiles as part of a build or test or deployment, you might be pushing its limits. I've seen some situations where Make is used for just the build and Ansible is used to tie the many pieces and tasks of moving things around for large systems.

[1]: https://www.gnu.org/software/make/manual/make.html

[2]: https://www.gnu.org/software/make/manual/make.html#How-Make-Works

[3]: https://en.wikipedia.org/wiki/Make_(software)