---
title: "Build from source"
linkTitle: "Build from source"
description: "How to build rqlite from source"
weight: 40
---
rqlite can be compiled for a wide variety of operating systems and platforms.

## Building rqlite
*Building rqlite requires Go 1.21 or later. [gvm](https://github.com/moovweb/gvm) is a great tool for installing and managing your versions of Go.*

One goal of rqlite is to keep the build process as simple as possible, to aid development and debugging. Download, build, and run rqlite like so (tested on 64-bit Ubuntu 20.04, macOS, and Windows):

```bash
mkdir rqlite # Or any directory of your choice.
cd rqlite/
export GOPATH=$PWD
mkdir -p src/github.com/rqlite
cd src/github.com/rqlite
git clone https://github.com/rqlite/rqlite.git
cd rqlite
go install ./...
$GOPATH/bin/rqlited ~/node.1
```
This starts a rqlite server listening on localhost, port 4001. This single node automatically becomes the Leader.

To rebuild and run, perhaps after making some changes to the source, do something like the following:
```bash
cd $GOPATH/src/github.com/rqlite/rqlite
go install ./...
$GOPATH/bin/rqlited ~/node.1
```

### Linking behavior
Note that the above commands build a version of `rqlited` that dynamically links _libc_. When officially released, `rqlited` for certain CPU architectures will statically link all code, including libc. The SQLite source code is **always** part of rqlite however, and you never need any SQLite library (often named `libsqlite3`) on your host machine.

### Raspberry Pi
The process outlined above will work for Linux, OSX, and Windows. For Raspberry Pi, check out [this GitHub issue](https://github.com/rqlite/rqlite/issues/340).

### Protobuf code generation
_This step is not necessary unless you are making changes to protobuf definitions._

Ensure you have the required tools installed, and that `GOPATH` is set.
```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOBIN
(cd $GOPATH/src/github.com/rqlite/rqlite/command/proto && protoc --go_out=. --go_opt=paths=source_relative command.proto)
(cd $GOPATH/src/github.com/rqlite/rqlite/cluster/proto && protoc -I $GOPATH/src/github.com/rqlite/rqlite/ --proto_path $GOPATH/src/github.com/rqlite/rqlite/cluster/proto --go_out=. --go_opt=paths=source_relative message.proto)
```

### Speeding up the build process
It can be rather slow to rebuild rqlite, due to the repeated compilation of the SQLite source code. You can compile and install the SQLite libary once, so subsequent builds are much faster. To do so, execute the following commands:
```bash
cd $GOPATH/src/github.com/rqlite/rqlite
go install github.com/rqlite/go-sqlite3
```

## Cloning a fork
If you wish to work with fork of rqlite, your own fork for example, you must still follow the directory structure above. But instead of cloning the main repo, instead clone your fork. You must fork the project if you want to contribute upstream.

Follow the steps below to work with a fork:

```bash
export GOPATH=$HOME/rqlite
mkdir -p $GOPATH/src/github.com/rqlite
cd $GOPATH/src/github.com/rqlite
git clone git@github.com:<your Github username>/rqlite
```

Retaining the directory structure `$GOPATH/src/github.com/rqlite` is necessary so that Go imports work correctly.

## Testing
Be sure to run the unit test suite before opening a pull request. An example test run is shown below.
```bash
$ cd $GOPATH/src/github.com/rqlite/rqlite
$ go test ./...
?       github.com/rqlite/rqlite       [no test files]
ok      github.com/rqlite/rqlite/auth  0.001s
?       github.com/rqlite/rqlite/cmd/rqlite    [no test files]
?       github.com/rqlite/rqlite/cmd/rqlited   [no test files]
ok      github.com/rqlite/rqlite/db    0.769s
ok      github.com/rqlite/rqlite/http  0.006s
ok      github.com/rqlite/rqlite/store 6.117s
ok      github.com/rqlite/rqlite/system_test   7.853s
```

## Development philosophy
### Clean commit histories
If you open a pull request, please ensure the commit history is clean. Squash the commits into logical blocks, perhaps a single commit if that makes sense. What you want to avoid is commits such as "WIP" and "fix test" in the history. This is so we keep history on master clean and straightforward.

### Third-party libraries
Please avoid using libaries other than those available in the standard library, unless necessary. This requirement is relaxed somewhat for software other than rqlite node software itself. To understand why this approach is taken, check out this [post](https://blog.gopheracademy.com/advent-2014/case-against-3pl/).

