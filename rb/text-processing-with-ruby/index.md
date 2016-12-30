- Read whole file into string `File#reads`

- `File#each` is alias for `File#each_line`,
so all `map`, `Enumerable` methods can be used per line

- `Kernel#open` can be used instead of `File#open`

- `File#readlines` reads all line into
an array (expensive)

- `File#read` takes argument to specify specific
number of bytes to read (useful for fixed format files)

- `File#seek` can skip arbitrarly into the file,
at **constant** (_O(1)_)
- - _Watchout_: For multi-byte UTF-8 encodings
while using seek and read as they operate on byte

- `$stdin` and `$stdout` to write to in and out

- `!#/usr/bin/env ruby` to tell shell to use os ruby

- `-npe` runs some method with implicit object
set to `$_`
- - `-n` runs in loop
- - `-p` runs in loop + print `$_` by default
at the end of each line

- `BEGIN` block can be used to initialize
a shell one-liner

```
  printf "foo\nbar\nbaz\n" | \
  ruby -ne 'BEGIN { i = 1 }; puts "#{i} #{$_}"; i += 1'

  printf "foo\nbar\nbaz\n" | \
  ruby -ne 'i ||= 1; puts "#{i} #{$_}"; i += 1'
```

- `END` block can be used for the final

```
  printf "12\n76\n42" | \
  ruby -ne 'BEGIN { n = 0 }; n += $_.to_i; END { puts n }'
```

- `ARGF` responds to `each_line`, so its an
IO that is automatically created by ruby

```
# argf.rb
ARGF.each_line do |line|
  puts line
end

ruby argf.rb foo.txt bar.txt baz.txt
```

- If file do not exists, ruby will raise exception

- When no file is specified, ruby will read
from stdin

```
$stdin.each_line do |line|
puts line
end

$ printf "foo\nbar" | ruby argf.rb
foo
bar
```

- Note that if arguments is given to ruby,
then it will automatically reads from file

```
if ARGV.length > 0
  # Read from the files specified on the command line
else
  # Read from standard input
end
```

- `ARGF.lineno`, `ARGF.file`, `ARGF.file.lineno`,
`ARGF.filename`

- `ruby -i` can modify text in-place

```
ARGF.each do |line|
  puts line.gsub("tranquillity", "tranquility")
end
$ echo "tranquillity" > file.txt
$ ruby -i argf-sub.rb file.txt
```

- `-i` takes argument, and will backup the
to-be-modified file

```
$ echo "tranquillity" > file.txt
$ ruby -i'.bak' argf-sub.rb file.txt
$ cat file.txt
tranquility
$ cat file.txt.bak
tranquillity
```

- We can modify `ARGV` _before_ reading from `ARGF`
to remove those arguments that might not be a
proper file, to remember _ARGV_ is mutable

- Use `shift` or `push` to add arguments to `ARGV`
from left and right

- `$/` is ruby's _input record separator_,
by default its `\n`, e.g. `each`

- `$;` is ruby _input record field separator_,
e.g. `split`

- `-F` can set the `$;` values in as option

```
$ ruby -F: -e 'p $;'
/:/
```

- `-a` the auto-split option, ruby will automatically
populate the `$F` variable with the content of
`$_.split`, it will automatically split every
line of input

```
$ ruby -F'\t' -ane 'puts $F[1]' shopping.tsv
```

- `require 'csv'` and use `CSV`, is very similar to
an ordinary IO object, with `::read`, `::foreach`, etc

- `/imox`, ignore case, multiline, interpolate
only once, extended (allow white space
and using # for comments)

- `=~` the regexp match operator, if matches
return truthy, else, return falsey

- `match` method is the same thing, and exists
on String and Regexp

- `match` accepts a block which will be executed
only when there is something that matches, thus
avoid writing checks code

- `$~` contains the entire matchdata of the last
match, same as what is returned by `match`

- `$1, $2, etc, to $9`, contains values of
capture group from last match

- `$&` contains the whole match as a string, it
is the same as accessing the match[0] element of
the matchdata object

- `$+` contains the final capture group of the most
recent match, similar to `$~.captures.last`, if
there is no match group then nil will be returned

- `$backtick` contains the first prematch group

- `$apostroph` contains the last prematch group

- `scan`, `sub`, `gsub`

- `gsub`, `\1, \2, etc` can be used to refer to
capture group of the regexp, you can also use
named capture group

```
puts discography.gsub(/^([0-9]{4}) - (.+)$/, '\2 (\1)')
puts discography.gsub(
  /^(?<year>[0-9]{4}) - (?<album>.+)$/,
  '\k<album> \k<year>'
)
```

- Difference between `STDOUT` and `$stdout` is the former
is a constant, so not redefinable, latter is just a global
variable, which can be changed

- TIL, 'CR' character move the line to the beginning of the
same line, and 'LF' line feed feeds a new line

```
1. Carriage return: It's a printer terminology meaning changing the print location to the beginning of current line. In computer world, it means return to the beginning of current line in most cases but stands for new line rarely.

2. Line feed: It's a printer terminology meaning advancing the paper one line. So Carriage return and Line feed are used together to start to print at the beginning of a new line. In computer world, it generally has the same meaning as newline.
```

- `$stderr` can be used as an out of band stream to communicate
while `$stdout` might be busy at printing something else. Also
using the `\r` to print to the same line is interesting

- Difference between `puts` and `print`, print is for outputting
records on a single line, and puts is for outputting whole
records
- - Both methods takes multiple arguments
- - After each argument, `print`, will output `$,`, the _output
field separator_, after final argument, output `$\`, the _output
record separator_
- - `puts` does the same, except `$\`, and `$,` is set
to be equal to `\n`
- - `$,` and `$\` is nil by default

- `puts` actually output the newline separator in a separated
operator, this leads when program with race condition,
multiple newlines are stucked together

```
5.times do
  fork do
    sleep 0.5
    puts "I'm in a new process!"
  end
end
Process.waitall
```

- The fix is actually simple, always include a `\n` at the end
of `puts`, internally, ruby checks whether the string contains
a `\n` or not, if it does, it will skip putting the newline

- `print`, `puts`, implicitly act on `$stdout` and _NOT_ `STDOUT`

- `StringIO` acts like an IO object, stdlib

- `IO.pipe`, returns read io and a write io

- `IO#sync` to true means that output will be flushed
immediately, otherwise, it will be buffered

- `open` can be used to open pipes in unix process

```
open("| sort | uniq -c | sort -rn | head -n 10", "w") do |sort|
  open("data/error_log") do |log|
    log.each_line do |line|
      if line =~ /^\[.+\] \[error\] (.*)$/
        sort.puts $1
      end
    end
  end
end
```

- Use `Tempfile` to create temporary files
- - Tempfile are created with absolutely unique file name
- - Tempfile are automatically deleted when outside of block
- - If handle manually creation and deletion, then its
advisable to `#close` and also `#unlink` the file handle
