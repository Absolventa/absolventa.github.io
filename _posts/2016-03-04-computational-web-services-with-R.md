---
layout: post
title: "Computational web services with R programming language"
date: 2016-03-04 05:30
author: rbn
teaser: "The R programming language is a swiss knife for mathematical problems. We examine a way of using these capabilities in a web development environment, concentrating on the setup of a simple web application completely written in R using the library Rook."
comments: true
tags:
  - R
  - Rook
  - web services
  - JSON
---
If you encounter computational problems of certain complexity, the [R programming language](https://www.r-project.org/)
is most probably a good tool of choice to satisfy your needs. The main tasks R was developed for
were dealing with large datasets, statistical analysis and computation. Within this area it has become
a solid, mature open source platform, widely used in academia. 

What I like about R is that the barriers for usage are low: It's freely available for nearly 
every major UNIX enviornment and Windows, it can be used within a special stand-alone stable [IDE](https://www.rstudio.com/home/)
that is very pleasant for beginners or just by the command line if you're used to.

In addition to that its maturity has grown so far that even people 
whithout a degree in mathematics or computer science can have access to advanced statistical 
techniques in a well designed manner for their projects. 

Most of a web dev's day-to-day business possibly lives in another neighborhood. The barycenter 
of web development is organizing the communication of participants of several distributed systems. 
Its far more often dealing with applications talking HTTP (the lingua franca of web applications!) 
to each other than optimizing specific algorithms w.r.t to numerical issues or similar.

But for me the connection of both worlds is very interesting and fruitful: 
Employing statistical methods can be a benefit for a vast amount of web application types, 
that's why it's interesting to elaborate how this can be achieved. I'd like to describe one 
possible way here. 

### A web application in R

The most natural approach would be to implement the desired algorithm and the web application
in the same language. As mentioned, R is a swiss knife for mathematical
flavoured problems related to data. But in 2016 most web applications were 
not typically written in R and I admit this felt a bit unfamiliar for me, too.

Thanks to the work that a lot of people who put a lot effort into additional libraries of R it 
actually *is* possible and as it turns out it's not that complicated. Therefore I'd like to 
explore with you an hello-world example of writing a small web application 
completely in R. I'd like to share a `Hello world!`-example of such a web application: 
It will take a query string from the URL and serves some basic JSON output including that parameter.

We are going to use a library for `R` named `Rook`. If you ever have seen some web development code
in Ruby, you surely will have faced [Rack](http://rack.github.io/) at some point. `Rack` is a popular simple web server interface 
for Ruby. There are more really nice alternatives for doing web application relevant work in R (e.g. [Shiny](http://shiny.rstudio.com/) or
  [plumber](http://plumber.trestletech.com/)), but the final point where Rook got me 
  motivated a lot was because it was highly inspired by Rack - and I knew of the power of simplicity of Rack
  in the Ruby ecosystem. I am very thankful for having Rook since it was exactly what I was looking for my specific problem.

### Resources for learning basic R syntax

If you're looking to learn the absolute basics with `R` try the 
[free Code School course](http://tryr.codeschool.com/) or have a look at the [language definition](https://cran.r-project.org/doc/manuals/r-devel/R-lang.html).

Let's get started!

### (1/5) Installation of R

Installing R on a unixoide system should be pretty easy. My example lives on a Ubuntu machine,
so I'll use this as reference. But you are encouraged trying it on other systems, too. It should be
possible! 

If you're running Ubuntu you can install it from the universe sources via

```bash
$ sudo apt-get install r-base r-recommended 
```
Afterwards there will be a binary named `R` ready for you at th command line. 
Called without arguments it will open a R console for you. You can exit it by invoking the quit function `q()`

```bash
$ R
> s <- "Hello"
> s
[1] "Hello"
> q()
Save workspace image? [y/n/c]: n
$
```

### (2/5) Installation of `rApache`

My example setup uses an Apache2 web server. In addition to the Apache web server
we need an extension called [rApache](rapache.net). The installation for Ubuntu works
this way:

```bash
$ sudo add-apt-repository ppa:opencpu/rapache
$ sudo apt-get update
$ sudo apt-get install libapache2-mod-r-base
```

### (3/5) Installation of `Rook` library

`R` has its own package/library management system called [CRAN](https://cran.r-project.org/), which
basically is a network of ftp and web servers delivering R libraries to your machine. To
install the library 'Rook' from the `CRAN` package sources, open a `R` console and type

```R
> install.packages('Rook')
```

Afterwards you can load the package when needed via

```R
> library('Rook')
```

You can find detailed information of `Rook` [here in the documentation file](https://cran.r-project.org/web/packages/Rook/Rook.pdf). 

### (4/5) A Hello World example of Rook

Let's examine how a Rook application has to look like. In the documentation you'll find

> A Rook application is an R reference class object that implements a ’call’ method or an R closure
> that takes exactly one argument, an environment, and returns a list with three named elements:
> 'status', 'headers', and 'body'.

Here you can see how Rook quotes Rack: An application as a function of something
called an environment returning a list of status, header and a body! So basically we need an object 
we can invoke a function named `call` on and we need to return a list 
containing the response data. But wait, let's have small break here: What does `reference class` mean?

For historical reasons, R has three distinct object oriented systems
built in: S3 classes, S4 classes and so-called Reference classes. Each of these are 
slightly different and completely distinct to each other. They have in common that
these systems are providing (in different ways) object orientation mechanisms for the 
R language.  

Let's concentrate on Reference Classes, or short refclasses, which are introduced in R v2.12. Even there
is complexity under the hood, let's simply think of it of a way of defining class-like structures in R. We can define
a new refclass by

```R
Monkey <- setRefClass("Monkey")
```

Afterwards one can create instances ("objects") of a refclass by 

```R
leila <- Monkey$new() 
```

`leila` is now a 'Reference class object of class "Monkey"'. 

Heading back to the definition of a Rook application this means two things for us: 
We have to implement our application logic as a function that returns the desired list with status, header information and a response body.
Then we need to "attach" this function to a refclass. After this 
we can construct "instances" of our application that can be used somewhere else. 

```R
hello_world <- function(env){
  body <- '<h1>Hello World!</h1>'
  
  list(
    status = 200L,
    headers = list(
      'Content-Type' = 'text/html'
    ),
    body = body
  )
}

application_factory <- setRefClass(
  'HelloWorld',
  methods = list(
    call = hello_world
  )
)
```

Having this above we cann create an instance of the refclass `HelloWord` and invoke the function `call`:

```R
# generate a fresh application
app <- application_factory() 

# Now invoke the application with an empty environment
env <- list() # … just an empty Array
app$call(env) # will return a list with status, header and body
```

If the whole refclass thing confuses you here's the good message: 
You don't need to state this as explicit as done above. It's enough
to implement the raw logic part as a function returning a list. 
With having a function `hello_rook` returning a list of
status, header and body you can bind our application to a server by calling 
`Rook::Server$call(hello_rook)`. The rest is maintained by Rook internally
then¹. Here's a running example:

```R
library('Rook')

hello_rook <- function(env){
  
  query_input <- env$QUERY_STRING

  body <- paste(
    '{',
      '"status": 200,',
      '"input":"', query_input, '",',
    '}', 
    sep=''
  )

  list(
    status = 200L,
    headers = list(
      'Content-Type' = 'application/json'
    ),
    body = body
  )
}

Rook::Server$call(hello_rook)
```

### (5/5) Point your Apache server to your R app

We're almost finished, all pieces of the puzzle are on the table
now. To "start" your app you just have to invoke `rApache` and
tell your Apache where to look for the script. Head for
your Apache config file (mine is located in `/etc/apache2/`). 
Add the following snippet to use `rApache` to run your Rook script:

```
LoadModule R_module           /apache/module/path/mod_R.so

<Location /hello_rook>
    SetHandler r-handler
    RFileHandler /home/robin/hello_rook/hello_rook.R
</Location>
```

If you visit `http://YOUR_HOST_APACHE_WAS_BOUND_TO/hello_rook`
you can inspect your sample JSON output!

If you visit `http://YOUR_HOST_APACHE_WAS_BOUND_TO/hello_rook?Yeah!`
your JSON input should have reflected your query input "Yeah!". 

### Ok. Where to go now? 

The problem I'm currently working on is passing data into the R application. You 
can simply put large CSV files onto the server, loading them with R is
pretty natural. But up to now I have not tried to catch and parse POST requests 
with Rook. R is able to maintain a database connection (e.g. with Postgres), too, but
that's another undiscovered (yet interesting!) area for me 😉 

Thanks for reading and happy coding!

¹) In fact I'm not really certain how this works. So finally I decided to document it how 
*I* understood it. Maybe that information is not 100% correct. If you know better, 
I'd be happy if you'd tell me.
