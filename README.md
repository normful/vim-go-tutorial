Hands-on demonstration video: https://www.youtube.com/watch?v=7BqJ8dzygtU

----

`:GoRun`

`:GoBuild`

quickfix: `:cnext` and `:cprevious`

```vi
map <C-n> :cnext<CR>
map <C-m> :cprevious<CR>
nnoremap <leader>a :cclose<CR>
```

```vim
autocmd FileType go nmap <leader>b  <Plug>(go-build)
autocmd FileType go nmap <leader>r  <Plug>(go-run)
```


## Test

`:GoTest` (`go test`)

`:GoTestFunc` tests function under cursor

`:GoTestCompile` check that test file compiles (does not run tests)

## Coverage

`:GoCoverage` (`go test -coverprofile tempfile`)

`:GoCoverageClear` clears coverage highlighting

`:GoCoverageToggle`

`:GoCoverageBrowser` (`go tool cover` to create a HTML page and opens)

## Imports

`:GoImport strings` (has tab completion support)

`:GoImportAs str strings`

`:GoDrop strings` removes it from the import declarations

`goimports` is a replacement for `gofmt`

```
let g:go_fmt_command = "goimports"
```

Some people do not prefer `goimports` as it might be slow on very large codebases. 
In this case we also have the `:GoImports` command (note the `s` at the end). With this, you can explicitly call `goimports`

### Text objects

`if` inner function
`af` `a function` (includes function declaration and function comment)

If you don't like comments being a part of the function declaration, you can easily disable it with:

```vim
let g:go_textobj_include_function_doc = 0
```

`gS` when cursor on the same line as struct expression will split struct expression into multiple lines
`gJ` in normal mode joins all field definitions

## Snippets

`errp` in insert mode and `tab`:

```
if err != nil {
    panic( )
          ^
          cursor position
}
```

```
fn -> fmt.Println()
ff -> fmt.Printf()
ln -> log.Println()
lf -> log.Printf()
```

Here `ff` and `lf` are special. They dynamically copy the variable name into
the format string as well.

```
type foo struct {
    Message string .
                   ^ put your cursor here 
}
```

In `insert` mode, type `json` and hit tab. You'll see that it'll be
automatically expanded to valid field tag. The field name is converted
automatically to a lowercase and put there for you. You should now see the
following:

```
type foo struct {
	Message  string `json:"message"`
}
```



It's really amazing. But we can do even better! Go ahead and create a
snippet expansion for the `ServerName` field. You'll see that it's converted to
`server_name`. Amazing right?

```go
type foo struct {
	Message    string `json:"message"`
	Ports      []int
	ServerName string `json:"server_name"`
}
```




For example `:GoLint`. Under the hood it calls `golint`, which is a command
that suggests changes to make Go code more idiomatic. There
is also `:GoVet`, which calls `go vet` under the hood. There are many other
tools that check certain things. To make it easier, someone decided to
create a tool that calls all these checkers. This tool is called
`gometalinter`. And vim-go supports it via the command `:GoMetaLinter`. So what
does it do?

If you just call `:GoMetaLinter` for a given Go source code. By default it'll run
`go vet`, `golint` and `errcheck` concurrently. `gometalinter` collects
all the outputs and normalizes it to a common format. Thus if you call
`:GoMetaLinter`, vim-go shows the result of all these checkers inside a
quickfix list. You can then jump easily between the lint, vet and errcheck
results. The setting for this default is as following:

```vim
let g:go_metalinter_enabled = ['vet', 'golint', 'errcheck']
```

There are many other tools and you can easily customize this list yourself. If
you call `:GoMetaLinter` it'll automatically uses the list above.


Because `:GoMetaLinter` is usually fast, vim-go also can call it whenever you
save a file (just like `:GoFmt`). To enable it you need to add the following to
your `.vimrc:`

```vim
let g:go_metalinter_autosave = 1
```

What's great is that the checkers for the autosave is different than what you
would use for `:GoMetaLinter`.  This is great because you can customize it so only
fast checkers are called when you save your file, but others if you call
`:GoMetaLinter`. The following setting let you customize the checkers for the
`autosave` feature.


```vim
let g:go_metalinter_autosave_enabled = ['vet', 'golint']
```

As you see by default `vet` and `golint` are enabled. Lastly, to prevent
`:GoMetaLinter` running for too long, we have a setting to cancel it after a
given timeout. By default it is `5 seconds` but can be changed by the following
setting:

```vim
let g:go_metalinter_deadline = "5s"
```

## Alternate files

`:GoAlternate` `main_test.go` alts between `main.go`

## Go to definition

* Use `ctrl-]` or `gd` to jump to a definition, locally or globally
* Use `ctrl-t` to jump back to the previous location 

`:GoDefStack`
`:GoDefStackClear`

### Move between functions

```
:GoDecls
:GoDeclsDir
```

`ma` you'll see that `ctrlp` filters the list for you. If you hit `enter` it will automatically jump to it.

The fuzzy search capabilities combined with `motion`'s AST capabilities brings
us a very simple to use but powerful feature.

For example, call `:GoDecls` and write `foo`. You'll see that it'll filter
`BarFoo` for you. The Go parser is very fast and works very well with large files
with hundreds of declarations.

Sometimes just searching within the current file is not enough. A Go package can
have multiple files (such as tests). A type declaration can be in one file,
whereas a some functions specific to a certain set of features can be in
another file. This is where `:GoDeclsDir` is useful. It parses the whole
directory for the given file and lists all the declarations from the files in the 
given directory (but not subdirectories).

Call `:GoDeclsDir`. You'll see this time it also included the declarations from
the `main_test.go` file as well. If you type `Bar`, you'll see both the `Bar`
and `TestBar` functions. This is really great if you just want to get an
overview of all type and function declarations, and also jump to them.

Let's continue with a question. What if you just want to move to the next or
previous function? If your current function body is long, you'll probably will
not see the function names. Or maybe there are other declarations between the
current and other functions.

Vim already has motion operators like `w` for words or `b` for backwards words.
But what if we could add motions for Go ast? For example for function declarations?

vim-go provides(overrides) two motion objects to move between functions. These
are:


```
]] -> jump to next function
[[ -> jump to previous function
```

Vim has these shortcuts by default. But those are suited for C source code and
jumps between braces. We can do it better. Just like our previous example,
`motion` is used under the hood for this operation

Open `main.go` and move to the top of the file. In `normal` mode, type `]]` and
see what happens. You'll see that you jumped to the `main()` function. Another
`]]` will jump to `Bar()` If you hit `[[` it'll jump back to the `main()`
function.

`]]` and `[[` also accepts `counts`. For example if you move to the top again
and hit `3]]` you'll see that it'll jump to the third function in the source file.
And going forward, because these are valid motions, you can apply operators to
it as well!

If you move your file to the top  and hit `d]]` you'll see that it deleted
anything before the next function. For example one useful usage would be typing
`v]]` and then hit `]]` again to select the next function, until you've done
with your selection.


### .vimrc improvements

* We can improve it to control how it opens the alternate file. Add the
  following to your `.vimrc`:


```vim
autocmd Filetype go command! -bang A call go#alternate#Switch(<bang>0, 'edit')
autocmd Filetype go command! -bang AV call go#alternate#Switch(<bang>0, 'vsplit')
autocmd Filetype go command! -bang AS call go#alternate#Switch(<bang>0, 'split')
autocmd Filetype go command! -bang AT call go#alternate#Switch(<bang>0, 'tabe')
```

This will add new commands, called `:A`, `:AV`, `:AS` and `:AT`. Here `:A`
works just like `:GoAlternate`, it replaces the current buffer with the
alternate file. `:AV` will open a new vertical split with the alternate file.
`:AS` will open the alternate file in a new split view and `:AT` in a new tab.
These commands are very productive depending on how you use them, so I think
it's useful to have them.

* The "go to definition" command families are very powerful but yet easy to use.
Under the hood it uses by default the tool `guru` (formerly `oracle`). `guru` has
an excellent track record of being very predictable. It works for dot imports,
vendorized imports and many other non-obvious identifiers. But sometimes it's
very slow for certain queries. Previously vim-go was using `godef` which is
very fast on resolving queries. With the latest release one can easily use or
switch the underlying tool for `:GoDef`.  To change it back to `godef` use the
following setting:

```vim
let g:go_def_mode = 'godef'
```

* Currently by default `:GoDecls` and `:GoDeclsDir` show type and function
  declarations. This is customizable with the `g:go_decls_includes` setting. By
  default it's in the form of:

```
let g:go_decls_includes = "func,type"
```

If you just want to show function declarations, change it to:

```
let g:go_decls_includes = "func"
```

# Understand it

Writing/editing/changing code is usually something we can do only if we first
understand what the code is doing. vim-go has several ways to make it easy to
understand what your code is all about. 

### Documentation lookup

Let's start with the basics. Go documentation is very well-written and is
highly integrated into the Go AST as well. If you just write some comments, the
parser can easily parse it and associate with any node in the AST. So what it
means is that we can easily find the documentation in the reverse order. If
you have the node from an AST, you can easily read the documentation (if you
have it)!

We have a command called `:GoDoc` that shows any documentation associated with
the identifier under your cursor. Let us change the content of `main.go` to:

```go
package main

import "fmt"

func main() {
	fmt.Println("vim-go")
	fmt.Println(sayHi())
	fmt.Println(sayYoo())
}

// sayHi() returns the string "hi"
func sayHi() string {
	return "hi"
}

func sayYoo() string {
	return "yoo"
}
```

Put your cursor on top of the `Println` function just after the `main` function
and call `:GoDoc`. You'll see that it vim-go automatically opens a scratch
window that shows the documentation for you:

```
import "fmt"

func Println(a ...interface{}) (n int, err error)

Println formats using the default formats for its operands and writes to
standard output. Spaces are always added between operands and a newline is
appended. It returns the number of bytes written and any write error
encountered.
```

It shows the import path, the function signature and then finally the doc
comment of the identifier. Initially vim-go was using plain `go doc`, but it
has some shortcomings, such as not resolving based on a byte identifier. `go
doc` is great for terminal usages, but it's hard to integrate into editors.
Fortunately we have a very useful tool called `gogetdoc`, which resolves and
retrieves the AST node for the underlying node and outputs the associated doc
comment.

That's why `:GoDoc` works for any kind of identifier. If you put your cursor under
`sayHi()` and call `:GoDoc` you'll see that it shows it as well. And if you put
it under `sayYoo()` you'll see that it just outputs `no documentation` for AST
nodes without doc comments.

As usual with other features, we override the default normal shortcut `K` so
that it invokes `:GoDoc` instead of `man` (or something else). It's really easy
to find the documentation, just hit `K` in normal mode!

`:GoDoc` just shows the documentation for a given identifier. But it's not a
**documentation explorer**, if you want to explore the documentation there is
third-party plugin that does it:
[go-explorer](https://github.com/garyburd/go-explorer). There is an open bug to
include it into vim-go.

### Identifier resolution

Sometimes you want to know what a function is accepting or returning. Or what
the identifier under your cursor is. Questions like this are common and we have
a command to answer it.

Using the same `main.go` file, go over the `Println` function and call
`:GoInfo`. You'll see that the function signature is being printed in the
status line. This is really great to see what it's doing, as you don't have to
jump to the definition and check out what the signature is. 

But calling `:GoInfo` every time is tedious. We can make some improvements to
call it faster. As always a way of making it faster is to add a shortcut:

```vim
autocmd FileType go nmap <Leader>i <Plug>(go-info)
```

Now you easily call `:GoInfo` by just hitting `<leader>i`. But there is still
room to improve it. vim-go has a support to automatically show the information
whenever you move your cursor. To enable it add the following to your `.vimrc`:

```vim
let g:go_auto_type_info = 1
```

Now whenever you move your cursor onto a valid identifier, you'll see that your
status line is updated automatically. By default it updates every `800ms`. This
is a vim setting and can be changed with the `updatetime` setting. To change it
to `100ms` add the following to your `.vimrc`

```vim
set updatetime=100
```

### Identifier highlighting

Sometimes we just want to quickly see all matching identifiers. Such as variables,
functions, etc.. Suppose you have the following Go code:

```go
package main

import "fmt"

func main() {
	fmt.Println("vim-go")
	err := sayHi()
	if err != nil {
		panic(err)
	}
}

// sayHi() returns the string "hi"
func sayHi() error {
	fmt.Println("hi")
	return nil
}
```

If you put your cursor on top of `err` and call `:GoSameIds` you'll see that
all the `err` variables get highlighted. Put your cursor on the `sayHi()`
function call, and you'll see that the `sayHi()` function identifiers all are
highlighted. To clear them just call `:GoSameIdsClear`

This is more useful if we don't have to call it manually every time. vim-go
can automatically highlight matching identifiers. Add the following to your
`vimrc`:

```vim
let g:go_auto_sameids = 1
```

After restarting vim, you'll see that you don't need to call
`:GoSameIds` manually anymore. Matching identifier variables are now highlighted
automatically for you.

### Dependencies and files

As you know a package can consist of multiple dependencies and files. Even
if you have many files inside the directory, only the files that have the
package clause correctly are part of a package.

To see the files that make a package you can call the following:

```
:GoFiles
```

which will output (my `$GOPATH` is set to `~/Code/Go`):

```
['/Users/fatih/Code/go/src/github.com/fatih/vim-go-tutorial/main.go']
```

If you have other files those will be listed as well. Note that this command is
only for listing Go files that are part of the build. Test files will be not
listed. 

For showing the dependencies of a file you can call `:GoDeps`. If you call  it
you'll see:

```
['errors', 'fmt', 'internal/race', 'io', 'math', 'os', 'reflect', 'runtime',
'runtime/internal/atomic', 'runtime/internal/sys', 'strconv', 'sync',
'sync/atomic ', 'syscall', 'time', 'unicode/utf8', 'unsafe']
```

### Guru

The previous feature was using the `guru` tool under the hood. So let's talk a
little bit about guru. So what is guru? Guru is an editor integrated tool for
navigating and understanding Go code. There is a user manual that shows all the
features: [https://golang.org/s/using-guru](https://golang.org/s/using-guru)

---

Let us use the same examples from that manual to show some of the features we've
integrated into vim-go:

```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

func main() {
	h := make(handler)
	go counter(h)
	if err := http.ListenAndServe(":8000", h); err != nil {
		log.Print(err)
	}
}

func counter(ch chan<- int) {
	for n := 0; ; n++ {
		ch <- n
	}
}

type handler chan int

func (h handler) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	w.Header().Set("Content-type", "text/plain")
	fmt.Fprintf(w, "%s: you are visitor #%d", req.URL, <-h)
}
```

Put your cursor on top of the `handler` and call `:GoReferrers`. This calls the
`referrers` mode of `vim-go`, which finds references to the selected identifier,
scanning all necessary packages within the workspace. The result will be a
quickfix list, so you should be able to jump to the results easily.

---

One of the modes of `guru` is also the `describe` mode. It's just like
`:GoInfo`, but it's a little bit more advanced (it gives us more information).
It shows for example the method set of a type if there is any. It shows the
declarations of a package if selected.

Let's continue with same `main.go` file.  Put the cursor on top of the `URL`
field or `req.URL` (inside the `ServeHTTP` function). Call `:GoDescribe`.  You'll
see a quick fix list populated with the following content:

```
main.go|27 col 48| reference to field URL *net/url.URL
/usr/local/go/src/net/http/request.go|91 col 2| defined here
main.go|27 col 48| Methods:
/usr/local/go/src/net/url/url.go|587 col 15| method (*URL) EscapedPath() string
/usr/local/go/src/net/url/url.go|844 col 15| method (*URL) IsAbs() bool
/usr/local/go/src/net/url/url.go|851 col 15| method (*URL) Parse(ref string) (*URL, error)
/usr/local/go/src/net/url/url.go|897 col 15| method (*URL) Query() Values
/usr/local/go/src/net/url/url.go|904 col 15| method (*URL) RequestURI() string
/usr/local/go/src/net/url/url.go|865 col 15| method (*URL) ResolveReference(ref *URL) *URL
/usr/local/go/src/net/url/url.go|662 col 15| method (*URL) String() string
main.go|27 col 48| Fields:
/usr/local/go/src/net/url/url.go|310 col 2| Scheme   string
/usr/local/go/src/net/url/url.go|311 col 2| Opaque   string
/usr/local/go/src/net/url/url.go|312 col 2| User     *Userinfo
/usr/local/go/src/net/url/url.go|313 col 2| Host     string
/usr/local/go/src/net/url/url.go|314 col 2| Path     string
/usr/local/go/src/net/url/url.go|315 col 2| RawPath  string
/usr/local/go/src/net/url/url.go|316 col 2| RawQuery string
/usr/local/go/src/net/url/url.go|317 col 2| Fragment string
```

You'll see that we can see the definition of the field, the method set and the
`URL` struct's fields. This is a very useful command and it's there if you need
it and want to understand the surrounding code. Try and experiment by calling
`:GoDescribe` on various other identifiers to see what the output is.

---

One of the most asked questions is how to know the interfaces a type is
implementing. Suppose you have a type and with a method set of several methods.
You want to know which interface it might implement. The mode `implement` of
`guru` just does it and it helps to find the interface a type implements.

Just continue with the same previous `main.go` file. Put your cursor on the
`handler` identifier just after the `main()` function. Call `:GoImplements`
You'll see a quick fix list populated with the following content:


```
main.go|23 col 6| chan type handler
/usr/local/go/src/net/http/server.go|57 col 6| implements net/http.Handler
```

The first line is our selected type and the second line will be the interface
it implements. Because a type can implement many interfaces it's a quickfix
list that you can navigate.

---

One of the `guru` modes that might be helpful is `whicherrs`. As you know
errors are just values. So they can be programmed and thus can represent any
type. See what the `guru` manual says:

> The whicherrs mode reports the set of possible constants, global variables,
> and concrete types that may appear in a value of type error. This information
> may be useful when handling errors to ensure all the important cases have
> been dealt with.

So how do we use it? It's easy. We still use the same `main.go` file. Put your
cursor on top of the `err` identifier which is returned from `http.ListenAndServe`.
Call `:GoWhicherrs` and you'll see the following output:

```
main.go|12 col 6| this error may contain these constants:
/usr/local/go/src/syscall/zerrors_darwin_amd64.go|1171 col 2| syscall.EINVAL
main.go|12 col 6| this error may contain these dynamic types:
/usr/local/go/src/syscall/syscall_unix.go|100 col 6| syscall.Errno
/usr/local/go/src/net/net.go|380 col 6| *net.OpError
```

This is a classic `quickfix` output and you can navigate between them. You'll
see that the `err` value may be the `syscall.EINVAL` constant or it also might
be the dynamic types `syscall.Errno` or `*net.OpError`. As you see this is
really helpful when implementing custom logic to handle the error differently if
needed. Note that this query needs a guru `scope` to be set. We'll going to
cover in a moment what a `scope` is and how you can change it dynamically.

---

Let's continue with the same `main.go` file.  Go is famous for its concurrency
primitives, such as channels. Tracking how values are sent between channels can
sometimes be hard. To understand it better we have the `peers` mode of `guru`.
This query shows the set of possible send/receives on the channel operand (send
or receive operation).

Move your cursor to the following expression and select the whole line:

```go
ch <- n
```

Call `:GoChannelPeers`. You'll see a quickfix window with the following content:

```
main.go|19 col 6| This channel of type chan<- int may be:
main.go|10 col 11| allocated here
main.go|19 col 6| sent to, here
main.go|27 col 53| received from, here
```

As you see you can see the allocation of the channel, where it's sending and
receiving from. Because this uses pointer analysis, you have to define a scope.

---


Let us see how function calls and targets are related. This time create the
following files. The content of `main.go` should be:

```go
package main

import (
	"fmt"

	"github.com/fatih/vim-go-tutorial/example"
)

func main() {
	Hello(example.GopherCon)
	Hello(example.Kenya)
}

func Hello(fn func() string) {
	fmt.Println("Hello " + fn())
}
```

And the file should be under `example/example.go`:

```go
package example

func GopherCon() string {
	return "GopherCon"
}

func Kenya() string {
	return "Kenya"
}
```

So jump to the `Hello` function inside `main.go` and put your cursor on top of
the function call named `fn()`. Execute `:GoCallees`. This command shows the
possible call targets of the selected function call. As you see it'll show us
the function declarations inside the `example` function. Those functions are
the callees, because they were called by the function call named `fn()`.

Jump back to `main.go` again and this time put your cursor on the function
declaration `Hello()`. What if we want to see the callers of this function?
Execute `:GoCallers`.

You should see the output:

```
main.go| 10 col 7 static function call from github.com/fatih/vim-go-tutorial.Main
main.go| 11 col 7 static function call from github.com/fatih/vim-go-tutorial.Main
```

As with the other usages you can easily navigate the callers inside the
quickfix window.

Finally there is also the `callstack` mode, which shows an arbitrary path from
the root of the call graph to the function containing the selection.


Put your cursor back to the `fn()` function call inside the `Hello()` function.
Select the function and call `:GoCallstack`. The output should be like
(simplified form):

```
main.go| 15 col 26 Found a call path from root to (...)Hello
main.go| 14 col 5 (...)Hello
main.go| 10 col 7 (...)main
```
It starts from line `15`, and then to line `14` and then ends at line `10`.
This is the graph from the root (which starts from `main()`) to the function we
selected (in our case `fn()`)

---

For most of the `guru` commands you don't need to define any scope. What is a
`scope`? The following excerpt is straight from the `guru`
[manual](http://golang.org/s/using-guru):


> Pointer analysis scope: some queries involve pointer analysis, a technique for
> answering questions of the form “what might this pointer point to?”.  It is
> usually too expensive to run pointer analysis over all the packages in the
> workspace, so these queries require an additional configuration parameter
> called the scope, which determines the set of packages to analyze.  Set the
> scope to the application (or set of applications---a client and server,
> perhaps) on which you are currently working.  Pointer analysis is a
> whole-program analysis, so the only packages in the scope that matter are the
> main and test packages.
> 
> The scope is typically specified as a comma-separated set of packages, or
> wildcarded subtrees like github.com/my/dir/...; consult the specific
> documentation for your editor to find out how to set and vary the scope.


`vim-go` automatically tries to be smart and sets the current packages import
path as the `scope` for you. If the command needs a scope, you're mostly
covered. Most of the times this is enough, but for some queries you might to
change the scope setting. To make it easy to change the `scope` on the fly with
have a specific setting called `:GoGuruScope`

If you call it, it'll return an error: `guru scope is not set`. Let us change
it explicitly to the `github.com/fatih/vim-go-tutorial" scope: 

```
:GoGuruScope github.com/fatih/vim-go-tutorial
```

You should see the message:

```
guru scope changed to: github.com/fatih/vim-go-tutorial
```

If you run `:GoGuruScope` without any arguments, it'll output the following

```
current guru scope: github.com/fatih/vim-go-tutorial
```

To select the whole `GOPATH` you can use the `...` argument:

```
:GoGuruScope ...
```

You can also define multiple packages and also subdirectories. The following
example selects all packages under `github.com` and the `golang.org/x/tools`
package:

```
:GoGuruScope github.com/... golang.org/x/tools
```

You can exclude packages by prepending the `-` (negative) sign to a package.
The following example selects all packages under `encoding` but not
`encoding/xml`:

```
:GoGuruScope encoding/... -encoding/xml
```

To clear the scope just pass an empty string:

```
:GoGuruScope ""
```

If you're working on a project where you have to set the scope always to the
same value and you don't want to call `:GoGuruScope` everytime you start Vim,
you can also define a permanent scope by adding a setting to your `vimrc`. The
value needs to be a list of string types. Here are some examples from the
commands above:

```
let g:go_guru_scope = ["github.com/fatih/vim-go-tutorial"]
let g:go_guru_scope = ["..."]
let g:go_guru_scope = ["github.com/...", "golang.org/x/tools"]
let g:go_guru_scope = ["encoding/...", "-encoding/xml"]
```

Finally, `vim-go` tries to auto complete packages for you while using
`:GoGuruScope` as well. So when you try to write
`github.com/fatih/vim-go-tutorial` just type `gi` and hit `tab`, you'll see
it'll expand to `github.com`

---

Another setting that you should be aware are build tags  (also called build
constraints). For example the following is a build tag you put in your Go
source code:

```
// +build linux darwin
```

Sometimes there might be custom tags in your source code, such as:

```
// +build mycustomtag
```

In this case, guru will fail as the underlying `go/build` package will be not
able to build the package. So all `guru` related commands will fail (even
`:GoDef` when it uses `guru`). Fortunately `guru` has a `-tags` flag that
allows us to pass custom tags. To make it easy for `vim-go` users we have a
`:GoBuildTags`

For the example just call the following:

```
:GoBuildTags mycustomtag
```

This will pass this tag to `guru` and from now on it'll work as expected. And
just like `:GoGuruScope`, you can clear it with:

```
:GoBuildTags ""
```

And finally if you wish you can make it permanent with the following setting:

```
let g:go_build_tags = "mycustomtag"
```

# Refactor it

### Rename identifiers

Renaming identifiers is one of the most common tasks. But it's also something
that needs to be done carefully in order not to break other packages as well. Also just
using a tool like `sed` is sometimes not useful, as you want AST aware
renaming, so it only should rename identifiers that are part of the AST (it
should not rename for example identifiers in other non Go files, say build
scripts)

There is a tool that does renaming for you, which is called `gorename`.
`vim-go` uses the `:GoRename` command to use `gorename` under the hood.  Let us
change `main.go` to the following content:

```go
package main

import "fmt"

type Server struct {
	name string
}

func main() {
	s := Server{name: "Alper"}
	fmt.Println(s.name) // print the server name
}

func name() string {
	return "Zeynep"
}
```

Put your cursor on top of the `name` field inside the `Server` struct and call
`:GoRename bar`.  You'll see all `name` references are renamed to `bar`. The
final content would look like:

```go
package main

import "fmt"

type Server struct {
	bar string
}

func main() {
	s := Server{bar: "Alper"}
	fmt.Println(s.bar) // print the server name
}

func name() string {
	return "Zeynep"
}
```

As you see, only the necessary identifiers are renamed, but the function `name`
or the string inside the comment is not renamed. What's even better is that
`:GoRename` searches all packages under `GOPATH` and renames all identifiers
that depend on the identifier. It's a very powerful tool.

### Extract function

Let's move to another example. Change your `main.go` file to:

```go
package main

import "fmt"

func main() {
	msg := "Greetings\nfrom\nTurkey\n"

	var count int
	for i := 0; i < len(msg); i++ {
		if msg[i] == '\n' {
			count++
		}
	}

	fmt.Println(count)
}
```

This is a basic example that just counts the newlines in our `msg` variable. If
you run it, you'll see that it outputs `3`. 

Assume we want to reuse the newline counting logic somewhere else. Let us
refactor it. Guru can help us in these situations with the `freevars` mode. The
`freevars` mode shows variables that are referenced but not defined within a
given selection. 

Let us select the piece in `visual` mode:

```go
var count int
for i := 0; i < len(msg); i++ {
	if msg[i] == '\n' {
		count++
	}
}
```

After selecting it, call `:GoFreevars`. It should be in form of
`:'<,'>GoFreevars`. The result is again a quickfix list and it contains all the
variables that are free variables. In our case it's a single variable and the
result is:


```go
var msg string
```

So how useful is this? This little piece of information is enough to refactor
it into a standalone function. Create a new function with the following content:

```go
func countLines(msg string) int {
	var count int
	for i := 0; i < len(msg); i++ {
		if msg[i] == '\n' {
			count++
		}
	}
	return count
}
```

You'll see that the content is our previously selected code. And the input to
the function is the result of `:GoFreevars`, the free variables. We only
decided what to return (if any). In our case we return the count. Our `main.go` will be in the form of:

```go
package main

import "fmt"

func main() {
	msg := "Greetings\nfrom\nTurkey\n"

	count := countLines(msg)
	fmt.Println(count)
}

func countLines(msg string) int {
	var count int
	for i := 0; i < len(msg); i++ {
		if msg[i] == '\n' {
			count++
		}
	}
	return count
}
```

That's how you refactor a piece of code. `:GoFreevars` can be used also to
understand the complexity of a code. Just run it and see how many variables are
dependent to it.

# Generate it

Code generation is a hot topic. Because of the great std libs such as go/ast,
go/parser, go/printer, etc.. Go has the advantage of being able to create great generators
easily. 

First we have the `:GoGenerate` command that calls `go generate` under the
hood. It just works like `:GoBuild`, `:GoTest`, etc.. If there are any errors it
also shows them so you can easily fix it.

### Method stubs implementing an interface

Interfaces are really great for composition. They make your code easier to deal
with. It's also easier for you to create tests as you can mock functions that
accept an interface type with a type that implements methods for testing.


`vim-go` has support for the tool [impl](https://github.com/josharian/impl).
`impl` generates method stubs that implement a given interface. Let us change
`main.go`'s content to the following:

```go
package main

import "fmt"

type T struct{}

func main() {
	fmt.Println("vim-go")
}
```

Put your cursor on top of `T` and type `:GoImpl`. You'll be prompted to write
an interface. Type `io.ReadWriteCloser` and hit enter. You'll see the content
changed to:

```go
package main

import "fmt"

type T struct{}

func (t *T) Read(p []byte) (n int, err error) {
	panic("not implemented")
}

func (t *T) Write(p []byte) (n int, err error) {
	panic("not implemented")
}

func (t *T) Close() error {
	panic("not implemented")
}

func main() {
	fmt.Println("vim-go")
}
```

That's really neat as you see. You can also just type `:GoImpl
io.ReadWriteCloser` when you're on top of a type and it'll do the same.

But you don't need to put your cursor on top of a type.  You can invoke it from
everywhere. For example execute this:

```
:GoImpl b *B fmt.Stringer
```

You'll see the following will be created:

```go
func (b *B) String() string {
	panic("not implemented")
}
```

As you see this is very helpful, especially if you have a large interface with
large method set. You can easily generate it, and because it uses `panic()`
this compiles without any problem. Just fill the necessary parts and you're
done.

# Share it

`vim-go` has also features to easily share your code with other via
https://play.golang.org/. As you know the Go playground is a perfect place to
share small snippets, exercises and/or tips & tricks. There are times you are
playing with an idea and want to share with others. You copy the code and visit
play.golang.org and then paste it.  `vim-go` makes all these better with the
`:GoPlay` command.

First let us change our `main.go` file with the following simple code:

```go
package main

import "fmt"

func main() {
	fmt.Println("vim-go")
}
```

Now call `:GoPlay` and hit enter. You'll see that `vim-go` automatically
uploaded your source code `:GoPlay` and also opened a browser tab that shows
it. But there is more. The snippet link is automatically copied to your
clipboard as well. Just paste the link to somewhere. You'll see the link is the
same as what's on play.golang.org.

`:GoPlay` also accepts a range. You can select a piece of code and call
`:GoPlay`. It'll only upload the selected part.

There are two settings to tweak the behavior of `:GoPlay`. If you don't like
that `vim-go` opens a browser tab for you, you can disable it with:

```
let g:go_play_open_browser = 0
```

Secondly, if your browser is misdetected (we're using `open` or `xdg-open`) you
can manually set the browser via:

```
let g:go_play_browser_command = "chrome"
```
