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
