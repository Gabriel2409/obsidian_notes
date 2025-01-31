#linux 
https://learnbyexample.github.io/learn_gnused/cover.html
https://www.gnu.org/software/sed/manual/sed.html

- separate multiple commands with ; : `sed 's/a/b/g; s/c/d/g' file`
- sed `y` command: map one char to another: `sed y/abc/123/`: will replace all occurences
- `&` char is the same as `\0`, full search
- `\L` and `\U` to make replacement lower/uppercase. Note that `\u` and `\l` only replace next matched char

- `-n` to only print lines with the `p` flag
	- `sed -n '1,3p;$p`=> shows lines 1 to 3 and last line
	- `sed '3d'` => deletes third line (note important not to use -n here)

- perform on range: 
	- `sed -n "1,3d;s/are/is/gp"`: first delete then print match
	- better: `sed -n "4,$ {s/are/is/gp}`

- Inside the command: `'n;'` Reads the next line into the pattern space, replacing the current line and advances to next line: for ex `sed "2~5{p;n;n;p}"`
	- starts on line 2
	- prints line 2, advances to line 3 then 4, then prints line 4
	- then applies pattern once again on line 7 (syntax is start~step)

- `q` flag: terminates input process: for ex `sed /is/q` stops after matching is (Q to exclude matching line )

- `sed -n /is/{/good/!p}` : 
	- -n: no automatic print
	- is: pattern to match
	- `{/good/!p}`: command to apply on matched pattern
		- if good is not matched (!), then print the line
	- In conclusion: prints all lines that match "is" but not "good"

- print line no before command: `sed '=' file`
