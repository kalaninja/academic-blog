+++
tags = ["golang", "git"]
categories = ["Git"]
title = "How to copy tags from one git repository to another"
date = 2017-02-22T17:17:55+03:00
draft = false
+++

Not long ago my team moved a legacy version of our application from a different branch out into a new repository. The problem was that they had forgotten to copy tags there, so we lived without tags for quite a time, until I decided to deal with the problem and made a utility that copies matching tags from one git repository to another.
<!--more-->

Actually, I found a solution on [github](https://github.com/foghina/git-copy-tags) which was a shell script written in ruby. But there were two issues. First, I neither had nor wanted to install ruby on my machine. Secondly, I use Windows on my office laptop. So, I decided to port the original script to Go and here is the code

```go
package main

import (
	"fmt"
	"log"
	"os"
	"os/exec"
	"runtime"
	"strings"
)

func usage() {
	fmt.Println("Usage: git-copy-tags <source-repo> <dest-repo> [-f]")
	fmt.Println("By default, the script is in \"dry run\" mode, which means that it only prints out what it would do, without actually doing it. If you are happy with the result, add -f.")
	os.Exit(1)
}

func shell(cmd string) ([]byte, error) {
	sh := "sh"
	c := "-c"
	if runtime.GOOS == "windows" {
		sh = "cmd"
		c = "/c"
	}

	return exec.Command(sh, c, cmd).CombinedOutput()
}

func exe(cmd string) string {
	result, err := shell(cmd)
	if err != nil {
		log.Fatal(err)
		os.Exit(1)
	}

	return string(result)
}

func getTags() map[string]string {
	tags := exe("git tag")
	dict := make(map[string]string)

	for _, tag := range strings.Split(tags, "\n") {
		tag = strings.TrimSpace(tag)
		if len(tag) < 1 {
			continue
		}
		cmd := fmt.Sprintf("git rev-list --max-count=1 %s", tag)
		commit := strings.TrimSpace(exe(cmd))
		dict[tag] = commit
	}

	return dict
}

func main() {
	if len(os.Args) < 3 {
		usage()
	}

	src := os.Args[1]
	dest := os.Args[2]
	force := false
	if len(os.Args) > 3 && os.Args[3] == "-f" {
		force = true
	}

	os.Chdir(src)
	srcTags := getTags()

	os.Chdir(dest)
	destTags := getTags()

	if !force {
		fmt.Println("Running dry, use -f to actually apply changes...")
	}

	for tag, commit := range srcTags {
		if _, ok := destTags[tag]; !ok {
			cmd := fmt.Sprintf("git rev-list --max-count=1 %s", commit)
			if _, err := shell(cmd); err == nil {
				if force {
					if _, err := shell(fmt.Sprintf("git tag %s %s\n", tag, commit)); err == nil {
						fmt.Printf("Tagged %s with %s", commit, tag)
					} else {
						fmt.Printf("Error while tagging %s with %s\n", commit, tag)
					}

				} else {
					fmt.Printf("Would tag %s with %s\n", commit, tag)
				}
			}
		}
	}
}
```

## Usage

```
git-copy-tags <source-repo> <dest-repo> \[-f\]
```

By default, the script is in "dry run" mode, which means that it only prints out what it would do, without actually doing it. If you are happy with the result, add -f.

After running the command with -f, make sure to run `git push --tags`  in the destination repository.

## Download

The source code is available in the github [repository](https://github.com/kalaninja/git-copy-tags). And here is a cross-compiled version for different operating systems (all 64-bit)

*   [Windows](https://github.com/kalaninja/git-copy-tags/releases/download/release/git-copy-tags.exe)
*   [MacOSX](https://github.com/kalaninja/git-copy-tags/releases/download/release/darwin.zip)
*   [Linux](https://github.com/kalaninja/git-copy-tags/releases/download/release/linux.zip)

## Bonus. Cross compilation in Go

Since Go 1.5 cross compilation is very simple. Just properly set a pair of `$GOOS` and `$GOARCH` variables and run a build process as usual. For example, on Windows it can be done in the following way:

```sh
set GOARCH=amd64
set GOOS=linux
go build
```

The actual list is defined in [src/go/build/syslist.go](https://github.com/golang/go/blob/master/src/go/build/syslist.go) and the valid combinations of `$GOOS` and `$GOARCH` are:

<table cellpadding="0">
<tbody>
<tr>
<th align="left" width="100">$GOOS</th>
<th align="left" width="100">$GOARCH</th>
</tr>
<tr>
<td>android</td>
<td>arm</td>
</tr>
<tr>
<td>darwin</td>
<td>386</td>
</tr>
<tr>
<td>darwin</td>
<td>amd64</td>
</tr>
<tr>
<td>darwin</td>
<td>arm</td>
</tr>
<tr>
<td>darwin</td>
<td>arm64</td>
</tr>
<tr>
<td>dragonfly</td>
<td>amd64</td>
</tr>
<tr>
<td>freebsd</td>
<td>386</td>
</tr>
<tr>
<td>freebsd</td>
<td>amd64</td>
</tr>
<tr>
<td>freebsd</td>
<td>arm</td>
</tr>
<tr>
<td>linux</td>
<td>386</td>
</tr>
<tr>
<td>linux</td>
<td>amd64</td>
</tr>
<tr>
<td>linux</td>
<td>arm</td>
</tr>
<tr>
<td>linux</td>
<td>arm64</td>
</tr>
<tr>
<td>linux</td>
<td>ppc64</td>
</tr>
<tr>
<td>linux</td>
<td>ppc64le</td>
</tr>
<tr>
<td>linux</td>
<td>mips</td>
</tr>
<tr>
<td>linux</td>
<td>mipsle</td>
</tr>
<tr>
<td>linux</td>
<td>mips64</td>
</tr>
<tr>
<td>linux</td>
<td>mips64le</td>
</tr>
<tr>
<td>netbsd</td>
<td>386</td>
</tr>
<tr>
<td>netbsd</td>
<td>amd64</td>
</tr>
<tr>
<td>netbsd</td>
<td>arm</td>
</tr>
<tr>
<td>openbsd</td>
<td>386</td>
</tr>
<tr>
<td>openbsd</td>
<td>amd64</td>
</tr>
<tr>
<td>openbsd</td>
<td>arm</td>
</tr>
<tr>
<td>plan9</td>
<td>386</td>
</tr>
<tr>
<td>plan9</td>
<td>amd64</td>
</tr>
<tr>
<td>solaris</td>
<td>amd64</td>
</tr>
<tr>
<td>windows</td>
<td>386</td>
</tr>
<tr>
<td>windows</td>
<td>amd64</td>
</tr>
</tbody>
</table>