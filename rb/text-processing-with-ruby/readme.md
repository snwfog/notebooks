- Read whole file into string `File#reads`

- `File#each` is alias for `File#each_line`, so all `map`, `Enumerable` methods can be used per line

- `Kernel#open` can be used instead of `File#open`

- `File#readlines` reads all line into an array (expensive)

- `File#read` takes argument to specify specific number of bytes to read (useful for fixed format files)

- `File#seek` can skip arbitrarly into the file, at **constant** (_O(1)_)
- - _Watchout_: For multi-byte UTF-8 encodings while using seek and read as they operate on byte

- `$stdin` and `$stdout` to write to in and out

- `!#/usr/bin/env ruby` to tell shell to use os ruby
