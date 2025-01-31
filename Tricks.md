#todo 

Suspend shell process with Ctrl-Z
Check with jobs
Restart in foreground with fg
Restart in background with bg
Note: add & after a shell command tto launch in bg


umask: default is 022. 
Substracted from file/dir on creation. NOTE: default nb for file is 666 and dir is 777. So when creating a new file, by default, perms are 644, so -rw-r--r--



ln -s linkname target: creates a soft link. You can see it with ls -l
ln without -s creates a hardlink. Hard to see but if you modify the content of one the other is modified. This can be seen with ls -i that checks for the inode nb.
find all files that have at least one hardlink: find mydir -type f -links +1

- if podman does not start with systemctl, run `podman system service --time=0 &`