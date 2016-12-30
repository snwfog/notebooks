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
