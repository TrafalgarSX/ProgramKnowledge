### df
`disk free` command gives you a quick 30,000-foot view of your filesystem and all mounted disks. 
It tells you the total disk size, space used, space available, usage percentage, and what partition the disk is mounted on.
`df -h` make the data human-readable.

### du
`disk usage` command is excellent when applied in the correct context.
This command is  at its best when you need to see the size of a given directory or subdirectory. It runs at the object level and only reports on the specified stats at the time of execution.
`du -sh` give a human-readable summary of a specified object(the directory and all subdirectories).

### df vs. du

- The `df` command provides asweeping ballpark figure for how much space is being utilized on your filesystem as a whole.  
>df looks at disk used blocks directly in filesystem metadata. Because of this it returns much faster that du but can only show info about the entire disk/partition.
- The `du` command is a much more accurate snapshot of a given directory or subdirectory.
>du is used more than df in day to day project as it show s the disk usage as per directory level.

Basically, df reads the superblock only and trusts it completely. du reads each object and sums them up