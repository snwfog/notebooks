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
