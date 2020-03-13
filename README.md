# clean-code-julia

## Table of Contents
  1. [Introduction](#introduction)
  2. [Variables](#variables)
  3. [Functions](#functions)
  4. [Objects and Data Structures](#objects-and-data-structures)
  5. [Classes](#classes)
     1. [S: Single Responsibility Principle (SRP)](#single-responsibility-principle-srp)
     2. [O: Open/Closed Principle (OCP)](#openclosed-principle-ocp)
     3. [L: Liskov Substitution Principle (LSP)](#liskov-substitution-principle-lsp)
     4. [I: Interface Segregation Principle (ISP)](#interface-segregation-principle-isp)
     5. [D: Dependency Inversion Principle (DIP)](#dependency-inversion-principle-dip)
  6. [Don't repeat yourself (DRY)](#dont-repeat-yourself-dry)

## Introduction

Software engineering principles, from Robert C. Martin's book
[*Clean Code*](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882),
adapted for Julia. This is not a style guide. It's a guide to producing
readable, reusable, and refactorable software in Python.

Not every principle herein has to be strictly followed, and even fewer will be universally 
agreed upon. These are guidelines and nothing more, but they are ones codified over many 
years of collective experience by the authors of *Clean Code*.

Inspired from [clean-code-python](https://github.com/zedr/clean-code-python)

Targets Julia 1.3.1

## **Variables**
### Use meaningful and pronounceable variable names

**Bad:**
```julia
ymdstr = Dates.format(now(), "yyyy-mm-dd")
```

**Good**:
```julia
current_date = Dates.format(now(), "yyyy-mm-dd")
```
**[⬆ back to top](#table-of-contents)**

### Use the same vocabulary for the same type of variable

**Bad:**
Here we use three different names for the same underlying entity:
```julia
get_user_info()
get_client_data()
get_customer_record()
```

**Good**:
If the entity is the same, you should be consistent in referring to it in your functions:
```julia
get_user_info()
get_user_data()
get_user_record()
```

**Even better**
Julia has multiple dispatch. If it makes sense, package the related data together in a struct and 
define functions are called with this struct.

```julia
struct User
    foo::String
    bar::Int
end

get_info(user::User)
get_data(user::User)
get_record(user::User)
```

**[⬆ back to top](#table-of-contents)**

### Use searchable names
We will read more code than we will ever write. It's important that the code we do write is 
readable and searchable. By *not* naming variables that end up being meaningful for 
understanding our program, we hurt our readers.
Make your names searchable.

**Bad:**
```julia
# What the heck is 86400 for?
sleep(86400);
```

**Good**:
```julia
# Declare them in the global namespace for the module.
SECONDS_IN_A_DAY = 60 * 60 * 24

sleep(SECONDS_IN_A_DAY)
```
**[⬆ back to top](#table-of-contents)**

### Use explanatory variables
**Bad:**
```julia
address = "One Infinite Loop, Cupertino 95014"
city_zip_code_regex = r"^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$"
matches = match(city_zip_code_regex, address)

save_city_zip_code(matches[1], matches[2])
```

**Not bad**:

It's better, but we are still heavily dependent on regex.

```julia
address = "One Infinite Loop, Cupertino 95014"
city_zip_code_regex = r"^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$"
matches = match(city_zip_code_regex, address)
city, zip_code = matches.captures

save_city_zip_code(city, zip_code)
```

**Good**:

Decrease dependence on regex by naming subpatterns.
```julia
address = "One Infinite Loop, Cupertino 95014"
city_zip_code_regex = r"^[^,\\]+[,\\\s]+(?P<city>.+?)\s*(?P<zip_code>\d{5})?$"
matches = match(city_zip_code_regex, address)

save_city_zip_code(matches["city"], matches["zip_code"])
```
**[⬆ back to top](#table-of-contents)**

### Avoid Mental Mapping
Don’t force the reader of your code to translate what the variable means.
Explicit is better than implicit.

**Bad:**
```julia
seq = "Austin", "New York", "San Francisco"

for item in seq
    do_stuff()
    do_some_other_stuff()
    # ...
    # Wait, what's `item` for again?
    dispatch(item)
end
```

**Good**:
```julia
locations = "Austin", "New York", "San Francisco"

for location in locations
    do_stuff()
    do_some_other_stuff()
    dispatch(location)
end
```
**[⬆ back to top](#table-of-contents)**


### Don't add unneeded context

If your struct name tells you something, don't repeat that in your
variable name.

**Bad:**

```julia
struct Car
    car_make::String
    car_model::String
    car_color::String
end
```

**Good**:

```julia
struct Car
    make::String
    model::String
    color::String
end
```

**[⬆ back to top](#table-of-contents)**

### Use default arguments instead of short circuiting or conditionals

**Tricky**

Why write:

```julia
function create_micro_brewery(name=nothing)
    name = isnothing(name) ? "Hipster Brew Co." : name
    do_something_with_name(name)
    # etc.
end
```

... when you can specify a default argument instead? This also makes it clear that
you are expecting a string as the argument.

**Good**:

```julia
function create_micro_brewery(name="Hipster brew Co.")
    do_something_with_name(name)
    # etc.
end
```

**[⬆ back to top](#table-of-contents)**
## **Functions**
### Function arguments (2 or fewer ideally)
Limiting the amount of function parameters is incredibly important because it makes 
testing your function easier. Having more than three leads to a combinatorial explosion 
where you have to test tons of different cases with each separate argument.

Zero arguments is the ideal case. One or two arguments is ok, and three should be avoided. 
Anything more than that should be consolidated. Usually, if you have more than two 
arguments then your function is trying to do too much. In cases where it's not, most 
of the time a higher-level object will suffice as an argument.

**Bad:**
```julia
function create_menu(title, body, button_text, cancellable)
    # ...
end
```

**Good**:
```julia
struct MenuConfig
    title::String
    body::String
    button_text::String
    cancellable::Bool
    
    MenuConfig(;title, body, button_text, cancellable) = new(title, body, button_text, cancellable)
end

config = MenuConfig(
    title = "My Menu",
    body = "Something about my menu",
    button_text = "OK",
    cancellable = true,
)

function create_menu(config::MenuConfig)
    # ...
end
```

**Also good**

Use keyword arguments instead of positional arguments

```julia
function create_menu(;title, body, button_text, cancellable)
    # ...
end

create_menu(
    title = "My Menu",
    body = "Something about my menu",
    button_text = "OK",
    cancellable = true,
)
```


**[⬆ back to top](#table-of-contents)**

### Functions should do one thing
This is by far the most important rule in software engineering. When functions do more 
than one thing, they are harder to compose, test, and reason about. When you can isolate 
a function to just one action, they can be refactored easily and your code will read much 
cleaner. If you take nothing else away from this guide other than this, you'll be ahead 
of many developers.

**Bad:**
```julia

function email_clients(clients::Array{Client})
    # Filter active clients and send them an email.

    for client in clients
        if client.active:
            email(client)
        end
    end
end
```

**Good**:
```julia
function get_active_clients(clients::Array{Client})::Array{Client}
    # Filter active clients.

    filter(client->client.active, clients)
end

function email_clients(clients::Array{Client})
    # Send an email to a given list of clients.

    for client in clients
        email(client)
    end
end
```

**[⬆ back to top](#table-of-contents)**

### Function names should say what they do

**Bad:**

```julia
struct Email
    # ...
end

function handle(email::Email)
    # Do something...
end

message = Email()
# What is this supposed to do again?
handle(message)
```

**Good:**

```julia
struct Email
    # ...
end

function send(email::Email)
    # Send the email
end

message = Email()
send(message)
```

**[⬆ back to top](#table-of-contents)**

### Functions should only be one level of abstraction

When you have more than one level of abstraction, your function is usually doing too 
much. Splitting up functions leads to reusability and easier testing.

**Bad:**

```julia
function parse_better_js_alternative(code::String)
    regexes = [
        # ...
    ]

    statements = split(code)
    tokens = []

    for regex in regexes
        for statement in statements
            # ...
        end
    end

    ast = []
    for token in tokens
        # Lex.
    end

    for node in ast
        # Parse.
    end
end
```

**Good:**

```julia

REGEXES = [
   # ...
]


function parse_better_js_alternative(code::String)
    tokens = tokenize(code)
    syntax_tree = parse(tokens)

    for node in syntax_tree
        # Parse.
    end
end

function tokenize(code::String)
    statements = split(code)
    tokens = []

    for regex in REGEXES
        for statement in statements:
           # Append the statement to tokens.
        end
    end

    tokens
end


function parse(tokens::Array)::Array
    syntax_tree = []

    for token in tokens
        # Append the parsed token to the syntax tree.
    end

    syntax_tree
end
```

**[⬆ back to top](#table-of-contents)**

### Don't use flags as function parameters

Flags tell your user that this function does more than one thing. Functions 
should do one thing. Split your functions if they are following different code 
paths based on a boolean.

**Bad:**

```julia

function create_file(name::String, temp::Bool)
    if temp
        touch("./temp/$(name)")
    else:
        touch(name)
    end
```

**Good:**

```julia

function create_file(name::String)
    touch(name)
end

function create_temp_file(name::String)
    touch("./temp/$(name)")
end
```

**[⬆ back to top](#table-of-contents)**

### Avoid side effects

A function produces a side effect if it does anything other than take a value in 
and return another value or values. For example, a side effect could be writing 
to a file, modifying some global variable, or accidentally wiring all your money
to a stranger.

Now, you do need to have side effects in a program on occasion - for example, like
in the previous example, you might need to write to a file. In these cases, you
should centralize and indicate where you are incorporating side effects. Don't have
several functions and classes that write to a particular file - rather, have one
(and only one) service that does it.

The main point is to avoid common pitfalls like sharing state between objects
without any structure, using mutable data types that can be written to by anything,
or using an instance of a class, and not centralizing where your side effects occur.
If you can do this, you will be happier than the vast majority of other programmers.

**Bad:**

```julia
# This is a module-level name.
# It's good practice to define these as immutable values, such as a string.
# However...
name = "Ryan McDermott"

function split_into_first_and_last_name()
    # The use of the global keyword here is changing the meaning of the
    # the following line. This function is now mutating the module-level
    # state and introducing a side-effect!
    global name
    name = split(name)
end

split_into_first_and_last_name()

print(name)  # ['Ryan', 'McDermott']

# OK. It worked the first time, but what will happen if we call the
# function again?
```

**Good:**
```julia
function split_into_first_and_last_name(name::String)
    split(name)
end

name = "Ryan McDermott"
new_name = split_into_first_and_last_name(name)

print(name)  # "Ryan McDermott"
print(new_name)  # ["Ryan", "McDermott"]
```

**[⬆ back to top](#table-of-contents)**

## **Objects and Data Structures**

*TBD*

**[⬆ back to top](#table-of-contents)**

## **Classes**

### **Single Responsibility Principle (SRP)**
### **Open/Closed Principle (OCP)**
### **Liskov Substitution Principle (LSP)**
### **Interface Segregation Principle (ISP)**
### **Dependency Inversion Principle (DIP)**

*TBD*

**[⬆ back to top](#table-of-contents)**

## **Don't repeat yourself (DRY)**

*TBD*

**[⬆ back to top](#table-of-contents)**