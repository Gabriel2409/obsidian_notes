#linux

https://learnbyexample.github.io/learn_gnugrep_ripgrep/cover.html

grep: 
- default: `-G` (BRE = Basic Regular Expression)
- use ERE with `-E`
- `-P` for perl regexes (for ex for non greedy match)
- use `-F` for litteral interpretation

rg: 
- very siimilar to ERE (uses rust regex engine)
- no `-G`
- `-E` specifies encoding
- `-F` works similarly

- For rg: don't show line numbers with `-N`, for grep, show them with `-n`


- No ignore case:
	- rg: `-s`
	- grep: `--no-ignore-case`

- `m3`: shows 3 matches

- count Nb matches: grep option `-c` counts the nb of lines but `rg` first applies the `-o` option to show each match in a different line. For some reason, grep does not. 
	- grep: `grep -oi pattern file | wc -l`
	- rg: `rg -oic pattern file` (note that when there are multiple files, you must use wc -l trick)

- `-x` is for whole line match:  equivalent to putting `^..$` around each pattern
- `-w` is for whole words: equivalent to putting `\b..\b`
- `-e` for multiple patterns
- with `-A`, `-B`, `-C`, remove group separators with `--no-group-separator` for grep and `--no-context-separator` for rg
- specify group separator with `--group-separator='xx'` vs `--context-separator 'xx'` (no equal sign)

- Transform empty lines into another separator:
	- `grep -0 --group-separator='----' - "." file`
	- `rg --passthru '^$ -r '----' file`: here passthru prints both matching and not matching lines and `-r` replaces the matched lines 
	- note: for grep `-r` allows to recursively search in a directory, but this is the default for rg so this option can be used with passthru. rg respect `.gitignore` by default but you can use the `--hidden` flag

- grep: use `-H` to show filename and`-h` to show no filename
- with rg, instead of `-h`, we use `-I` for no filename (`-I` option in grep is equivalent to `--binary-files=without-match`)

- `-l` to show filename only, 
- `-L` shows filenames without match for grep, for rg, it follows symlink (`-R` for grep), so we must use `--files-without-match`

- glob with grep 
	- `--exclude=PAT`: exclude files whose filename match a pattern
	- `--exclude-from=FILE`: the patterns are now in the file, similarly to searching with `-f` option
	- `--exclude-dir=PAT`: exclude directory matching a pattern
	- there is also `--include=PAT`
- glob with rg: 
	- here with use directly `-g` for glob match, so for ex `-g "foo/**` searches in the foo directory and `-g "!backups"` ignores the backups dir
- Note: to search a given filetype, with grep, you use a combination of include and exclude, with rg, you use the `-t` flag

- when dealing with filenames which contain space, it can be useful to use `-Z` (or `--null`) for grep (equivalent is `--null-data` for rg). For ex: `grep -rlEZ PAT | xargs -r0 grep -L PAT`: here the filenames are passed as args for the second grep command


- Search only in symbolic links: `find . -type l -exec grep 'greeting' {} +`
	- `find . -type l`: search symbolic links in current dir
	- `-exec`: apply following command to found files
	- `grep 'greeting'`: the command to apply
	- `{}`: placeholder for find, replaced by actual file
	- `+`: pass all files at once to grep command. If we wanted to run a different grep command for each file, we would have used `\;`

- Multiline search:
	- grep: use `-z` to convert new line to null byte, do the search in a standard way and pipe to `tr '\0' '\n'`
	- rg: use `-U` flag
	- Not: in vim `\_.` matches any char including new line