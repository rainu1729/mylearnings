# Sample Shell Script Commands

### Basic Commands
```bash
# Print "Hello, World!" to the terminal
echo "Hello, World!"

# List files in the current directory
ls -l

# Display the current working directory
pwd
```
### Groups 
```bash
# List the groups the logged in user is part of 
id
#list all groups and users in it
cat /etc/group
```
### Group creation 

```bash
# create a new group 
addgroup mymaingroup

#
```

### File Operations
```bash
# Create a new file
touch newfile.txt

# Copy a file
cp source.txt destination.txt

# Move or rename a file
mv oldname.txt newname.txt

# Delete a file
rm unwantedfile.txt
```

### Conditional Statements
```bash
# Check if a file exists
if [ -f "file.txt" ]; then
    echo "File exists."
else
    echo "File does not exist."
fi
```

### Loops
```bash
# For loop example
for i in {1..5}; do
    echo "Iteration $i"
done

# While loop example
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

### Functions
```bash
# Define and call a function
greet() {
    echo "Hello, $1!"
}

greet "User"
```

### Some common features of bash 
```bash
echo "\$1 each" ## escape the character
$1 each

echo '$1 each' ## $1 was not interpret
$1 each 

```

### File and directory permissions 
Setting up permission and ownership on files and directories 

viewing file permissions 
```bash 
echo "My sample first file" > mynewfile.txt

ls -l mynewfile.txt
-rw-r--r-- 1 group user 25 Dec 22 17:47 x
```
`-rw-r--r--` broken into 4 parts   
If it a directory it will start with `d` 

`rw-` permission the logged in user has     
`r--` permission the mentioned group in has   
`r--` permission all other users have  

It means the file `mynewfile.txt` can be read by any user but only I can write on it and no one can execute the file.


file permission names    
`r` can read the file content   
`w` can modify the file content  
`x` can execute the file  

Directory permission   
`r` list directory content using `ls`   
`w` add or remove files or directories    
`x` enter directory using cd command    

use `chmod` command to change permissions on file
```bash
chmod go-r mynewfile.txt
```

`go-r` remove the read permission from the `g` group and `o` others. 

### Sort and display lines 

view the lines in sorted alphanumerically 
```bash
sort mynewfile.txt
```

Drop any lines in the file that are identical and consective. 
```bash 
uniq mynewfile.txt
```

Command to specify a pattern and search for lines within a file.
```bash
grep people mynewfile.txt
```

| Option | Description | Example |
|----------|----------|----------|
|`-n`|along with matching lines print line number|` grep -n people mynewfile.txt`|
|`-c`|get the count of matching lines |` grep -c people mynewfile.txt`|
|`-i`|ignore the case of the text while matching |` grep -i people mynewfile.txt`|
|`-v`|Print all the lines that do not contain the pattern|`grep -v people mynewfile.txt`| 
|`-w`|Match only if the pattern matches whole words|`grep -w people mynewfile.txt`|  

### Merging two files 
`paste` command to view the two files merged together , line-by-line as columns delimited by a `Tab` character.
```bash
paste fileone.txt filetwo.txt
```

### Managing archive files 

|Option|Description|Example|
|------|-----------|-------|
|`-c`|Create new archive file|`tar -cvf bin.tar /bin`|
|`-v`|Verbosely list files processed|
|`-f`|Archive file name|
|`-t`|To see the list of files in the archive|`tar -tvf bin.tar`|
|`-x`|To untar the archive or extract files from the archive|`tar -xvf bin.tar`|

### User and System Information

- Return your user name:** `whoami`
- Return your user and group id:** `id`
- Return operating system name, username, and other info:** `uname -a`
- Display reference manual for a command:** `man <command>`
- List available man pages:** `man -k .`
- Get help on any command:** `<command> --help`
- Return the current date and time:** `date`

### Navigating and Working with Directories

- List files and directories by date:** `ls -lrt`
- Find files ending in .sh:** `find -name "*.sh"`
- Return path to present working directory:** `pwd`
- Make a new directory:** `mkdir <directory>`
- Change the current directory:** `cd <path>`
- Remove directory verbosely:** `rmdir <directory> -v`

### Monitoring System Performance and Status

- List running processes and their PIDs:** `ps`, `ps -e`
- Display resource usage:** `top`
- List mounted file systems and usage:** `df`

### Creating, Copying, Moving, and Deleting Files

- Create or update file timestamp:** `touch <file>`
- Copy a file:** `cp <source> <destination>`
- Move or rename a file:** `mv <source> <destination>`
- Remove a file verbosely:** `rm <file> -v`

### Working with File Permissions

- Make file executable for all users:** `chmod +x <file>`
- Make file executable for current user:** `chmod u+x <file>`
- Remove read permissions from group and others:** `chmod go-r <file>`

### Displaying File and String Contents

- Display file contents:** `cat <file>`
- Display file contents page-by-page:** `more <file>`
- Display first/last 10 lines:** `head -10 <file>`, `tail -10 <file>`
- Display string or variable value:** `echo "text"`, `echo "$VARIABLE"`

### Basic Text Wrangling

- Sort lines:** `sort <file>`
- Sort in reverse order:** `sort -r <file>`
- Drop consecutive duplicates:** `uniq <file>`

### Displaying Basic Stats

- Count lines:** `wc -l <file>`
- Count words:** `wc -w <file>`
- Count characters:** `wc -m <file>`

### Extracting Lines of Text Containing a Pattern

- Extract lines with pattern (case-insensitive, whole words):** `grep -iw <pattern> <file>`
- Extract lines from all .txt files:** `grep -l <pattern> *.txt`

### Merging and Extracting Columns

- Merge files line-by-line (tab-delimited):** `paste <file1> <file2> ...`
- Merge files with comma delimiter:** `paste -d "," <file1> <file2> ...`
- Extract column from CSV:** `cut -d "," -f <n> <file>`
- Extract bytes from each line:** `cut -b <range> <file>`

### Compression and Archiving

- Archive files:** `tar -cvf <archive> <files>`
- Compress files/folders:** `zip <archive> <files/folders>`
- Extract files from zip:** `unzip <archive> [-d <directory>]`

### Networking Commands

- Print hostname:** `hostname`
- Send packets to URL:** `ping <url>`
- Display/configure network interfaces:** `ip`
- Display contents of file at URL:** `curl <url>`
- Download file from URL:** `wget <url>`


