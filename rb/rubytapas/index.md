- ?y => 'y', is 1 character literal
- - can be useful to denote the intend that is only 1 character

- You can invoke process with `system 'ln -s file1 file2'`, as a string
- - Or you can pass a list
- - Using list has multiple advantage
- - - It forces ruby to run the command
- - - Using a string, the command is passed to shell
- - - Adding overhead
- - - Inconsistent on the OS
- - - May be subject to shell injection attacks
- - List of string is pass from ruby directly to the OS to be executed

- To use `forwardable`, require the lib, extend Forwardable, then call def_delegators
- - def_deletagors can be called on instance variable as well, e.g. :@account, vs :account
- - In fact, the delegator can be anything that is evaluable and can respond to the delegated method, e.g. def_delgator '@account.name', NOTE: no 's', and alias is accepted

- `allocate` is a method in Class, that will create an object of Class, but not call initialize hook on it

- We could turn `new` into a private method

```ruby
class Test
  class << self
    private :new
  end

  def self.instance
    @instance || new
  end
end

Test.instance # => #<... new instance>
Test.instance # => #<... same instance>

Test.new # => error, new method is private
```

- - This is basically how `Singleton` stdlib works
- - We could also override the new method, to provide more logic

```ruby
class Test
  def self.new
    ... do something ...
    super
  end

  def initialize
    ...
  end
end
```

- `Hash#fetch` instead of `[]`

# 9
- Generate symbol, %s, to_sym, :
- - Ruby runtime keeps a table of all the symbols, so that an unique id can be given to each
- - Generate a lot of symbols can leads to memory leaks

# 10
- `Dir::home` gives user's home
- - Depends on `HOME` environment variable to be set
- - Unless you pass the user login
- - `etc` stdlib can return the current user login for you
- - - `Etc::getlogin`, `config_file = File.join(Dir.home(user), ".zsh_history")`
- - This works on unix, but fails on windows

# 11 Message and methods
- Methods are not first class object in ruby
- Sending message is more flexible, while binding method to a specific implementation introduces a tighter coupling

# 12 fetch with defaults
- fetch can be supplied with a block, and is evaluated when the key is missing
- block can be used with defaults, or raise exception
- fetch differentiate between when value is _missing_ and value is _nil_ or _false_

```ruby
{}.fetch(:foo){:default} => :default
{foo: nil}.fetch(:foo){:default} => nil
{foo: false}.fetch(:foo){:default} => :false

# VS

{}[:foo] || :default => :default
{foo: nil}[:foo] || :default => :default
{foo: false}[:foo] || :default => :default
```

# 13 Singleton Objects

```ruby
LIVING_CELL = Object.new
class << LIVING_CELL
  def...
end

or

class << (LIVING_CELL = Object.new)
```

# 14 super
- `super` will call parent with all original arguments
- `super()` will force empty method, __BUT__ will pass the block to the parent regardless
- - To make sure the block is not passed, we must use `super(&nil)`

# fetch III
- You can call fetch on array, and will raise index error on non-existing indices
- `ENV` is a pseudo-hash, so fetch is available
- For accessing hash, we could do

```ruby
config.fetch(:database){{}}.fetch(:type){'sql'}
```

- - This way, we provide default values if the subtree is not available or do not exist
- fetch passes the missing key to the block
- - Below is interesting

```ruby
default = -> (key) do
  puts "#{key} is missing"
  gets
end

config.fetch(:missing_key, &default)
```

- fetch takes a second arguments, that is supplied if the key is missing
- - however, the second argument is always evaluated, can be a performance regression

# 16 super in Module
- In a module, (therefore in a class), we can check whether parent implement the same method

```ruby
module ...
  def hello
    if defined?(super) # the defined __operator__
      super
    else
      ... child class implementation ...
    end
  end
end
```

# 17 Pay it forwards (command query separation)
- the pattern tells us to not mix method that is a command and command that is a query
- therefore, instead of _returning_ the results, we can push the results forward, into a block for example

# 18 Subclassing array
- Use delegation instead of inheritance on core ruby data structures or classes

# 19 Pluggable selector
- __Pluggable selector pattern__ from Smalltalk best practice pattern by Kent Beck
- Basically, instead of calling a specific method, pass the method (message) to be called, by providing it as an argument
- Use `public_send`, making sure not to call private methods that could use a private implementation of a class
- "Any problem can be solve by introducing another layer of indirection, except the problem of having too much indirection"
- Might be overkill, but good to introduce less coupling

# 20 Struct
- Struct.new returns a class
- Attributes are accessible via symbol, string, and normal message call
- Struct classes includes Enumerable
- Struct#members iterate through all members
- Struct.new accepts constructor which is instance_eval'ed, so new method can be added

# 21 Domain Model Events
- Observer pattern

# 22 Inline Rescue
- expression `rescue` something
- - If expression fails, then rescue will return the following expression
- - Code smell

# 23 Tempfile
- Tempfile helps you to create tempfile in the tmp folder

# 24 Incidental Change
- Single responsibility principle

# 25 OpenStruct
- OStruct#new accepts a hash and allow calling hash attributes on the struct object
- Allow dynamic method definition
- OStruct misses some methods from struct, such as iterate through keys
- Performance issues
- Prototyping object models
- Stub object in test

# 26 FFI

# 27 Macros and Modules
- Macro: class level method that generates other methods, modules or classes

# 28 Macro and Modules
- Creating anonymous module using Module.new, you can implement to_s to give a signifiant name when introspecting the ancestors
- const_defined?, const_get, const_set

# 29 Redirecting Output
- Standard out
- STDOUT can be changed by Object.const_set(:STDOUT, fake_stdout)
- - It will reassign the constant, and a warning will be raised
- - Uses IO.pipe to create reader and writer pipe, but I'm still confused

# 30 Backtick
- Is an actual operator
- Can be overloaded
```ruby
alias old_backtick `
def `(lol)
  puts something
  old_backtick(lol)
end
```
- You can also override backtick in class... as a method, similar to `to_s`

# 31 Observer Variations
- More observer patterns variations...

# 32 Hash Default Blocks
- Default Hash.new blocks is invoked if hash is accessed with a missing key
- `default_proc` stores the proc, else nil is returned
- This leads to: which automatically allows arbitrary hash nesting

```ruby
Hash.new do |hash, missing_key|
  hash[missing_key] = Hash.new(&hash.default_proc)
end
```

# 33 Classes and Constants
- point = Class.new {}
- point.name => nil
- Point = point => Point.name => "Point"
- point.name => "Point"

# 34 Struct from Hash
- Struct#members for introspection is great, but keep it internal
- Do not call introspect method on object outside of the class, to reduce coupling, and swapping struct object out

# 35 Callable
- `call` method is useful

# 36 Blocks, Procs, and Lambdas
- Usual info about procs and lambdas
- Proc keep the context of execution, and will end the method which proc was called
- - The return act as if it was executed in that context
- `yield` calls the implicit block arguments

# 37 proc and threequals
- proc alias === to the call method

# 38 Caller specific Callback
- Use the default proc implicit arguments of method to handle edge cases for a library method

# 39 Gem Love part 1
- How to add a custom command to `gem`

# 40 Gradual Stiffening

```ruby
ruby -n -a -rjson \
-e 'BEGIN { $/="\r\n"; $;=/:\s*/; headers={} }' \
-e 'break if $F.size < 2' \
-e 'headers[$F[0]] = $F[1].chomp' \
-e 'END { puts JSON.pretty_generate(headers) }' \
< data.txt
```

# 41 String#scan
- scan takes a block, and is regex group aware, the block will yield different groups as arguments

# 42 Streaming
- #each method returns Enumerator, which can be a lazy iterable object over a collection

# 43 Exclusive Or
- ^ is the exclusive or operator, and the first expression should be a boolean
- - The easiest way to convert it to a boolean is to use the double ban `!!` operator

# 44 #one?
- [42, nil, 'banana'].one? => false
- [42, nil, nil].one? => true
- [].one? => false
- [1].one? => true
- [45, 43, 90].one?(:odd?)

# 45 Hash Default Values
- The hash constructor with a default value always returns the _same_ object
- - Hence an array will always be the same array returned

# 46 Gem Love Part 2
- Shellwords => Manipulates strings like the UNIX Bourne shell
- Parsing, or escaping strings according to shell rules
- Interesting integration tests
- In memory Sqlite database using Datamapper library

# 47 FFI Part 2 Smoke Test
- Smoke test: high level test that uses to verify that nothing is horribly broken, or produce black smoke

# 48 Memoize
- Macro for a memoizable method

# 49 Utility Function
- Creates module functions for the named methods
- These functions may be called with the module as a receiver, and also become available as instance methods to classes that mix in the module

```ruby
module Utils
  module_function
  def hi
    puts 'hi'
  end
end

Utils::hi
class Test
  include Utils
  def method
    hi
  end
end
```

- Utils => pronounced as 'U-deul'

# 50 Include Namespace

```ruby
module A
  def func1
  end

  module B
    include A
  end
end
```

# 51 FFI Part 3

# 52 When to Stop Mocking
- Only mock what you own
- Adapter interface actually is poor for mocking
- - Its better to use integration style tests to test on real data

# 53 Selectively Run Tests
- Some command line arguments to run tests by categories or by names for minitest and rspec

# 54 FFI Part 4
# Runnable Library
- `__FILE__` filename that is running the current code
- - If `require`ed, then its the full qualify path that will be printed out
- - If passed in arg, then its the name that will be passed that is printed

```ruby
ruby foo.rb => puts __FILE__ => foo.rb
ruby ./foo.rb => puts __FILE__ => ./foo.rb
```

- $PROGRAMNAME or $0, a lot of time $PROGRAMNAME is just the filename, except if the file is `require`ed

# 56 xmpfilters
# 57 FFI Part 5

# 58 ARGF
- Special objects that behaves like an IO readonly IO object
- If its files name, try to read and act as if the multiple files concatenated into a single giant file
- If no files, then act as STDIN

# 59 Enumerator
- Takes any method that yield to a block, and turns it into a lazy iterable objects which contains all the handy methods from Enumerable
- #to_enum turns a method into Enumerator
- Enumerator#with_index

# 60 Ascend
- Code explains better

```ruby
p = Pathname.new("/usr/local/bin")
ascender = p.to_enum(:ascend)
ascender.to_a => [<#Pathname>'/usr/local/bin',
                  <#Pathname>'/usr/local',
                  <#Pathname>'/usr',
                  <#Pathname>'/']
```

# 61 Fiber
- Using fiber to implement an Enumerator

# 64 Yield or Enumerate
- __callee__ special variable is always the name of the current calling method
- `return to_enum(__callee__) unless block_given?` is a special way of building an enumerator if no block is given, else call and the block and yield the value

# 66 Caching an API
- Using a simple hash to act as a cache

# 67 Moneta

# 68 Display Builder
- Its like a builder/renderer pattern
- Tell Don't Ask pattern, the model tell renderer about its attributes instead of the renderer asking for attributes
- Design pattern

# 69 Gem Love Part 4
-

# 70 Break keyword
- break can be used to terminate the yield
- break break also respects the ensure clause in a method, similar to exception

# 71 Break with a value
- break can take an option arguments
- if break is provided an argument, then it effectively override the return values of the method

```ruby
result = f.lines.detect do |line|
  break "Not found" if f.lineno >= 100
  line =~ /banana/
end
```

# 72 Tail Part 1 Random Access
- File#tell current reading cursor offset in random read
- File#seek jumps to offset wrt to the beginning of the file
- File#seek(byte, IO::SEEK_CUR), move x byte relatively to the current position of the cursor
- File#seek(0, IO::SEEK_END), move x byte wrt to the end of the file
- Negative argument move back from the end of the file

# 73 Tail Part 2 Do-While
- do-while in Ruby is expressed as `begin ... end while cond`

# 74 Tail Part 3 rindex
- `string#rindex` is like index, but starts from the end
- Regex is okay too

# 75 Tail Part 4 copy_stream
- TIL: `yes` unix command
- `copy_stream` is invoked with 2 IO objects, and will copy data from one stream to another
- Efficient
- Respect offset of the source

# 76 Tail Part 5
# 77 Tail Part 6
# 78 Tail Part 7 Cooperating Objects
- Refactors

# 79 Concat
- Use `Array#concat` to append objects to array
- `<<`, `+=`, have their own problems which is not desirable
- `Array#concat` is also more efficient

# 80 Splat basic
- splat is a form of destructuring
- `x,y,z = *[1,2,3]`, btw, the splat is explicit but in this case can be omitted
- `*x,y,z = [1,2,3,4]` => [1,2], 3, 4

# 81 Implicit splat
- Occurs when the array is alone on the right side of the assignment
- Set can also be splatted, so using a splat is sort of 'duck type safe' than the implicit one
- A single variable on the left side will collapse everything on the right side implicitly, `x = 1,2,3`

# 82 Inline assignment
- `while chunk = file.read`
- Essentially, its when you do assignment and boolean check in a single expression

# 83 Custom splat
- In ruby an object is considered to be an _array like_ if it respond to `to_ary`
- Therefore, the splat operator and implicit splat can be used when a custom object implements the `to_ary` method
- `to_ary` is used for implicit conversions, while `to_a` is used for explicit conversions
- Splat is explicit, parallel assignment is implicit

# 84 Splat group
- Essentially, we can nest destructure arrays on the left side of an assignment expression, like so

```ruby
expr = [:*, 3, [:+, 4, 9]]
op, f1, (inner_op, t1, t2) = expr
inner_op => :+
t1 => 4
t2 => 9
```

- We can also use the splat operator further

```ruby
expr = [:*, 3, [:+, 4, 9, 9, 2, 1, 2]]
op, f1, (inner_op, *ts) = expr # ts will absorb all the values of the second operation into an array
inner_op => :+
ts => [4, 9, 9, 2, 1, 2]
```

- This can be useful for hash

```ruby
h = { :a => 1, :b => 2 }
h.each_with_index do |(key, value), index| # Here the key and value will be splatted
  puts key, value, index
end
```

# 85 Ignore arguments
- `_`

# 86 Naked splat
- The naked splat basically says, _I dont care what arguments this method is taking_
- For example, when subclassing existing class, you want to completely have not to deal with the arguments of the original super class

```ruby
class CachedHTTP < Net::HTTP
  def initialize(*) # This will ignore all arguments
    @cache = {}
    super
  end

  def initialize(*args) # This will put all arguments into args, but args is never used
    @cache = {}
    super
  end

  def initialize(arg1, arg2, arg3) # This means we have to manually maintain all the arguments
    @cache = {}
    super
  end
end
```

# 87 Naming: Head Count
- How to better naming things, changed from _attendees_ to _headcount_

# 88 Gem love: Part 5
# 89 Coincidental duplication
- DRY
> Every piece of knowledge must have a single, unambiguous, authoritative representation within a system

- Given a code that handle error made from a REST client

```ruby
@logger.log ....
raise ServerError, response.code
```

- In this situation, if we were to subclassing the REST client and would like to provide a different way to handle the error instead of raising an exception, then raise server error would be labeled as coincidental duplication
- - Because although we are _sure_ to log the error everytime, we are _not_ sure to raise the error everytime, thus the concern is different, the logging and how to react to the error
- - We can split the logic into two different methods
- Its not easy to tell from real knowledge duplication and coincidental duplication

> The last thing we want, is to build real duplication into our code, out of the mis-sense of loyalty to our past decision

# 90 class << self
- Avid promotes that using self. is better than using class << self, because of the visual shifting of the code can be hard to see and realize when there are a lot of class methods

# 91 Ruby 2.0 Rebinding methods

```ruby
role.instance_method(method_name) # produces unbounded method
    .bind(self)                   # binds to a receiver object
    .call(*args, &block)          # calls the method on the object to which is was bound to
```

# 92 Coincidental duplication redux
# 93 Boolean
- In ruby, true is represented by a single instance of the TrueClass, and false is similar using FalseClass
- `TrueClass.ancestors` => TrueClass, Object, Kernel, BasicObject
- `FalseClass.ancestors` => FalseClass, Object, Kernel, BasicObject
- - Note that there are no common BooleanClass or a similar parent class
- - One reason might be that there are no real shared behaviour between TrueClass and FalseClass
- - These class contains only 3 similar methods, `&`, `|`, `^`, which behaves differently, when either true appears first or false appears first

# 94 Bang bang
- `!!` to convert to boolean, similar to js
- Normally, using the truthy or falsey value is enough in ruby
- However, look out when exporting these values into other system, i.e. in json, a value of truthy might _not_ be a boolean value

# 95 Gem Love Part 6
# 96 Gem Love Part 7
# 97 Gem Love Part 8
# 98 Gem Love Part 9

# 99 String subscript with regex
- str[regex, capture], `'083 Gem Love'[/^\d{3} (.+)/, 1]`, will returns the 1-first capture group

# 100 Screen-scrapting gateway
- Gateway 'Pattern of Enterprise Application Architecture'
> An object that encapsulates access to an external system or resource

- Class at the border of a system is best tested with integration tests which includes the external system as much as possible
- VCR gem is like a http cache, it will save request into yaml file, thus reduce roundtrip time
- Mechanize gem is like a page object, but automatically build by the gem
- URI.join can be used to join url parts with domain

# 101 Intention revealing message
- We sometime write code that do not convay enough information about the reason why the code is build this way
- For example, in the DPD website scraping, the array was used as [0..-2], because we know that the last rows is actually a button and not a show episode, we also did a reverse on the array because we inspected the HTML elements and realize that the rows are sorted in a decreasing order
- People who reads our code would have no idea that this infact is obtained after analysing the HTML layout
- We could add comments, but comments are easily forgotten, and so they become 'lies waiting to happen'
- We should use the Intention Revealing Message pattern (Kent Beck)
- Solution is to actually add methods with names that reveals the intent of the code that is been performed
- This might increase the size of code

# 102 Gem Love 10
# 103 Gem Love 11

# 104 Parsing Time
- Time.parse
- require 'date', and use Date.strptime, which contains the string to parse and the format of the time
- It doesnt actually uses the unix strptime, and ruby provide its own implementation
- The documentation to use is Time.strftime, which is used to convert time to string
- Datetime assums that the time to be parsed is in UTC
- There is an obscure DateTime.\_strptime, which does not parse the time into time format, but into a hash which contains the time part of the time that was just parsed
- We then can use Time.local, to actually get a time object
- Chronic gem can be used to parse the time, it almost no require formatting string, and can guess the time format

# 105 Checking for a terminal
- $stdout.tty? can be used to ask if the current standard output is a terminal

# 106 Class accessors
- Use self.attribute for class level accessor
- In rails, you can use the `cattr_accessor`, you can use a block which would generate the default
- They also generate convenient instance level accessor that refers to the class attributes

# 107 String subscript assignment
- Examples of string subscript assignment

```ruby
str[/^\d{3} (.*)/, 2] = "How cool is this?" => 107 How cool is this?
str[/^(?<number>\d{3}) (?<name>.*)/, :name] => "It's crazy cool!" => 107 It's crazy cool!
```

# 108 The trouble with nil
- Nil is the default return value for a hash if the key is not found, but it doesn't means that the key was not found, because the value might have been nil { a: nil }
- Empty method returns nil by default
- If a conditional statement returns false and there is no else block, then nil is returned by default
- If a case statement has no match, nil is returned
- If a local variable in a conditional branch that is not triggered, it will be set to nil by default
- Unset instance variable is always set to nil

# 109 SAX
- Simple API for XML
- Its actually a class that should implements a few hook methods that will be called when start, end, content of a xml element is found
- It is used with Nokogiri gem
- The net http can be used to get the web page chunk by chunk

# 110 Catch and throw
- In ruby, you can use the catch(:symbol) with throw(:symbol) to control code flow
- This removes the need to raise exception and rescue exception

# 111 Symbol placeholder

```ruby
options.fetch(:credentials) { :credentials_not_found }
```

- This will returns :credentials_not_found if there is no credential, which means that further call on fetch on the symbol will fail
- It throws an error with a message telling us where the problem might be originated using a text search

# 112 Special case (PoEEA)
- In a rails application for example, we keep using `current_user` to check whether there is an user, or the user exist; instead of doing this, it would be better to create a `GuestUser`, so we can substitute this user in for the current_user when the current user is _not_ logged in or known
- This way, we can eliminate a lot of special case check codes
- What we did was to find a special case in our code, and represented this special case with an object in our code

# 113 p
- Using puts is not showing the real representation of the object, for instance, p array will print the array on each line, in a debug session
- Appending .inspect to the object with puts is better
- p is the shortcut
- p returns the first value passed to it, while puts returns nil

# 114 NullObject
# 115 pp
- pp is a stdlib, can be required
- pp stands for pretty print, its awesome for printing long line, such as large hash
- pp gives the `#pretty_inspect`

# 116 Extract Command Object
- Avdi is demonstrating live how to refactor a code from a gem that he had

# 117 Admin Session Object
- Refactor the DPD code with a AdminSession which helps login into the DPD site

# 118 Even and Odd
- In ruby, the idiomatic way of checking whether a numeric number is even or odd, is to send the message `even?` and `odd?`

# 119 Intention revealing arguments
- Invoke method and write method which reveals the intention of the arguments
- Use arguments options to indicate usage of the argument passed
- If the option hash is not supported, a symbol can be used for the truthy value, because symbol is always true in ruby, and for negative value use the bang symbol style to negate the truthy value
- If the option hash is not supported, use a clear local variable name
- If the option hash is not supported, use a variable assignment in the method call, because the variable assigned will be passed as the value to the method i.e. method_call(value = false)

# 120 Outside in
- How many tests is enough? Am I writing too many, too few or just enough tests?
- The approach Avdi is showing us is outside in:
- Begin with accceptance test
- - Uses Open3#capture2 and capture2e to feed in stdin and capture stdout and stderr
- When using TDD, we want constant feed back from the tests
- - As soon as a test is high level that does not provide relevant information during development, we should maybe write a new tests, or drop deeper down a level to provide more specific tests
- In other words, how long has this test provide useful new information

# 121 Testing blocks
- How to test blocks? We can create a outside variable, and set this variable to true within the block and assert against the value of this variable when the block is completed
- We can pre-set this outside variable to be a symbol, it may provide clearer intent and role
- Multiple yield blocks, we can use an array to collect all the yielded values

# 122 Testing blocks with rspec

```ruby
expect{|probe| find_email_addressses(input, &probe)}.to yield_control
expect{|probe| find_email_addressses(input, &probe)}.not_to yield_control
expect{|probe| find_email_addressses(input, &probe)}.to yield_with_args("bob@gmail.com")
expect{|probe| find_email_addressses(input, &probe)}.to yield_successive_args("bob@gmail.com", "john@gmail.com")
```

# 123 Removing debug output
- Monkeypatch `p` in Kernel p

```ruby
module Kernel
  def p(*)
    raise Exception, "Debug output!"
  end
end
```

- Uses Kernel#caller to backtrack the call stack

# 124 Elixir
# 125 And/or
- ||, &&, and, or
- Symbolic operator and English operator
- The English operator actually has a much lower precendence than the Symbolic operator

```ruby
user = Struct.new(:name).new("Avdi")
user_name = user && user.name => "Avdi"
user_name = user and user.name => #<struct name="Avdi">
```

- This is because

```ruby
user_name = (user && user.name)
(user_name = user) and user.name
```

- More difference

```ruby
:x || :y && nil => :x
:x or :y and nil => nil
```

- This is because in symbolic operator, the && has higher precedence than ||
- While in English operator, the `and` and `or` has _same_ precedence
- The English operator actually are originated from Perl, and in Perl those English operator has distinct purpose, and they are used as _control flow_ operators
- In Perl, this code works

```ruby
chdir '/usr/spool/news' or die "Can't cd to spool: $!\n"
```

- In ruby this can be written as

```ruby
raise "Cant't read from STDIN" unless line = $stdin.gets
```

- This will not work

```ruby
line = $stdin.gets || raise "Can't read from STDIN"
SyntaxError: unexpected tSTRING_BEG, expecting keyword_do or '{' or '('
line = $stdin.gets || raise "Can't read from STDIN"
```

- This can be fixed by using the English or

```ruby
line = $stdin.gets or raise "Can't read from STDIN"
```

- Another example with `and`

```ruby
enable_notification = true
puts "Something happened" if enable_notifications => "Something happened"
enable_notifications && puts "Something happened"
SyntaxError: unexpected tSTRING_BEG, expecting keyword_do or '{' or '('

enable_notifications and puts "Something happened" => "Something happened"
```

- In summary, English and/or is good as control flow, we might wish to use them to group a series of expression that needs to be executed in order and depended on the success of the previous expression (and and and)
- Do not use English operator for the sake of using the return value of such

# 126 Queue
- Integer#upto, can replace range e.g. 1.upto(6), instead of (1..6)
- Using an array as queue is okay for single threaded, but is fucked up in multithreaded environment (duh...)
- `Thread#list` => A list of all thread that are either runnable or stopped
- `Thread#main` => The current main thread
- Instead, for multi-threaded access, use the `Queue` class in stdlib `thread`

> Queue: This class provides a way to synchronize communication between threads.

```ruby
require 'thread'
queue = Queue.new
queue << :a << :b << :c
1.upto(6) do |n|
  Thread.new do puts "Thread #{n}: Got #{queue.shift.inspect}" end
end

Thread.pass

queue.size => 0
queue.empty? => true
queue.num_waiting => 3

# Thread 6: Got :a
# Thread 4: Got :b
# Thread 1: Got :c
```

- Few things to note:
1. The thread access on the queue is totally arbitrary, and depends on ruby and OS thread scheduler
2. Even if a thread schedule earlier, it might runs later at the pop operation
3. The other threads are __blocked__, so this queue cannot pop anymore if there are no more item in it, thus other thread that tries to pop with __block__ and wait
4. Queue has aliases for other method, `shift` and `<<` (shovel), (pop and push (different than array, this is a __queue__ operation, not a __stack__ operation)), (deq and enq)
5. The queue causes any thread asking for the next item to block until the next one is available, when another item is pushed into the queue, the queue wakes up another thread and returns that item
6. This queue is thread aware, and is the building block for higher level concurrent programming pattern

# 127 Parallel Fibonacci
- Simple actor model parallel fibonacci processor

```ruby
require 'thread'
require 'benchmark'

module FibSolver
  def self.fib(scheduler_queue, my_queue)
    loop do
      scheduler_queue << [:ready, my_queue]
      message, *args = my_queue.pop # <<<<<<<< This will block until my_queue has something
      case message
      when :fib
        n, client_queue = args
        client_queue << [:answer, n, fib_calc(n), my_queue] # client_queue is the scheduler queue, because its the message sender
      when :shutdown
        break # End the thread, allow it to finish
      end
    end
  end

  # We ok with stack overflow here, as we will not feeding large fib number
  def self.fib_calc(n)
    case n
    when 0, 1 then 1
    else fib_calc(n-1) + fib_calc(n-2)
    end
  end
end

module Scheduler
  def self.run(num_threads, mod, meth, to_calculate)
    my_queue = Queue.new
    threads = (1..num_threads).map {
      thread_queue = Queue.new
      thread = Thread.new do
        mod.public_send(meth, my_queue, thread_queue)
      end
      { thread: thread, queue: thread_queue }
    }

    schedule_threads(threads, to_calculate, my_queue)
  end

  def self.schedule_threads(threads, to_calculate, my_queue)
    results = []
    loop do
      message, *args = my_queue.pop # This is the scheduler thread popped
      case message
      when :ready
        thread_queue = args
        if to_calculate.size > 0
          next_job = to_calculate.shift
          thread_queue << [:fib, next_job, my_queue]
        else
          thread_queue << [:shutdown]
          if threads.size > 1
            threads.delete_if { |t| t[:queue] == thread_queue }
          else
            return results.sort { |r1,r2| r1[:number] <=> r2[:number] }
          end
        end
      when :answer
        number, result, _ = args
        results << { number: number, result: result }
      end
    end
  end
end

to_process = [ 27, 33, 23, 54, 45 ... ]

Benchmark.bm(3) do |b|
  (1..10).each do |num_threads|
    b.report("#{num_threads}:") {
      Scheduler.run(num_threads, FibSolver, :fib, to_process.dup)
    }
  end
end
```

- Few notes, in Ruby mri, the results is slow due to GIL, so no performance gain on adding thread
- This problem is purely CPU bound

> A program is CPU bound if it would go faster if the CPU were faster, i.e. it spends the majority of its time simply using the CPU (doing calculations). A program that computes new digits of π will typically be CPU-bound, it's just crunching numbers.

> A program is I/O bound if it would go faster if the I/O subsystem was faster. Which exact I/O system is meant can vary; I typically associate it with disk. A program that looks through a huge file for some data will often be I/O bound, since the bottleneck is then the reading of the data from disk.

- Using rubinius is faster than MRI, mri was about 15-17s, rbx was 4-6s and jruby as 2-3s (which was similar to Elixir)

# 128 Enumerable queue
- The `Queue` class does not implement the _each_ method and nor includes the _Enumerable_ module
- To allow queue to be enumerable

```ruby
require 'thread'
q = Queue.new
eq = Enumerator.new do |y|
  loop do
    y << q.shift
  end
end

doubler = Thread.new do
  eq.each do |n|
    break if :stop == n
    puts n + n
  end
end

q << 1
q << 2
q << 3
q << :stop

doubler.join
```

- However, at this point, the enumerator has no idea about its current size
- To remedy this, we can add a proc (callable object) to the enumerator constructor
- `Enumerator.new(-> {q.size}) { ... }`

# 129 Rake
# 130 Rake File Lists

```ruby
files = Rake::FileList["**/*.md", "**/*.markdown"]
files.exclude("~*")
files.exclude(/^scratch\//)
files.exclude { |f| `git ls-files #{f}`.empty? }
puts files
```

# 131 Rake Rules
- rake --trace shows debugging breadcrumb
- rake -P shows list of prerequisites
- Rake.application.options.trace_rules = true shows tracing information about rules in the rake file

# 132 Rake Pathmap
- Pathmap is really awesome for files and paths manipulation, use it!

# 133 Rake file operations
- `directory` tasks will create the directory if it is missing
- `mkdir_p`, `rm_rf` are valid ruby file operations
- - Since these are ruby methods, we can pass file or filelists without needs for string interpolation
- - They are also sensitive to rake's quiet flag __rake -q__
- For a fulllist `ri FileUtils` ruby's stdlib

# 134 Rake clean
- require 'rake/clean' library
- Use `CLEAN` and `CLOBBER` as a global constant
- Using rake -T, 2 new tasks will be created, which will clean up CLEAN files or CLOBBER files

# 135 Rake multitask
- You can create/run tasks in parallel by doing either `rake -m`, use `multitask` instead of plain `task`
- You can also specify the degree of parallel using the `rake -j`

# 136 Dead thread
- In ruby, a thread that runs into an exception is swallowed and no error is outputted (silent failure)
- Using `Thread.abort_on_exception = true` or `$DEBUG = true`, or in console `ruby -d`, this does:

> When set to true, all threads will abort (the process will exit(0)) if an exception is raised in any thread.

- When the thread dies, there is no message indicating that the thread has died, even if we does `t.join`
- Use `SizedQueue` if we want to set an upper limit to how many elements the queue can accomodate
- This will trigger an exception in our example, since the consumer dies only after 1 pop, producer is waiting queue to be popped, and main thread is waiting on 2 other threads, since all thread is either dead or waiting, ruby infers that this program will wait forever, and raise a deadlock error
- We can include the `timeout` stdlib, and wrap the code to be executed within a `Timeout#timeout` closure

```
Timeout.timeout(time_in_seconds) {}
```

- __Warning__: Timeout library can introduce bugs prior to mri ruby 2.0 in multi-threaded code, see episode 143

> One of the biggest challenge in multi-threaded programming is the likelyhood of silent failure. If you want to write concurrent program, its better to be always looking at ways to make code fails quickly and loudly if something unanticipated happens

# 137 Mutex
- The following code simulate ruby's `Queue` class, however, there is a bug
```
class MyQueue
  def initialize
    @items = []
  end

  def push(obj)
    @items.push(obj)
  end

  def pop
    # This is a busy way, its CPU intensive
    # Also there is no restriction access to the critical section, therefore, this code will be buggy

    while(@items.empty?)
      # do nothing
    end
    puts "Pop goes the object!"
    @items.shift
  end
end
```

- We can use a `Thread#exclusive` around the critical section, this garantes that the critical section can be access by one thread at all time
- - However, `Thread#exclusive` is limited as throughout the __whole program__, one `Thread#exclusive` can be run
- - Some ruby internal also uses `Thread#exclusive`
- A modified version of the code using a mutex

> Mutex is short for mutual-exclusion, it gives us a way to give threads exclusive access to segment of code in a granular way

```
class MyQueue
  def initialize
    @items = []
    @lock = Mutex.new
  end

  def push(obj)
    @items.push(obj)
  end

  def pop
    @lock.synchronize do
      while(@items.empty?)
        # do nothing
      end
      puts "Pop goes the object!"
      @items.shift
    end
  end
end
```

- In ruby 2.0, all what `Thread#exclusive` do is to call `Mutex#synchronize` on a global mutex

# 138 Condition value
- The _busy wait_ loop on the pop operation is extremely inefficient and CPU intensive
- When a thread release a resource, it signals the conditional variable, which then wakes up the next thread that is in queue
- - This is an analogy to taking a number at a department store waiting to be served, when the servant (resource) is ready, it will call the next number in the queue

```
require 'thread'

class MyQueue
  def initialize
    @items = []
    @lock = Mutex.new
    @item_avaiable = ConditionalVariable.new
  end

  def push(obj)
    @items.push(obj)
    # We signal the condition, if there are any thread waiting to pop an item off,
    # this will wake one and only one thread
    @item_avaialble.signal
  end

  def pop
    @lock.synchronize do
      # A conditional variable always works with a mutex
      # Passing the mutex in allows the conditional variable to release
      # the mutex early, so that other threads can enters to this critical section
      # If we didn't release the lock earlier, then other thread would not be able
      # to run into this critical section as the synchronize will not be allowed
      # and the thread would not be able to put themself in line within the conditional
      # variable, so to speak

      # We only wait if the item is empty, if there is an item, then there is no need to wait
      # as there is already an item ready to be used
      @item_available.wait(@lock) if @items.empty?
      puts "Pop goes the object!"
      @items.shift
    end
  end
end
```

- This version will runs faster, as the thread will not occupy the CPU with endless looping

# 139 Timed queue
- Thread can die, and hang indefinitely, a good way to ensure that errors like this are detected quickly is to never allow thread to wait indefinitely for something
- Instead, anytime when thread is waiting for a resource, we should provide a timeout for the wait
- The condtional variable `#wait` can specify a timeout, if the conditional variable does not signal within the time, the wait call will timeout and returns
- - __Warning__: Make sure to check the value of the resource after the wait, because it could be timed out, or a normal resource acquire
- To give our thread the proper time for shutdown (a bit like cancellationtoken in C#)
-- Modify the queue class first

```
require 'thread'

$shutdown = false
trap('INT') { $shutdown = true }

class MyQueue
  def initialize
    @lock = Mutex.new
    @item_available = ConditionVariable.new
    @items = []
  end

  # Note this code is wrong and will be corrected in episode 140
  def push(obj)
    @items.push(obj)
    @item_availble.signal
  end

  def pop(timeout = :never, &timeout_policy)
    # Symbolic parameter for clarity
    cv_timeout = timeout == :never ? nil : timeout
    timeout_policy ||= -> {nil}
    @lock.synchronize do
      @item_avaiable.wait(@lock, cv_timeout) if @items.empty?
      if @items.any?
        @items.shift
      else
        timeout_policy.call # Allow client code to handle timeout and when there is no item in the queue to be popped
      end
    end
  end
end
```

-- Modify our consumer and producer to take into account of the thread timeout

```
q = MyQueue.new

producer = Thread.new do
  i = 0
  until($shutdown) do
    widget = "widget#{i+=1}"
    puts "producing #{widget}"
    q.push(widget)
    sleep 1 # lets wait a second between producing so that our consumer can timeout
  end
  puts "Producer is shutting down"
end

def consumer(thread, q)
  until($shutdown) do
    widget = q.pop(0.5)
    if widget
      puts "#{thread} consumer #{widget}"
    else
      puts "#{thread} timed out waiting for a widget"
    end
  end
  puts "#{thread} shutting down"
end

threads = (1..3).map { |n| Thread.new { consume("thread #{n}", q) }}
threads.each(&:join)
```

- Its due to the presence of the timeout that allows the thread to periodically stop waiting, and check whether it needs to shutdown or not


# 140 Threads are hard
- A demonstration of why multithread programming is hard, and that there is a bug in the original implementation of the queue

> next inside a block returns from the block. break inside a block returns from the function that yielded to the block. For each this means that break exits the loop and next jumps to the next iteration of the loop (thus the names). You can return values with next value and break value.

- Array in mri ruby are implemented in C which is run on the GIL
- Array in other ruby implementation are not thread safe even for a single operation

# 141 Bounded queue

# 142 Infinity
- Infinity is a __Bengin value__

> A __bengin value__ is a value that signal a special case, while been completly compatible with the code that handles the ordinary case

# 143 Thread interrupt

> The point of using __ensure__ clauses, is to ensure that cleanup that is necessary to retain consistency will always be performed, even if an exception is raised

- `Thread#kill`, and `Thread#abort` becomes a problem, they can cause an exception to be raised or termination to occur at literally any point in the target thread
- http://stackoverflow.com/questions/2191632/begin-rescue-and-ensure-in-ruby

```
begin
  # something which might raise an exception
rescue SomeExceptionClass => some_variable
  # code that deals with some exception
rescue SomeOtherException => some_other_variable
  # code that deals with some other exception
else
  # code that runs only if *no* exception was raised
ensure
  # ensure that this code always runs, no matter what
end
```

- `Thread#kill` and `Thread#raise` are the main issue why `Timeout#timeout` is an unsafe method to run
-- Because within the implementation, ruby use `Thread#raise` which is sure to interrupt thread at some random point
-- In ruby 2.0, there is a new `Thread#handle_interrupt`, which indicate how exception should be handled
-- Therefore, when calling `Thread#raise`, the handle interrupt will delay the raised exception until the setting on the type of the exception is met
--- The settings are :never, :immediate, and :on_blocking

# 144 Bulk generation
- 200.times.map { SecureRandom.hex(8) }
- 200.times.collect { SecureRandom.hex(8) }

# 145 Thread pool
-


# 148 Rake invoke
- To programmatically load rake tasks

```
require 'rake'
Rake.application.init
Rake.application.load_rakefile # Load rakefile based on the current directory
Rake::Task['my_task'].invoke
```

- `Task#execute` will rerun task, instead of been lazy and runs the task only once
-- However, `execute` will not runs any prerequisites of the task
- Can load rake file with `Rake::load_rakefile`
- Can run namespace with `Rake::Task['my_namespac:my_taskname']`

# 149 Sum
- Sum integer using `Enumerable#reduce`
- `data.reduce(0, &:+)`
- `data.reduce(0, :+)` reduce is smart about the second argument and will automatically call to_proc on the second argument if a symbol is passed to it
- `data.reduce(:+)` if we don't provide a seed, then reduce uses the first element as the seed

# 150 Stats
- min, max, sum, avg, median and std

# 151 Sleep
- When we want to make the ruby program or the current thread of the program stop and do nothing, we use the sleep method

# 152
# 153
# 154
# 155

# 156 Array.new
- To read: the best ruby quiz
- Use `Array.new(number) { to_do }` to generate arrays, instead of (1..2).map, or 3.times.collect
- The reason is map is more for _transform_, than to _generate_

# 157

# 158 Constant lookup scope
- Using `Module.nesting` we can look at the constant lookup path
- If you define a constant at the top level, then this constant is added to the `Object` constant, which will __not__ be shown when querying `Module.nesting`
- Ruby constant lookup is only lexical
- Shorthand module declaration does not create intermediate module, for clarification, shorthand is like so: `Planets::Jupiter`
-- One of the reason might be ruby have no idea whether to create the intermediate constant as class or module
- The take away is this

> There are no reason to ever use the shorthand declaration

# 159 Array set operations
- `Array#|` is like (Array1 + Array2).uniq!
- `Array#&`, `Array#-`
- Use it to discover methods `URI::HTTP.instance_methods - Object.instance_methods`
- __Warning__: These operation creates new objects
- Use `require 'set'`

# 160 Reduce redux
- `[].reduce(:+)` can be dangerous, as it would return nil, instead of 0 from `[].reduce(0, :+)`
-- This is mostly due to dynamic ruby language, who cannot figure out what kind of object is in the array
- Reduce can be used to traverse data-structure by combining it with reduce + fetch

# 161 Thread local variable
- A variable that is local to a single thread
- We can set thread local variable by treating the thread object as a hash `thread[:key] = value`
- We can unset using `thread[:key] = nil`
- Since ruby 1.9, they are also fiber local variable as well
- _Tips_: define thread local variable with namespace or prefix, so it doesnt conflict with another variable

# 162 PStore
- The pstore library can binary save ruby objects, persistance library
- Does deep serialization for storage in a file
- pstore allows one 1 writing transaction, and multiple reading transaction is fine
- This applies to multi-thread, because pstore uses OS's file locking mechanism to update 1 file at a time
- Every transaction reads the entire content in, and writes it all back (inefficient for large content, locks is on the file)

# 163 yaml store
- yaml store has the same api as the pstore

# 164 mapper
- Use a lazy enumerator sometimes is better because the enumerator is lazy

```
def all
  each_posts.to_a
end

def each_posts
  return to_enum(__callee__) unless block_given? # see 64
end
```

# 165 Refactor tapas::queue

# 166 Not implemented
- In ruby, NotImplementedError, it does not inherit from standard error, generic rescue block will not catch it

# 167 Debugging gems
- We could download or unpack the source code of the gem and altering the load path or the gem setting to use the local source code (many steps)
- Gems can be installed in multiple places
-- System (/usr/local/lib/site_ruby)
-- User (~/.gem)
-- Version-specific (~/.rvm/gems/ruby-2.0.0)
-- Gemset-specific (~/.rvm/gems/ruby-2.0.0@my_project)
-- Project local (my_project/vendor/)

- Use `bundle show gem_name` to show which gem and path is current project using
- Use `bundle open gem_name` to open gem in default editor
- To local gem installation in __none-bundler__ managed projects, use `gem-open`
-- Install gem-open (gem install gem-open), then use `gem open gem_name`
- 'gem pristine gem_name' will use the local gem package cache to restore the gem to its original unmodified states

# 168 Enumerable internals
- Feature hosts outline how c code in ruby implements the enumerable

# 169

# 170 Hash merge
- If you provide a block to a `Hash#merge` method, it will call - block with the key, and the conflicted left and right value, then you can flesh out the logic for on to handle the duplicate

# 171 puts
- puts => put-string, similar to the C-method
- puts will implicitly call the `Kernel#puts` method
- puts accept multi-variables, flatten arrays, which means often, loops for putting arrays is superfluous, we can just call `puts arr`
- If the string to puts contains a line-ending '\n', then puts will not add a new one
- puts actually add the new line after the original line is printed

```
puts 'the line'
puts '\n' unless 'the line' ends with \n
```

- This is a reason why puts in multi-thread code might run onto each other
