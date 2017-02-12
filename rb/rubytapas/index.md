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
```
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

```
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
```
{}.fetch(:foo){:default} => :default
{foo: nil}.fetch(:foo){:default} => nil
{foo: false}.fetch(:foo){:default} => :false

VS

{}[:foo] || :default => :default
{foo: nil}[:foo] || :default => :default
{foo: false}[:foo] || :default => :default
```

# 13 Singleton Objects
```
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
```
config.fetch(:database){{}}.fetch(:type){'sql'}
```
- - This way, we provide default values if the subtree is not available or do not exist
- fetch passes the missing key to the block
- - Below is interesting
```
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
```
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
- << Pluggable selector pattern >> from Smalltalk best practice pattern by Kent Beck
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
result = f.lines.detect do |line|
  break "Not found" if f.lineno >= 100
  line =~ /banana/
end
```

# 72 Tail Part 1 Random Access
- File current offset in random read File#tell
- File#seek jumps to offset wrt to the beginning of the file
- File#seek(byte, IO::SEEK_CUR), move x byte relatively to the current position of the cursor
- File#seek(0, IO::SEEK_END), move x byte wrt to the end of the file
- Negative argument move back from the end of the file

# 73 Tail Part 2 Do-While
- do-while in Ruby is expressed as `begin ... end while cond`

# 74 Tail Part 3 rindex
- `string#rindex` is like index, but starts from the end

# 75 Tail Part 4 copy_stream
- 
