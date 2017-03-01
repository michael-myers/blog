+++
title = "A Look at the Rust Programming Language"
Description = "A Safer Compiled Language"
Categories = ["programming", "rust", "security"]
Tags = ["programming", "rust", "security"]
date = "2017-02-27T13:06:16-05:00"
+++

## Where to Find More Execution Performance

Moore's Law [is just about done](http://fortune.com/2017/01/05/intel-ces-2017-moore-law/). It once described a trend of transistor count doubling every 24 months (enabled by increasing the density of transistors by making them ever-smaller). Now:

> Between the introduction of 65 nm and 45 nm chips, about 23 months passed. To get from 45 nm to 32 nm took about 27 months, 28 months to go down from there to 22 nm and 30 months to shrink to the current 14 nm process. And that's where Intel has been stuck since September 2014.

Intel *might* release 10nm scale chips in late 2017, which would mean that they worked 36-40 months in order to shrink from 14nm to 10nm scale. In other words, the most recent density doubling (the shrink from 22nm to 10nm), by the time it happens, will have taken over 5 years. The next doubling is likely to take at least that long, assuming the multiple breakthroughs required to do so can even be achieved. 10nm is already fairly close to the atomic scale: ~45 silicon *atoms* across (one atom: 0.22nm). One of the obstacles at this scale to be addressed is [quantum tunneling](https://en.wikipedia.org/wiki/Quantum_tunnelling), not that I pretend to understand it.

Of course, Moore's Law can be satisfied one other way without changing density, which is to simply use bigger and bigger processor dies. You may have seen charts showing that transistor *count* continues to increase on schedule with Moore's Law, but this is only true for dedicated GPUs and high-end server CPUs, which are already up against cost practicality limits due to these die sizes.

Even if we were still on track for Moore's Law, increasing transistor counts alone have [provided diminishing returns as of late](https://cartesianproduct.wordpress.com/2013/04/15/the-end-of-dennard-scaling/). Recent density increases have mainly just served to reduce power draw and to make more space on the CPU die dedicated to graphics rendering (an ideal parallelizable task). Tech being an optimistic culture makes it slow to acknowledge the obvious truth here: CPU *cores* aren't getting significantly faster. Unless your work is on a mobile device or can be delegated to a GPU or server farm, your only performance upgrades since 2010 have been I/O-related ones.

Granted, transistor density improvements have continued to increase CPU power efficiency. But I have a Intel "Core i7" (2.66 GHz i7-620M, 2-core) laptop that will turn 7 years old in a couple of months, and today's equivalent CPUs *still* offer only a marginal performance improvement for tasks that aren't 3D graphics. The equivalent CPU today, the Intel "Core i7" (2.7GHz i7-7500U, 2-core), has single-threaded performance only about 60% better than my CPU from 7 years ago. Not enough to make me throw out my old laptop.

All of this background is to make my point, which is that the next performance leap has to come from improved software, rather than relying on "free" improvements from new hardware. A few software methods for achieving a generational improvement in performance might be:

- Parallelism
- Optimizing compilers
- Moving tasks from interpreted languages back to compiled languages

All of these things are already happening, but it's the last one that I'm interested in most.

## Parallelism

Parallelism has brought great performance improvements in graphics, "AI," and large data set processing (so-called "Big Data"), and is the reason why GPUs [continue to march forward in transistor count](https://en.wikipedia.org/wiki/Transistor_count#GPUs) (although, again, check out those increasing die sizes; those are approaching their own limits of practicality). The problem with parallelism, though, is that while there are some workloads that are naturally suited to it, others aren't and never will be. Sometimes, computing Task B is dependent on the outcome of Task A, and there is just no way to split up Task A. Even when parts of a task *can* be parallelized, there are swiftly diminishing returns to adding more cores, as described [by Amdahl's Law](https://en.wikipedia.org/wiki/Amdahl%27s_law). What parallelized processing *does* scale well for [is large data sets](https://en.wikipedia.org/wiki/Gustafson%27s_law), although the home user is not typically handling large data sets, and won't directly benefit from this kind of parallelism.

## Optimizing Compilers

Here are [Daniel J Bernstein's 2015 slides](https://cr.yp.to/talks/2015.04.16/slides-djb-20150416-a4.pdf) about the death of "optimizing compilers," or rather, that despite all the hype about them, we are still manually tuning the performance critical portions of our programs. The optimizing compilers' optimization of non-critical code portions is irrelevant, or at least not worth the effort put into optimizing compilers. It appears that a compiler to generically optimize any code as well as an expert human could, would require something like a [general AI](https://en.wikipedia.org/wiki/Artificial_general_intelligence) with a full contextual understanding of the problem being solved by the code. Such a thing doesn't exist, and is not on the horizon.

## Better (Safer) Compiled Languages

C and C++ never really left us, and neither have all of the inherent memory errors in code programmed in C and C++. That includes Java, whose runtime is still written in C. The Java runtime has been the source of many "Java" security issues over the years, to the point where the Java plug-in was effectively banned from all web browsers. Despite that, the rest of the browser is also written in C and C++, and just as prone to these problems. There hasn't been any viable alternative but to try to sandbox and privilege-reduce the browser, because any safer language is too slow.

The real cost of C and C++ 's performance is their high maintenance burdens: coding in them means always opening up subtle concurrency errors, memory corruption bugs, and information leak vulnerabilities. This is why simply improving the C++ standard library and adding more and more features to the language has not altered its basic value proposition to developers, who have already fled to "safe" languages.

That's where the experimental language, Rust, comes in. It's a compiled systems programming language with performance on par with ([or better than](http://benchmarksgame.alioth.debian.org/u64q/which-programs-are-fastest.html)) C++, but with compile-time restrictions on memory management and concurrency that should prevent entire classes of bugs. At some point in the next 5 years, I predict that we will see Rust (or something like it, whether it's [Swift](https://en.wikipedia.org/wiki/Swift_(programming_language)) or some new really strict C++ compiler) slowly start replacing C/C++ wherever performance and security are both primary concerns. It's exciting to think that a well-designed compiled language could solve most of the reasons for the ~20-year flight away from native code programming.

Having played with Rust for a few days, I can say it will certainly not replace Python for *ease* of development, but it's a really interesting disruptor for anyone writing native code. Security researchers should also take notice.

## Rust Programming Language

For what it's worth, Rust was the “Most Loved Programming Language of 2016 in the Stack Overflow Developer Survey.” It enforces memory management and safety at compile-time. Some memory safety features of the language include:

- Rust does not permit null pointers or dangling pointers. Since pointers are never NULL, you can always safely dereference a pointer.
- There are no “void” pointers.
- Pointers can not be downcast to a more specific type, only upcast to a more generic type. If generic data structures are needed, you use parameterized types/functions.
- Variables can be allocated on the heap and are cleaned up without the need for “free” or “delete.”
- There can be only one pointer pointing to an allocation, and it is passed back and forth between “owners” such that concurrent access race conditions are impossible.

If you just wanted a statically typed, compiled language with a modern standard library that is easy to extend, you could also choose Go. But Rust claims to be all of that, plus faster and safer. Rust will work in embedded devices and other spaces currently occupied by C/C++; Go will not. [Some think Rust is just fundamentally better](http://yager.io/programming/go.html), but I am not qualified to judge that.

### Rust and parallelism

Rust makes parallelization an integral part of the language, with support for all of the necessary parallel programming primitives. Parallelized versions of various programming constructs can be swapped in without changing your existing code. This is possible because the Rust language forces the programmer to specify more about how data will be used, which prevents race conditions at runtime by turning them into errors at compile time, instead.

### Concept of "Ownership" in Rust

The major innovation of the Rust language (inspired by a prior language, "Cyclone") is that its compiler, in order to do memory management and prevent race conditions at compile time, tracks "ownership" of all variables in the code. Once a variable is used (like in a call to a function) it is considered to be passed to a new "owner," and using it in a subsequent statement is illegal and would trigger a compiler error. If the developer's intention was to copy-on-use ("clone"), they must specify that in their code. For certain simple data types (integers, etc.), they are automatically copied-on-use without any explicit intent from the developer. Another aspect of ownership in Rust is that all variables are (what in C/C++ would be called) `const`, by default. In Rust, if you want a variable to be mutable, it has to be explicitly stated in the declaration.

This concept is the foundation of the Rust language. It's hard to grasp at first, since it is very different from programming in C or C++, or even Java. The most detailed explanation of Rust ownership that I've seen is [this article by Chris Morgan](https://chrismorgan.info/blog/rust-ownership-the-hard-way.html), but to actually learn the concept I'd recommend starting with [this 25 minute video by Nikolas Matsakis](http://intorust.com/tutorial/ownership/). 

At first, it seems like another mental burden on the programmer, but adopting this concept of memory management means the programmer is also *relieved* of having to manage memory with carefully paired calls to `malloc()` and `free()` (or `new` and `delete`). "So what, isn't this what you get with C# or Java?" Not quite: those languages use a Garbage Collector to track references to data at *runtime*, which has an inherent performance overhead and whose "stop-the-world" resource management can be [inconsistent and unpredictable](http://stackoverflow.com/questions/16695874/why-does-the-jvm-full-gc-need-to-stop-the-world). Rust does it in the language, at compile time. *So, without the use of a Garbage Collector, Rust makes memory management (and concurrent access to data) safe again.*

### Rust is a Drop-In Replacement for C

Just like C/C++, Rust can [be coupled to Python or any other language with a native interface, in order to leverage the strengths of both](https://blog.sentry.io/2016/10/19/fixing-python-performance-with-rust). And, debugging Rust programs is officially [supported by GDB](https://www.mail-archive.com/info-gnu@gnu.org/msg02192.html). This works the other way around too, i.e., you can build a Rust program on top of native code libraries written in C/C++. Mozilla is even working on [a web browser engine in Rust](https://servo.org), to replace Gecko, the Firefox engine. [Benchmarks in 2014](https://www.phoronix.com/scan.php?page=news_item&px=MTgzNDA) showed a 300% increase in performance vs Gecko, and by early 2016, it was [beating Webkit and Chrome as well](https://www.phoronix.com/scan.php?page=news_item&px=Google-Servo-Perf-Comparison) (at least in some hand-picked benchmarks where they leverage Rust's ease of parallelism to delegate a bunch of stuff to the GPU). If you're interested in the details of how Rust can improve browser engines, Mozilla [wrote about it here](https://arxiv.org/pdf/1505.07383v1.pdf). Buried in the paper is a detail that they seem to have downplayed elsewhere, though: the new browser engine is actually still bootstrapped by an existing codebase, so it's still 75% C/C++ code. On the other hand, that also goes to show how Rust integrates well with C/C++.

### Rust has a Package Manager, which is also its Build Tool

Makefiles are impossible to write and debug, and basically you're always just copy-pasting a previous Makefile into the new one, or hoping an IDE or build tool abstracts away all that crap for you, which is why [this wheel has been reinvented many times](https://en.wikipedia.org/wiki/List_of_build_automation_software). I generally don't have a favorite build tool (they're all bad), since it always seems to come down to a manual troubleshooting cycle of acquiring all the right dependencies. The worst is having a build system that is a big layer cake of scripts on top of XML on top of Makefiles.

Rust package manager "Cargo" simply uses TOML files to describe what a Rust project needs in order to build, and when you build with Cargo, it just goes out and gets those dependencies for you. Plus, the packages are served from Crates.io, so if you're keeping score that's a double tech hipster bonus for using both the .io domain *and* TOML.

## Installation and Hello World

Assuming you're using MacOS like me (there is plenty of info out there already for Windows and Linux users) and you have Homebrew:

```sh
    $ brew install rust
    $ rustc --version
    rustc 1.15.0
```

You probably want an editor with Rust syntax highlighting and code completion. [These are your choices](https://areweideyet.com). I went with Visual Studio Code, aka VS Code. It's not what I'd call an IDE, and I still haven't gotten it to integrate with a debugger, but hopefully JetBrains will step up and make a Rust IDE – once there is a market for it.

VS Code doesn't understand Rust out of the box. Launching VS Code, hit Command-P to open the in-app console:

    ext install vscode-rust
    (install the top search result, should be the extension by kalitaalexey)

Optionally, you can install a GDB/LLDB integration layer to attempt to debug from VS Code (in theory – YMMV but I haven't gotten it to work for LLDB with C++ yet, let alone Rust):

    ext install webfreak.debug
    (install the top search result)
Notice in the bottom right: “Rust tools are missing” … click install. It will invoke Cargo (the Rust package manager) to download, compile, and install more of the Rust toolchain for you: racer, rustfmt, rustsym, etc. And all of the dependencies for those. Go have a coffee, this will take a while. About 18 minutes on my system.

Finally: close VS Code, and open up Terminal so we can put all these new Rust binaries on your $PATH.

    $ open -a /Applications/TextEdit.app ~/.bash_profile

Add the line `export PATH="/Users/yourusername/.cargo/bin:$PATH"` and save.

Open a new instance of VS Code. It should no longer tell you that Rust tools are missing. 👍🏻

Test the environment with a Hello World in Rust! Save the following as hello.rs:

```rust
fn main() {
    println!("Hello World!");
}
```
Open "View -> Integrated Terminal." From here you can compile by hand like a peasant, because VS Code isn’t an actual IDE.

```sh
bash-3.2$ cd ~/Desktop
bash-3.2$ rustc hello.rs
bash-3.2$ ./hello
Hello World!
```

But for a realistic scenario, we could have also used Cargo to both create a new Rust project and then build it.

In a future post, I will share my thoughts on what it's like to try to actually write a program in Rust.

## Rust References

- [Rust by Example](https://rustbyexample.com/index.html)
- [The Rust Programming Language](https://doc.rust-lang.org/book/README.html) (free e-book)