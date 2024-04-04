---
layout: post
title: Short Tale of Intel -> Apple Silicon Migration
excerpt: A few notes about my experience getting my hands on an Apple Silicon Mac and migrating my stuff from my old Intel Mac.
modified: 2024-03-13T00:00:00+01:00
categories: [misc]
tags: [apple, shell-scripting]
comments: true
share: true
---

![image](https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/f3db49bf-8d00-4cf0-bc57-c1e179d6b2a4)


> **Note**: I will be using the terms `amd64`, `x86_64`, and `Intel` interchangeably in this post. They all refer to the same architecture. Similarly `arm64` and `Apple Silicon` are used interchangeably.

I recently got handed a new MacBook at work. It's a MacBook Pro with an M3 chip and my first encounter with Apple Silicon. I have been hearing many good things about Apple Silicon and having experienced it for myself for the past day I must say that my old Intel MacBook can't compare. I know that comparing my 2020 Intel MacBook, which has been used and abused for 4 years, to a brand new, out-of-the-box 2023 M3 Mac is not an [Apples to Apples comparision](https://media.tenor.com/a2gst_5S-RAAAAAi/uarrr-finger-guns.gif). So I will take the word of tech reviewers through the years as well as my own biased experience (I no longer have to wait several seconds for a compilation step) and say that Apple Silicon is a game changer.

The point of this post is not to talk up Apple's chips but to talk about the challenges I faced getting up and running after moving my stuff from an Intel Mac to an Arm Mac.

## Migration Assistant

While not specific to the Intel -> Arm transfer, I was reminded once again that I should probably be running Migration Assistant over a cable instead of WiFi. It took all night and the way the numbers kept going up instead of down, sometimes, gave me a ton of anxiety. The next time I do this I have to remind myself that taking 5 extra minutes to get that cable out and connected will save me _hours_ later.

## It Does Not Come with Rosetta

I'm not entirely sure if this is always the case or if it ended up being the case because I performed a Migration, but [Rosetta](https://support.apple.com/en-us/102527) was not already installed on my system. Instead, any time I tried to run a process that was Intel based, I got a pop-up that asked me if I want to install Rosetta.

![image](https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/2e55563b-7f61-4ad9-aeb9-1ff42469ff98)

I put it off for as long as I could. I wanted to have everything running as natively Apple Silicon as I could possibly manage. I finally did cave when I discovered a few tools did not have an Arm counterpart, but I think I managed to reinstall a lot of things before I hit that stage.

## You Can See What's Running on Rosetta

If you open the [Activity Monitor](https://support.apple.com/guide/activity-monitor/welcome/mac), you can [check which apps are running on Rosetta](https://thenextweb.com/news/how-to-check-app-running-m1-native-version-on-mac). In the CPU tab, look at the Kind column. It will either say _Intel_ or _Apple_.

I managed to discover several things running on Rosetta and then went and found Apple Silicon versions of them. For example, I have some bash scripts that open tunnels to my company's internal network. For managing these multiple tunnels the script uses [tmux](https://github.com/tmux/tmux/wiki). I discovered in Activity Monitor that I had several `bash` processes running as Kind `Intel`. I suspected `tmux` spawned these processes, so I did a quick check:

```sh
➜  which tmux
/usr/local/bin/tmux
➜  file /usr/local/bin/tmux
/usr/local/bin/tmux: Mach-O 64-bit executable x86_64
```

It looks like `tmux` is still an x86 executable.

```sh
➜  which bash
/bin/bash
➜  file /bin/bash
/bin/bash: Mach-O universal binary with 2 architectures: [x86_64:Mach-O 64-bit executable x86_64] [arm64e:Mach-O 64-bit executable arm64e]
/bin/bash (for architecture x86_64):	Mach-O 64-bit executable x86_64
/bin/bash (for architecture arm64e):	Mach-O 64-bit executable arm64e
```

`bash` on the other hand, is a [universal binary](https://en.wikipedia.org/wiki/Universal_binary), an app format that encapsulates multiple architectures and acts as one of the bridges between Intel and Arm Macs.

But since these bash processes were spawned by an _x86_64_ `tmux`, it was the _x86_64_ version that got spawned. Once I replaced `tmux` with the arm version, the bash processes also spawned with their arm counterparts. So far so makes sense!

## A Scriptological Journey

> **Warning**: This section ended up being longer than anticipated and is filled with arcane unix tools and commands. It was also the most fun section to write!

I wanted to find other binaries that were still x86_64 and see if we could clean them up. So I went into the `/usr/local/bin/` directory and ran the following command:
```sh
➜  find -L . -type f -exec file {} + | grep 'x86_64' | grep -v 'arm64'
``` 
Breaking it down:
- `find -L . -type f` searches in the current directory and subdirectory, limited to items that are files. The `-L` option follows symbolic links as well.
- `-exec file {} +` executes the [file](https://en.wikipedia.org/wiki/File_(command)) command on each file.
- `grep 'x86_64'` finds us instances of `x86_64` appearing in the output (the output for x86_64 binaries is `Mach-O 64-bit executable x86_64`).
- `grep -v 'arm64'` excludes instances where there is a universal binary (like `bash`, as mentioned before), which has both x86_64 and arm variants available in one package.

That last grep is to explicitly ensure that if I have a universal binary with both x86_64 and arm64 architectures, we keep that universal binary. 

> **Side Note**: The `+` modifier is an interesting one. It instructs `find` to accumulate the found files and execute the command passed to exec (in this case `file`) command on them in a single invocation. So while without `+` each of the found files would have `file` executed against them one by one, with `+` the files are passed to the command in one go.

The result of running this command:

```shell
./vagrant-go (for architecture x86_64):	Mach-O 64-bit executable x86_64
./dummy:                                        Mach-O 64-bit executable x86_64
./taste:                                        Mach-O 64-bit executable x86_64
./python3.10-intel64:                           Mach-O universal binary with 1 architecture: [x86_64:Mach-O 64-bit executable x86_64]
./python3.10-intel64 (for architecture x86_64):	Mach-O 64-bit executable x86_64
```

If you look closely, you'll notice that the command is not quite completely giving us the result we want. For `./python3.10-intel64`  (which is a universal binary with only 1 architecture) we are getting two lines of output. And for `./vagrant-go` (which we can deduce to be a universal binary with both architectures), we are getting the x86_64 line of output when we actually want none at all. Recall that the ultimate goal is to delete the Intel-only binaries, for which we want a clean list of files.

`awk` to the rescue!

For universal binaries with two architectures, `file` gives us the following.
```sh
➜ file vagrant-go
vagrant-go: Mach-O universal binary with 2 architectures: [arm64:Mach-O 64-bit executable arm64] [x86_64]
vagrant-go (for architecture arm64):	Mach-O 64-bit executable arm64
vagrant-go (for architecture x86_64):	Mach-O 64-bit executable x86_64
```

The first line tells us that this is a universal binary with 2 architectures and the next two lines give more details on those architectures, one line being for each architecture.

We can't treat this as a line-by-line problem, anymore. We need to process the whole 3-line block. We need to recognize that we're processing a 2-architecture universal binary and then figure out a way to exclude the whole block.

```sh
➜  find -L . -type f -exec file {} + | \
    awk '/Mach-O universal binary with 2 architectures/ && /arm64/ && /x86_64/ { getline; getline; next } { print }' | \
    grep 'x86_64' | \
    grep -v 'arm64'
```

We've added an `awk` invocation after listing our files via `find` (and `file`) and before we start grepping them. Breaking down the command:
- `awk` is our invocation of the [awk](https://www.gnu.org/software/gawk/manual/gawk.html) tool.
- `/Mach-O universal binary with 2 architectures/ && /arm64/ && /x86_64/` is the pattern that matches the first line output by `file`.
- `{ getline; getline; next }` is the action block that executes on the lines matched by our pattern. We call [getline](https://www.baeldung.com/linux/awk-getline) twice to skip over the two architecture lines. `next` stops processing the current line since we don't want to do anything with the architecture lines.
- `{ print }` the default action block to apply to anything that does not match our pattern. It lets them pass through and be printed.

So to put it all together, our `awk` invocation looks for lines that indicate a universal binary with two architectures and skips over the architecture lines, essentially ignoring the whole block.

The result of this invocation is:
```sh
./dummy:                                        Mach-O 64-bit executable x86_64
./taste:                                        Mach-O 64-bit executable x86_64
./python3.10-intel64:                           Mach-O universal binary with 1 architecture: [x86_64:Mach-O 64-bit executable x86_64]
./python3.10-intel64 (for architecture x86_64):	Mach-O 64-bit executable x86_64
```

Now we no longer get `./vagrant-go` as part of our results, but `./python3.10-intel64` still appears twice. This is easily remedied by tacking on another `grep -v "universal binary"` to the end of our commands.

```sh
➜  find -L . -type f -exec file {} + | \
    awk '/Mach-O universal binary with 2 architectures/ && /arm64/ && /x86_64/ { getline; getline; next } { print }' | \
    grep 'x86_64' | \
    grep -v 'arm64' | \
    grep -v 'universal binary'
./dummy:                                        Mach-O 64-bit executable x86_64
./taste:                                        Mach-O 64-bit executable x86_64
./python3.10-intel64 (for architecture x86_64):	Mach-O 64-bit executable x86_64
```

At this point our command chain is getting a bit too long. For the sake of sanity I will put it all in a file called `find86_64.sh`.

> **Side Note**: If you're wondering what `dummy` is, let me show you the trick to creating a dummy [Mach-O x86_64](https://en.wikipedia.org/wiki/Mach-O) executable file for testing purposes.
> ```sh
> ➜  printf '\xcf\xfa\xed\xfe\x07\x00\x00\x01\x03\x00\x00\x00\x02\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00' > dummy
> ```
> This file won't execute, but running `file` on it gives us
> ```sh
> dummy: Mach-O 64-bit executable x86_64
> ```
> Which is good enough for testing.

The next step is to extract _just_ the name of the file. We can leave in the leading `./` since `ls` (which we will run next) will accept that just fine.

`awk` to the rescue again!

```sh
➜ find86_64.sh | awk -F':| ' '{print $1}'
./dummy
./taste
./python3.10-intel64
```

Breaking it down:
- `awk -F':| ` uses either `:` or _space_ as a field separator to split the line into multiple fields.
- `{print $1}` prints the first field, which, in our case, is the file name.

Our updated script now looks like this.
```sh
#!/bin/sh
find -L . -type f -exec file {} + | \
    awk '/Mach-O universal binary with 2 architectures/ && /arm64/ && /x86_64/ { getline; getline; next } { print }' | \
    grep 'x86_64' | \
    grep -v 'arm64' | \
    grep -v 'universal binary' | \
    awk -F':| ' '{print $1}'
```

Pipe that through `xargs ls -l`.

```sh
➜ find86_64.sh | xargs ls -l
-rw-r--r--  1 shak  admin  48  9 Mar 12:47 ./dummy
lrwxr-xr-x  1 shak  admin  81  9 Mar 12:57 ./python3.10-intel64 -> ../../../Library/Frameworks/Python.framework/Versions/3.10/bin/python3.10-intel64
-rw-r--r--  1 shak  admin  48  8 Mar 23:50 ./taste
```

Note that we have a mix of regular files and symlinks (the `l` at the beginning of the permission string indicates a symlink, which we can also see by the arrow pointing from `./python3.10-intel64` to what it is linked to). This is not unexpected. Package managers and installation scripts often write their binaries to a location of choice and make them reachable for execution by symlinking from places like `/usr/local/bin/`.

> **Side Note**: If we were to pipe anything into `ls` by invoking `| ls -l`, it would ignore the piped input and run `ls -l` in the working directory, which is clearly not the behavior we want. This is because `ls` does not accept input piped into it. [xargs](https://man7.org/linux/man-pages/man1/xargs.1.html) converts the piped in input to arguments for `ls` to consume.

If we were to pipe the output of `find86_64.sh` directly into `xargs rm`, we would remove the files stored in `/usr/local/bin`. Which works well for the binaries that are stored in this directory but for the ones that are symlinked, we will only have their symlinks removed and not the underlying files.

After a bit of research I found the [readlink](https://www.geeksforgeeks.org/readlink-command-in-linux-with-examples/) command which can be used to find the target of a symbolic link.

```sh
➜ readlink ./python3.10-intel64
../../../Library/Frameworks/Python.framework/Versions/3.10/bin/python3.10-intel64
➜ echo $?
0
```

On regular files it outputs nothing and exits with a `1`.

```sh
➜ readlink ./dummy
➜ echo $?
1
```

With this knowledge in mind, we can extend our script to read each line and do something different based on whether it's a symlink or a regular file.

```sh
#! /bin/bash
find -L . -type f -exec file {} + | \
    awk '/Mach-O universal binary with 2 architectures/ && /arm64/ && /x86_64/ { getline; getline; next } { print }' | \
    grep 'x86_64' | \
    grep -v 'arm64' | \
    grep -v 'universal binary' | \
    awk -F':| ' '{print $1}' | \
    while read -r file; do
        if [ -L "$file" ]; then 
            target=$(readlink "$file")
            echo "$file is a symlink targeting $target"
        else
            echo "$file is a regular file"
        fi
    done
```

Breaking it down:
- `while read -r file` reads each line, the `-r` prevents any backslashes from being processed as escape sequences, preserving the original content of the line.
- `if [ -L "$file" ]` [checks if the file is a symlink](https://koenwoortman.com/bash-script-check-if-file-is-symlink/)

And the rest is just `echo`s. You can imagine how this could be replaced with a bunch of `rm`s to clean up our unwanted binaries. Disinclined to take responsibility for shenanigans apart from my own, I will leave that final bit of scripting to the reader.

## Easier Than I Imagined

I talked to a colleague who got his Apple Silicon Mac more than three years ago when these things were fresh out of the oven. His tales of trial and tribulations made me grateful that I am making the move now when so many projects have adopted Apple Silicon and have arm versions released. I code primarily in Golang, and it was almost trivial to clean up my old Golang versions and install Apple Silicon compatible ones.

## Wrapping it Up

I started a draft for this post when I began my migration journey. I imagined I would have to go through a lot more trouble than I actually ended up going through. Almost every app had an Apple Silicon version that was a simple download+replace operation and the story held true for nearly everything I needed to install via brew. I was up and running in a matter of hours.

Having said that, the scripting journey was a lot of fun. I had to dust off some old Unix tools and learn a few new ones. I also got to play with some of the more arcane features of these tools. I hope you had as much fun reading about it as I did writing about it.

I thank everyone who have been experimenting with Apple Silicon these last few years. They walked with hobbling steps, so people like me could run.
