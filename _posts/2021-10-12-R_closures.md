---
title: 'R closures and their surprising behaviours'
date: 2021-10-20
permalink: /posts/2021/10/20/
tags:
  - R
  - Random Coding Discoveries
  - Functional Programming
---

Functional programming allows you to both use and create functions
inside other functions. Using function as arguments works exactly the
same as specifying other arguments however when creating functions
within function there are a few more quirks to be aware of.

## Pure Functions

Ideally we would like to work as much as possible with **Pure
Functions** these are functions which, given the same arguments, always
return the same value and don’t modify any global state. There are times
when this is simply not possible however for example when reading or
writing out data or when working with some random element. In all other
situations, we aim to make our functions as “pure” as possible.

## Closures & Environments

When creating functions within other functions, we need to be careful as
some nuances occur. Here is a simple example to illustrate. Suppose we
want to create a function which returns a function to take numbers to a
chosen power. We would do this as seen below.

    exp_func_creator<-function(exponent){
      exp_func<-function(x){x^exponent}
      return(exp_func)
    }

Now we create a cubing function and it works as expected

    cubing_func<-exp_func_creator(3)
    cubing_func(4)

    ## [1] 64

When we look at the function however we can see something strange

    cubing_func

    ## function(x){x^exponent}
    ## <environment: 0x560fdd9b81d8>

The function still takes in an argument `exponent` but where does it get
this value from? If we look at all the objects in our current
environment using the `ls()` function we see that there is indeed no
value of `exponential`.

    ls()

    ## [1] "cubing_func"      "exp_func_creator"

Also, if we set a value of `exponential` in the global environment this
doesn’t affect how `cubing_func` runs.

    exponential<-2
    cubing_func(4)

    ## [1] 64

This is all because of the environment in which `cubing_func` was
created which we call the **enclosing environment**. This is the
environment that we see when we run type `cubing_func` and can also be
examined by typing `environment(cubing_func)`. This environment is the
**executing environment** created when we ran `exp_func_creator`.

If we modify `exp_func_creator` we can see more clearly what is going
on.

    exp_func_creator2<-function(exponent){
      exp_func<-function(x){x^exponent}
      #return the current environment
      print(environment())
      #return all variables within environment
      print(ls())
      #return value of exponent in this environment
      print(get("exponent",inherits = F))
      return(exp_func)
    }

    cubing_func2<-exp_func_creator2(3)

    ## <environment: 0x560fdd58fd78>
    ## [1] "exp_func" "exponent"
    ## [1] 3

    #We can now confirm the enclosing environment of cubing
    environment(cubing_func2)

    ## <environment: 0x560fdd58fd78>

If we were to run `exp_func_creator` again it would create a new
environment to execute this function. So that each created function has
it’s own separate enclosing environment.

## Enclosing Environments

Enclosing environments are not unique to functions created inside other
functions. They exist for all function in R. This can lead to some
slightly surprising results.

**Important Fact** The parent environment of a function’s execution
environment is the functions enclosing environment NOT the environment
in which the function was called. Here is a simple example with the same
function

    exp_func<-function(x){x^exponent}
    shell_func<-function(x,exponent){
      exp_func(x)
    }

When `exp_func` is called inside `shell_func` it will look for
`exponent` first in its own execution environment, then when it doesn’t
find it there, it will look in its enclosing environment which is the
global environment. As such it doesn’t matter what value we set for the
`exponent` argument of `shell_func`

    exponent=2
    shell_func(x=2,exponent=10)

    ## [1] 4

We can even change the value of exponent in the global environment and
change the output of the function.

    exponent=4
    shell_func(x=2,exponent=10)

    ## [1] 16

If we want a function to inherit a value from the environment where it
was called we need to do via setting variables of the function, as we do
with `x`.

## Lazy loading

Even if you’re careful with all your environments you can still get
caught out by when R loads in values. R does something called **Lazy
Loading** which means it only loads in values when it absolutely
requires them. This can lead to some weird cases as seen in the example
below. To make everything explicit, we will clear our working
environment before doing anything.

    #Clear global environment
    rm(list=ls())

    exp_func_creator<-function(exponent){
      exp_func<-function(x){x^exponent}
      return(exp_func)
    }
    outer_exponential=4
    quad_func=exp_func_creator(exponent=outer_exponential)
    #update exponential value
    outer_exponential=3
    quad_func(x=2)

    ## [1] 8

Because the value of exponential in `quad_func` is taken from the value
of the global variable `outer_exponential` when the function is called,
it is set to the value of `3` instead of `4`. Now that the value has
been loaded, it is safe from future change.

    outer_exponential=10
    quad_func(x=2)

    ## [1] 8

If we want it to be set when we create the function we need to use the
`force` function to force R to load the value.

    exp_func_creator<-function(exponent){
      force(exponent)
      exp_func<-function(x){x^exponent}
      return(exp_func)
    }
    outer_exponential=4
    quad_func=exp_func_creator(exponent=outer_exponential)
    #update exponential value
    outer_exponential=3
    #This now uses the value of exponent when quad_func was created NOT when it 
    #was run
    quad_func(x=2)

    ## [1] 16

## Other Stupid Things

### 1: Why `x<-x` is not a useless line of code (sort of)

Remember when we called upon an object from the global environment
inside our function. This had the issue of meaning we can still change
how the function works even after it has been run. We can fix this with
the outrageously stupid command `exponential<-exponential`.

**Without `exponential<-exponential` **

    make_exp_func<-function(){
      f=function(x){x^exponential}
      return(f)
    }

    #Set value of exponential
    exponential<-2
    #Create our squaring function
    square_func<-make_exp_func()
    #works as expected
    square_func(3)

    ## [1] 9

    #Change value of exponential in the global environment
    exponential<-4
    #square_func output now changes
    square_func(3)

    ## [1] 81

**With `exponential<-exponential` **

    exponential<-2
    make_exp_func<-function(){
      exponential<-exponential
    f=function(x){x^exponential}
    return(f)
    }

    #Set value of exponential
    exponential<-2
    #Create our squaring function
    square_func<-make_exp_func()
    #works as expected
    square_func(3)

    ## [1] 9

    #Change value of exponential in the global environment
    exponential<-4
    #square_func output now still uses original exponential value
    square_func(3)

    ## [1] 9

### 2: How lazy loading makes seemingly innocuous commands can change the way your code runs

Lazy loading means that even doing the most innocuous of things changes
the way the function runs. Here is an example where running the `print`
function changes the output of these 2 chunks of code.

**Without the Print Function**

    exp_func_creator<-function(exponent){
      exp_func<-function(x){x^exponent}
      return(exp_func)
    }

**With the print function**

    exp_func_creator<-function(exponent){
      print(paste0("The value of exponent is =",exponent))
      exp_func<-function(x){x^exponent}
      return(exp_func)
    }
    #set initial exponential value
    outer_exponential=4
    #Create function 
    quad_func=exp_func_creator(outer_exponential)

    ## [1] "The value of exponent is =4"

    #update exponential value
    outer_exponential=3
    #function uses the value of outer_exponential from when it was created
    quad_func(2)

    ## [1] 16

You can think of `print` here working similarly to `force`. Imagine how
much of a nightmare debugging would be with this! Similar behaviours can
happen when checking the values of parameters in a debugger.

### 3: Lazy loading function parameters

Lazy loading is so lazy that it wont even check if all the variables of
a function have been specified.

    func_with_useless_var=function(x,y){
      x^2
    }

    func_with_useless_var(x=2)

    ## [1] 4

This function astoundingly works fine. If you want to error if `y` is
not given even `force(y)` won’t help. This is another case where doing
`y<-y` will get check if `y` exists (a more fancy and perhaps clearer
way of doing this is `get("y",inherit=FALSE)`)

### 4: The only time R doesn’t inherit from the parent environment (not that stupid)

Although specifying a parameter for a function doesn’t force you to even
use it, it does still have some affect. Below we have 2 functions which
are identical except one has `y` specified as a variable and the other
doesn’t.

    sum_func_with_y<-function(x,y){x+y}
    sum_func_without_y<-function(x){x+y}
    y=5
    sum_func_without_y(x=2)

    ## [1] 7

    sum_func_with_y(x=2)

    ## Error in sum_func_with_y(x = 2): argument "y" is missing, with no default

What we see here is that although a function doesn’t have to be given
all of it’s arguments, if an argument (`y` say) is used within a
function, the function will only look for that variable within the
execution environment and will not inherit from any parent environment.
## Some general rules

-   If at all possible all variables used within a function should be
    either specified as arguments to the function or created within the
    body of the function (the only exception to this is when some of
    those variables are variables of some subfunction you are creating
    within the main function.)
-   If you are writing a function to create a sub-function, all
    variables used to create this sub-function should be forced.

If these two rules are followed none of the above issues arise (although
I can’t guarantee you’ll never run into stupid stuff.)
