# Day 16 – Shell Scripting Basics

## Task : Start your shell scripting journey — learn the fundamentals every script needs.

---

- ### What will happen if we don't use shebang in our scripts ?

#### Ans : When we write shebang (#!/bin/bash) we tells OS that it is a bash script and the OS gets it and runs the script accordingly but as we know there are other types of bash too like sh ,  zsh , dash & etc... so if we don't write shebang at the start of our script the OS will get confused. So, if we're on a different shell like sh , dash or zsh and we wrote a bash script but didn't add shebang then it give some error and that is only because of syntax and some shell specific-features.

```
- Script without shebang
name="John"
echo "Hello $name"

- Bash-specific feature used:
echo ${name^^}    # uppercase - works in bash only!
```

**One more Example** 

```
- Without shebang - runs under /bin/sh (dash on Ubuntu)
- dash is faster but doesn't support all bash features like:

arrays=("a" "b" "c")     #  breaks in sh/dash
[[ condition ]]          #  breaks in sh/dash  
echo {1..5}              #  breaks in sh/dash
```

#### We can also run the script using "bash" keyword it is as same as using shebang in the script but if we run the script using "./" then it the script depends on shebang or current shell 

```
bash script.sh    # works regardless of shebang
./script.sh       # depends on shebang or current shell
```

---

- #### Double quotes → Smart → Reads variables and strings differently While Single quotes → Dumb → Reads everything literally, no exceptions

**Ex :** 

```
NAME="John"

echo "My name is $NAME"    Output : My name is John
echo 'My name is $NAME'    Output : My name is $NAME
```

---

- #### To check weather a file exists or not we use -f flag in condition block 

**Ex :** 
- For file : `if [ -f "$file_name" ]; `
- For directory : `if [ -d "$dir_name" ]; `


---

- #### dpkg : To check installation status, installing local .deb files, and comparing software versions we use dpkg (debian package)

Ex : `if dpkg -s $package_name >/dev/null 2>&1;`  is used to find weather it is installed or not
- Here ` >/dev/null` means the output of the following will go to null (A blackhole) and then `2>&1` means Send error output (2) to wherever 1(Normal Output which is going to Blackhole) is going        
- ` >/dev/null 2>&1` together it means Send BOTH outputs to /dev/null (silence everything)
- /dev/null is like a dustbin ; Whatever you send to it → disappears forever



---

- #### Loops : We can Either write  `for i in {1..5}`  or  `for ((i=1; i<=10; i++))`

**Ex :**

- *"Range Iteration" or "Brace Expansion"*

```
for i in {1..5}
do
        read -p "Enter the name of the user (Type exit): " username

        if [ $username == "exit" ]; then
                echo "Exiting the loop...."
                break
        fi

        sudo useradd -m $username
        echo "User '$username' added successfully"
done
```


- *Incremental loop using double parentheses*

```
for ((i=1; i<=5; i++>))
do
        read -p "Enter the name of the user you want to delete : " username

        if [ $username == "exit" ]; then
                echo "Exiting the loop...."
                break
        fi

        sudo userdel  $username
        echo "User '$username' deleted successfully"
done
```

---
---

**Note** : 

```
When we want to use any by-default conditions (-lt , -gt) we have to use square-brackets "[ ]" 
But when we use commands as conditions we shouldn't use the brackets as the other commands won't work properly 

Ex : 

1st Case :  

if [dpkg -s nginx >/dev/null 2>&1]; then
    echo "Package Already Installed"
    exit 1
fi

Here exit 1 won't work
But if we remove square brackets 

2nd Case : 

if dpkg -s nginx >/dev/null 2>&1; then
    echo "Package Already Installed"
    exit 1
fi

Here exit 1 will work properly...
```


---
---