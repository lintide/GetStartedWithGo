Get Started With Go
================
## What is Go?
Go is a new, general-purpose programming language.

- Compiled
- Statically typed
- Concurrent
- Simple
- Productive

"Go is a wise,clean,insightful,fresh thinking approach to the greatest-hits subset of the well understood." - Michael T.Jones

## History ##
- Project starts at google in 2007 (by Griesemer, Pike, Thompson)
- Open source release in November 2009
- More than 250 contributors join the project
- Version 1.0 relase in March 2012

## Install Go ##
  [golang.org/doc/install](http://golang.org/doc/install)

- Install from binary distributions or build from source
- 32- and 64-bit x86 and ARM processors
- Windows, Mac OS X, Linux, and FreeBSD
- Other platforms may be supported by **gccgo**

## Test your Go installation ##
Put this code into hello.go:

	package main

	import "fmt"

	func main(){
		fmt.Println("Greetings, fellow gopher")
	}

Run the program:

	$ go run hello.go

## The go tool ##
The **go** tool is the standard for building,testing, and installing Go programs.
Compile and run **hello.go**

	$ go run hello.go

Run **zip** tests:

	$ go test archive/zip

Build and format the files in the current directory:

	$ go build
	$ go fmt

Fetch and install **websocket**:

	$ go get code.google.com/p/go.net/websocket

## Workspaces ##
The **go** tool derives build instructions from Go source code.

There's no need to write and maintain build scripts.

For this to work, some prescribed directory structure, know as a workspace, is required.

	workspace/
	  bin # executable binaries
	  pkg # compiled object files
	  src # source code


## Create a workspace
Create your workspace now.

	$ mkdir -p $HOME/gocode/src

(The **bin** and **pkg** sub-directories will be created by the **go** tool)

Tell the **go** tool where your workspace is by setting the **GOPATH** environment variable:

	export GOPATH=$HOME/gocode

You may also want to add the bin sub-directory of your workspace to your **PATH**:

	export PATH=$PATH:$GOPATH/bin

This lets you run your Go programs without specifying their full path.

b (You may want to put these **export** commands in the **.bash_profile** file in your home directory.)

## Choose a namespace
Choose a special place for your Go code.

I use **"gitbub.com/nf"**, the root of my GitHub account(useful with go get).

	$ mkdir -p $GOPATH/src/github.com/nf

Create a **hello** directory in your namespace and copy **hello.go** there:

	$ mkdir $GOPATH/src/github.com/nf/hello
	$ cp hello.go $GOPATH/src/github.com/nf/hello

Now you can build install the hello program with the **go** tool:

	$ go install github.com/nf/hello

This builds an executable named **hello**, and installs it to the **bin** directory of your workspace.

	$ $GOPATH/bin/hello
	Hello, fellow gopher

## Our project ##
A command-line program that fetches and displays the latest headlines from the **golang** page on Reddit.

The program will:

- make an HTTP request to the Reddit API.
- decode the JSON response into a Go data structure, and
- print each link's title, URL, and number of comments.

To get started, create directory inside your namespace called **reddit**:

	$ mkdir $GOPATH/src/github.com/nf/reddit

This is where you will put your Go source files.

## Make an HTTP request ##
This program makes an HTTP request to the Reddit API and copies its response to standard output. Put this in a file named **main.go** inside your **reddit** directory.

	package main

	import (
		"io"
		"log"
		"net/http"
		"os"
	)

	func main(){
		resp, err := http.Get("http://reddit.com/r/golang.json")
		if err != nil {
			log.Fatal(err)
		}
		if resp.StatusCode != http.StatusOK {
			log.Fatal(resp.Status)
		}
		_, err = io.Copy(os.Stdout, resp.Body)
		if err != nil {
			log.Fatal(err)
		}
	}

## Make an HTTP request: package statement ##
All Go code belongs to a package.

	package main

Go programs begin with function **main** inside package **main**.

## Make an HTTP request: import statement ##
The import declaration specifies the file's dependencies.

	import (
		"io"
		"log"
		"net/http"
		"os"
	)

Each string is an import path. It tells the Go tools where to find the package.

These packages are all from the Go standard library.

## Make an HTTP request: function declaration ##

	func main(){
    	resp, err := http.Get("http://reddit.com/r/golang.json")
    	if err != nil {
        	log.Fatal(err)
    	}
    	if resp.StatusCode != http.StatusOK {
        	log.Fatal(resp.Status)
    	}
    	_, err = io.Copy(os.Stdout, resp.Body)
    	if err != nil {
        	log.Fatal(err)
    	}
	}

This is a function declaration. The main function takes no arguments and has no return values.

## Make an HTTP request: http.Get ##

	func main(){
    	resp, err := http.Get("http://reddit.com/r/golang.json")
    	if err != nil {
        	log.Fatal(err)
    	}
    	if resp.StatusCode != http.StatusOK {
        	log.Fatal(resp.Status)
    	}
    	_, err = io.Copy(os.Stdout, resp.Body)
    	if err != nil {
        	log.Fatal(err)
    	}
	}

Call the **Get** function from the **http** package, passing the URL of the Reddit API as its only argument.

Declare two variables(**resp** and **err**) and give them the retrun values of the function call, (Yes, Go functions can return multiple values.) The **Get** function returns ***http.Response** and an **error** values.

## Make an HTPP request:error handing ##
	
	if err != nil {
        	log.Fatal(err)
    	}
    .
    .
    .
    if err != nil {
        	log.Fatal(err)
    	}

Compare **err** against **nil**, the zero-value for the built-in **error** type.

The **err** variable will be nil if the request was successful.

If not, call the **log.Fatal** function to print the error message and exit the program.

## Make an HTTP request: check status ##

	if resp.StatusCode != http.StatusOK {
		log.Fatal(resp.Status)
	}

Test that the HTTP server returned a "200 OK" response.

If not, bail, printing the HTTP status message ("500 Internal Server Error", for example).

## Make an HTTP request: copy ##

	_, err = io.Copy(os.Stdout, resp.Body)

Use **io.Copy** to copy the HTTP response body to standard output (**os.Stdout**).

	package io
	func Copy(dst Writer, src Reader)(written int64, err error)

The **resp.Body** type implements **io.Reader** and **os.Stdout** implements **io.Writer**.

## Data structures ##
The Reddit API returns JSON data like this:

	{"data": {"children": [
		{"data": {
			"title": "The Go homepage",
			"url": "http://golang.org",
			...
		}},
		...
	]}}

Go's **json** package(**encoding/json**) decodes JSON-encoded data into native Go data structures. To decode the API response, declare some types that reflect the structure of the JSON data:

	type Item struct {
		Title string
		URL string
	}

	type Response struct {
		Data struct {
			Children []struct {
				Data Item
			}
		}
	}

## Decode the response ##
Instead of copying the HTTP response body to standard output

	_, err = io.Copy(os.Stdout, resp.Body)

we use the json package to decode the response into our Response data structure.

	r := new(Response)
	err = json.NewDecoder(resp.Body).Decode(r)

Initialize a new **Response** value, store a pointer to it in the new variable **r**.

Create a new **json.Decoder** object and decode the response body into **r**.

As the decoder parses the JSON data it looks for corresponding fields of the same names in the **Response** struct. The **"data"** field of the top-level JSON object is decoded into the **Response** struct's **Data** field, and JSON array **children** is decoded into **Children** slice, and so on.

## Print the data ##

	for _, child := range r.Data.Children {
		fmt.Println(child.Data.Title)
	}

Iterate over the **Children** slice, assigning the slice value to **child** on each iteration.

The **Println** call prints the item's **Title** followed by a newline.

## Separation of concerns ##
So far, all the action happens in the main function.

As the program grows, structure and modularity become important.

What if we want to check several subreddits? Or share this functionality with another program?

Create a function named **Get** that takes the name of subreddit, makes the API call, and returns the items from that subreddit.

	func Get(reddit string) ([]Item, error) {}

**Get** takes a tring, **reddit**, and returns a slice of **Item** and an **error** value.

## Get:construct the URL ##

	func Get(reddit string) ([]Item, error){
		url := fmt.Sprintf("http://reddit.com/r/%s.json", reddit)
		resp, err := http.Get(url)
		if err != nil {
			return nil, err
		}
		defer resp.Body.Close()
		if resp.StatusCode != http.StatusOK {
			return nil, errors.New(resp.Status)
		}
		r := new(Response)
		err = json.NewDecoder(resp.Body).Decode(r)
		if err != nil {
			return nil, err
		}
		items := make([]Item, len(r.Data.Children))
		for i, child := range r.Data.Children {
			items[i] = child.Data
		}
		return items, nil
	}

Use **fmt.Sprintf** to construct the request URL from the provided **reddit** string.

## Get: return ##

	func Get(reddit string) ([]Item, error){
		url := fmt.Sprintf("http://reddit.com/r/%s.json", reddit)
		resp, err := http.Get(url)
		if err != nil {
			return nil, err
		}
		defer resp.Body.Close()
		if resp.StatusCode != http.StatusOK {
			return nil, errors.New(resp.Status)
		}
		r := new(Response)
		err = json.NewDecoder(resp.Body).Decode(r)
		if err != nil {
			return nil, err
		}
		items := make([]Item, len(r.Data.Children))
		for i, child := range r.Data.Children {
			items[i] = child.Data
		}
		return items, nil
	}

Exiting the function, return a nil slice and a non-nil error value, or vice versa.

## Get: making an error ##

	func Get(reddit string) ([]Item, error){
		url := fmt.Sprintf("http://reddit.com/r/%s.json", reddit)
		resp, err := http.Get(url)
		if err != nil {
			return nil, err
		}
		defer resp.Body.Close()
		if resp.StatusCode != http.StatusOK {
			return nil, errors.New(resp.Status)
		}
		r := new(Response)
		err = json.NewDecoder(resp.Body).Decode(r)
		if err != nil {
			return nil, err
		}
		items := make([]Item, len(r.Data.Children))
		for i, child := range r.Data.Children {
			items[i] = child.Data
		}
		return items, nil
	}

The response's **Status** field is just a string; use the **errors.New** function to convert it to an **error** value.

## Get: defer clean-up work ##

	func Get(reddit string) ([]Item, error){
		url := fmt.Sprintf("http://reddit.com/r/%s.json", reddit)
		resp, err := http.Get(url)
		if err != nil {
			return nil, err
		}
		defer resp.Body.Close()
		if resp.StatusCode != http.StatusOK {
			return nil, errors.New(resp.Status)
		}
		r := new(Response)
		err = json.NewDecoder(resp.Body).Decode(r)
		if err != nil {
			return nil, err
		}
		items := make([]Item, len(r.Data.Children))
		for i, child := range r.Data.Children {
			items[i] = child.Data
		}
		return items, nil
	}

Defer a call to the response body's **Close** method. to guarantee that we clean up after the HTTP request.The call will be executed after the function returns.

## Get: prepare the response ##

	func Get(reddit string) ([]Item, error){
		url := fmt.Sprintf("http://reddit.com/r/%s.json", reddit)
		resp, err := http.Get(url)
		if err != nil {
			return nil, err
		}
		defer resp.Body.Close()
		if resp.StatusCode != http.StatusOK {
			return nil, errors.New(resp.Status)
		}
		r := new(Response)
		err = json.NewDecoder(resp.Body).Decode(r)
		if err != nil {
			return nil, err
		}
		items := make([]Item, len(r.Data.Children))
		for i, child := range r.Data.Children {
			items[i] = child.Data
		}
		return items, nil
	}

Use the make function to allocate an **Item** slice big enough to store the response data.

## Get: convert the response ##

	func Get(reddit string) ([]Item, error){
		url := fmt.Sprintf("http://reddit.com/r/%s.json", reddit)
		resp, err := http.Get(url)
		if err != nil {
			return nil, err
		}
		defer resp.Body.Close()
		if resp.StatusCode != http.StatusOK {
			return nil, errors.New(resp.Status)
		}
		r := new(Response)
		err = json.NewDecoder(resp.Body).Decode(r)
		if err != nil {
			return nil, err
		}
		items := make([]Item, len(r.Data.Children))
		for i, child := range r.Data.Children {
			items[i] = child.Data
		}
		return items, nil
	}

Iterate over the response's **Children** slice, assigning each child's **Data** element to the coresponding element in the items slice.

## Use Get in main ##
In the **main** function, replace the http request and JSON decoding code with a single call to **Get**.

	func main(){
		items, err := Get("golang")
		if err != nil {
			log.Fatal(err)
		}
		for _, item := ranage items {
			fmt.Println(item.Title)
		}
	}

The print loop becomes clearer, too.

However, it's not very useful to print only the title of the items. Let's address that.

## The Stringer interface ##
The **fmt** package knows how to format the built-in types, but it can be told how to format user-defined types, too.

When you pass a value to the **fmt.Print** functions, it checks to see if it implements the **fmt.Stringer** interface:

	type Stringer interface {
		String() string
	}

Any type that implements a **String() string** method is a **Stringer**, and the **fmt** package will use that method to format values of that type.

## Formatting Items ##
A method declaration is just like a function declaration, but the receiver comes first.

Here's a **String** method for the **Item** type that returns the title, a newline, and the URL:

	func (i Item) String() string {
		return fmt.Sprintf("%s\n%s", i.Title, i.URL)
	}

To print the item we just pass it to Println, which uses the provided String method to format the **Item**.

	fmt.Println(item)

## Richer formatting(1/2)
Let's go a step further. One way to judge how interesting a link might be is by the dicussion surrounding it.Let's display the number of commnets for each **Item** as well.

	{"title": "The Go homepage",
	 "url": "http://golang.org",
	 "num_comments": 10 
	}

Update the **Item** type to include a **Comments** field:

	type Item struct {
		Title string
		URL string
		Comments int `json:"num_comments"`
	}

The new **Comments** field has a ”struct tag", a string that annotates the field. Go code can use the **reflect** package to inspect this information at runtime.

This tag, **json:"num_comments"**, tells the **json** package to decode the **"num_comments"** field of the JSON object into the **Comments** field (and the reverse, when encoding).

## Richer formatting (2/2) ##
Now the **String** method can be a little more complex:

	func (i Item) String() string {
		com := ""
		switch i.Comments {
		case 0:
			// nothing
		case 1:
			com = " (1 comment)"
		default:
			com = fmt.Sprintf(" (%d comments)", i.Comments)
		}
		return fmt.Sprintf("%s%s\n%s", i.Title, com, i.URL)
	}

Observe that, unlike some languages, Go's switch statements do not fall through by default.

Now when we run our program we should see a nicely formatted list of links.

## A new package (1/3) ##
This is useful code. Let's organize it to make it more accessible to others by putting it in an importable package.

Create a new directory inside your **reddit** directory named **geddit**, and copy your **main.go** file there.

**reddit** is the name of the library and **geddit** as that of the command-line client.

	$ cd $GOPATH/src/github.com/nf/reddit
	$ mkdir geddit
	$ cp main.go geddit/

Rename the **main.go** inside the **reddit** directory to **reddit.go**.(Not necessary; just a convention.)

	$ mv main.go reddit.go

## A new package (2/3) ##
Change the package statement at the top of **reddit.go** from **package main** to **package reddit**.

It is convention that the package name be the same as the last element of the import path.

The convention makes packages predictable to use:

	import "github.com/nf/reddit"

	func foo() {
		r, err := reddit.Get("golang") // "reddit" here is the package name
		// ...
	}

The only strict requirement is that it must not be **package main**.

Also remove the **main** function from **reddit.go**, and any unused package imports. (The compiler will tell you which packages are unused.)

## A new package (3/3) ##
The **reddit.go** file now looks like this:

	package reddit
	import (
		// omitted
	)

	type Response struct {
		// omitted
	}

	type Item struct {
		// omitted
	}

	func (i Item) String() string {
		// omitted
	}

	func Get(reddit string) ([]Item, error){
		// omitted
	}

## Using the reddit package ##
Edit the **geddit/main.go** file to remove the **Get**, **Item**, and **Response** declarations, import the **reddit** package, and use the **reddit**. prefix before the **Get** invocation:

	package main

	import (
		"fmt"
		"github.com/nf/reddit"
		"log"
	)

	func main() {
		items, err := reddit.Get("golang")
		if err != nil {
			log.Fatal(err)
		}
		for _, item := range items {
			fmt.Println(item)
		} 
	}

## Documentation (1/3) ##
**Godoc** is the Go documentation tool. It reads documentation directory from Go source files. It's easy to keep documentation and code in sync when they live together in the same place.

Here's our reddit package when viewed from **godoc**:

	$ godoc github.com/nf/reddit
	PACKAGE

	package reddit
	    import "github.com/lintide/reddit"


	FUNCTIONS

	func Get(reddit string) ([]Item, error)


	TYPES

	type Item struct {
	    Title    string
	    URL      string
	    Comments int `json:"num_comments"`
	}

	func (i Item) String() string

	type Response struct {
	    Data struct {
	        Children []struct {
	            Data Item
	        }
	    }
	}


	SUBDIRECTORIES

		geddit

## Documentation (2/3) ##
First, hide the **Response** type by renaming it to **response**.

In Go, top-level declarations beginning with an uppercase letter are "exported"(visible outside the package). All other names are private and inaccessbile to code outside the package.

To document the remaining visible names, add a comment directly above their declarations:

	// Item describes a Reddit item.
	type Item struct {

<span></span>

	// Get fetchs the most recent Items posted to the specified subreddit.
	func Get(reddit string) ([]Item, error){

Most importantly, document the package itself by adding a comment to the package clause:

	// Package reddit implements a basic client for the Reddit API.
	package reddit

Don't worry about documenting the **String** method, as all Go programmers should be familiar with it and its purpose.

## Documentation (3/3) ##
The **godoc** output for our revised package:

	PACKAGE

	package reddit
	    import "github.com/lintide/reddit"

	    Package reddit implements a basic client for the Reddit API

	FUNCTIONS

	func Get(reddit string) ([]Item, error)
	    Get fetches the most recent Items posted to the specified subreddit.


	TYPES

	type Item struct {
	    Title    string
	    URL      string
	    Comments int `json:"num_comments"`
	}
	    Item describes a Reddit item.

	func (i Item) String() string

## Push to github ##

	$ cd $GOPATH/src/github.com/nf/reddit

	$ git init

	$ git add reddit.go geddit/main.go

	$ git commit -m "initial commit"

	$ git remote add origin git@github.com/nf/reddit.git

	$ git push -f origin master

## Install with go get ##
Use the go tool to automatically check out and install Go code:

	$ go get github.com/nf/reddit/geddit

This checks out the repository to **$GOPATH/src/github.com/nf/reddit** and installs the binary to **$GOPATH/bin/geddit**. the **bin** directory is in my PATH, so I can now run:

	$ geddit

The **go get** command can fetch code from

- Bitbucket
- GitHub
- Google Code
- Launchpad

as well as arbitrary Git, Mercurial, Subversion, and Bazaar repositories.

## Homework ##
Some ideas:

- Implement a command-line interface to specify the subreddit(s) to query.
- Expand the reddit package to support more of the Reddit API.
- Learn about Go's concurrency primitives and perform multiple requests in parallel.

## Where to go from here ##
Learn Go:
<tour.golang.org>

Documentation and articles:
<golang.org/doc>

Standard library reference:
<golang.org/pkg>

