# Day 21 – Shell Scripting Cheat Sheet: Build Your Own Reference Guide

## Task

```
You've spent the last several days learning Shell scripting — from basics to real-world projects. Now it's time to consolidate everything into a **personal cheat sheet** that you can use as a quick-reference guide for the rest of your DevOps journey.

The best way to revise is to **teach it back**. Writing a cheat sheet forces you to organize your understanding and identify gaps.
```

---


### Basics Commands

1. Shebang (`#!/bin/bash`) — what it does and why it matters
    - Tells the OS which interpreter to use when running the script
    - Without `#!/bin/bash` , running ./script.sh might use the wrong shell and break bash-specific syntax like arrays or [[ ]]

2. Running a script — `chmod +x`, `./script.sh`, `bash script.sh`
    - After writing a script , we have to make it executable by giving the execution access to `user group others` to whomever we want to give ----> `chmod u+x "filename.sh"` gives execution permission to user
    - Only then `./script.sh` or `bash script.sh` will run the script   
    - ./script.sh runs the script using the shebang interpreter — the ./ tells the shell to look in the current directory
    - bash script.sh runs it explicitly with bash — ignores the shebang, works even without execute permission
 

3. Comments — single line (`#`) and inline
    - `#` at the start of a line makes the entire line a comment — ignored completely by the interpreter 
    - For multiline comments use `<< comment` then add 'n' number of lines and wherever you write `comment` again the multiline comment will stop right there (The word can be anything : `comment` , `note` , `instructions` , `readme` , `usage` etc...)    


4. Variables — declaring, using, and quoting (`$VAR`, `"$VAR"`, `'$VAR'`)
    - Declaring of a variable : `VAR_NAME="Prem"` or `Var_Name=value` no spaces around =, and no $ when assigning
    - $VAR or "$VAR" expands the variable — always prefer "$VAR" (double quotes) to handle spaces and special characters safely
    - '$VAR' in single quotes treats everything literally — the variable is NOT expanded, $VAR prints as-is


5. Reading user input — `read`
    - read VAR pauses the script and waits for user input, storing what's typed into VAR
    - Use -p flag for an inline prompt message (`read -p "Enter name : " Name` here name is a  variable), and -s flag to hide input (useful for passwords)
    

6. Command-line arguments — `$0`, `$1`, `$#`, `$@`, `$?`
    - `$0` = script name
    - `$1`/`$2`... = positional arguments (./scrip.sh first`($1)` second`($2)`) 
    - `$#` = Total number of arguments passed
    - `$@` = All arguments as separate words (best for looping)
    - `$*` = All arguments as a single string
    - `$?` = Exit status of the last command — 0 means success, anything non-zero means failure

---

### Operators and Conditionals

1. String comparisons — `=`, `!=`, `-z`, `-n`
    - `=` checks if two strings are equal
    - `!=` checks if they are not equal — always use inside [[ ]]
    - `-z "$VAR"` returns true if the string is empty (zero length) ---> Ex : `name=""`
    - `-n "$VAR"` returns true if it is not empty   ---> Ex : `name="Prem"`




2. Integer comparisons — `-eq`, `-ne`, `-lt`, `-gt`, `-le`, `-ge`
    - `-eq` (equal), `-ne` (not equal) — check if two numbers are the same or different
    - `-lt` (less than), `-gt` (greater than) — check order between two numbers
    - `-le` (less than or equal), `-ge` (greater than or equal) — check order inclusively



3. File test operators — `-f`, `-d`, `-e`, `-r`, `-w`, `-x`, `-s`
    - `-e` checks if a file/dir exists, `-f` checks if it's a regular file, `-d` checks if it's a directory
    - `-r` checks if readable, `-w` checks if writable, `-x` checks if executable
    - `-s` checks if file exists and is non-empty (size > 0)



4. `if`, `elif`, `else` syntax
    - `if [[ condition ]]; then` starts the block — then is required, can be on the same line with `;` or on the next line
    - `elif [[ condition ]]; then` adds additional conditions — checked only if previous conditions were false
    - `else` is the fallback block — runs if no condition matched
    - `fi` closes the entire if block


5. Logical operators — `&&`, `||`, `!`
    - `&& = AND` — second command/condition runs only if the first succeeds (exit code 0)
    - `|| = OR` — second command/condition runs only if the first fails (non-zero exit code)
    - `! = NOT` — negates a condition (make the condition negative/not true) or exit status


6. Case statements — `case ... esac`
    - `case $VAR in` matches a variable against patterns — cleaner than long if/elif chains for multiple fixed values
    - Each pattern ends with `)` close bracket and its block ends with `;;` 
    - `;;` is required to break out of the match
    - `*` at the end acts as the default/fallback case — like else in an if block
    - `esac` closes the case block

Ex : 
    
```bash
    #!/bin/bash

    read -p "Enter day (Mon/Tue/Wed...): " DAY

    case "$DAY" in
        Mon)
            echo "📅 Start of work week!"
            ;;
        Tue | Wed | Thu)
            echo "📅 Midweek grind!"
            ;;
        Fri)
            echo "🎉 It's Friday!"
            ;;
        Sat | Sun)
            echo "😴 Weekend! Rest up!"
            ;;
        *)
            echo "❌ Invalid day entered"
            ;;
    esac
```



---

### Loops

1. `for` loop — list-based and C-style
    - List-based for loops iterate over a set of values — can be a static list, array, or command output
        Ex 
        ```bash
        # List-based — static list
        for NAME in Alice Bob Charlie; do
            echo "Hello, $NAME!"
        done

        #List-based — range
        for NUM in {1..5}; do
            echo "Number: $NUM"
        done
        ```
    - C-style for loops use `(( ))` with `init; condition; increment` — just like C/Java style loops
        Ex
        ```bash
        for ((i=1; i<=10; i++)); do
            echo "Count : $i"
        done
        ```    


2. `while` loop
    - `while [[ condition ]]; do` keeps looping as long as the condition is true — check happens before each iteration
    
    

3. `until` loop
    - `until [[ condition ]]; do` is the opposite of while — keeps looping until the condition becomes true
    - Think of it as `while NOT condition` — useful when you want to wait for something to happen

    Ex
    ```bash
    PASSWORD="secret"
    
    until [[ "$INPUT" = "$PASSWORD" ]]; do
        read -sp "Enter password: " INPUT
        echo ""
        if [[ "$INPUT" != "$PASSWORD" ]]; then
            echo "❌ Wrong password, try again!"
        fi
    done
    
    echo "Access granted!"    
    ```

    - `until [[ x ]]` is just `while [[ ! x ]]` 


4. Loop control — `break`, `continue`
    - `break` exits the loop early
    - `continue` skips to the next iteration 


5. Looping over files — `for file in *.log`
    - Use `for f in *.txt` to loop over file globs, or `$(command)` to loop over command output
        
        Ex
        ```bash
        # Loop over files
        for FILE in *.txt; do
            echo "Processing: $FILE"    #print every .txt file names
        done

        # Loop over command output
        for USER in $(cat /etc/passwd | cut -d: -f1); do
            echo "User: $USER"
        done
        ```  



6. Looping over command output — `while read line`
    - Use `while read line` to read a file line by line — the most common and efficient pattern for file processing
    
    Ex
    ```bash
    while read LINE; do
    echo "Line: $LINE"
    done < file.txt
    ```



---

### Functions

1. Defining a function — `function_name() { ... }`
    - Functions are defined with `function_name() { ... }` or `function function_name { ... }` — both are valid in bash
    - A function must be defined before it is called — bash reads top to bottom, calling before defining will fail
    - The function body goes inside `{ }` — always leave a space after `{ and put content }` on its own line 



2. Calling a function
    - Call a function by simply typing its name , no parentheses needed when calling (unlike defining)
    - Functions are called just like commands , you can pass arguments right after the name


3. Passing arguments to functions — `$1`, `$2` inside functions
    - Arguments passed to a function are accessed via `$1`, `$2`... inside the function , same as script arguments
    - `$@` inside a function gives all arguments passed to that function (not the script)
    - `$#` inside a function gives the count of arguments passed to that function
 


4. Return values — `return` vs `echo`
    - `return N` sends an `exit status (0-255)` only , not a string/value, accessible via `$?`
    - To return an actual value (string/number), use `echo` inside the function and capture with `$()`
    - Prefer `echo + $()` pattern for returning real data , return is best used for success/failure status

    Ex
    ```bash
    # return — exit status only
    is_even() {
        if (( $1 % 2 == 0 )); then
            return 0   # success = even
        else
            return 1   # failure = odd
        fi
    }

    is_even 4 && echo "Even" || echo "Odd"      # If true (return 0) then prints --> Even else prints --> Odd


    # echo — return actual value
    square() {
        echo $(( $1 * $1 ))
    }

    read -p "Enter the number you want square of : " num

    Result=$(square $num)
    echo "Square of $num is $Result"
    ```



5. Local variables — `local`
    - Variables inside functions are global by default — they can accidentally overwrite outer variables
    - Use keyword `local VAR=value` to make a variable scoped only to that function
    - Note : Always use local for function variables — it's a best practice to avoid side effects


---

### Text Processing Commands

1. `grep` — search patterns, `-i`, `-r`, `-c`, `-n`, `-v`, `-E`
    - `grep "pattern" filename` searches for matching lines in the file , by default case-sensitive and fixed string (means if you write `"error"` it will not show `"Error"` word)
    - `-i` = case insensitive, `-r` = recursive in directories, `-n` = show line numbers, `-c` = count matches, `-v` = invert match
    - `-E` enables extended regex (ERE) — lets you use +, ?, |, () without escaping 
    ```bash
    grep -E "error|warn" log.txt  # match "error" OR "warn"
    ```


2. `awk` — print columns, field separator, patterns, `BEGIN/END`
    - `awk '{print $1}'` prints the first column , fields are split by whitespace by default ($1, $2... $NF = last column)

    - `-F` sets a custom field separator — e.g. `-F: for /etc/passwd`, -F for CSV(comma separated value) files 
        - By default, awk assumes that columns (fields) in a text file are separated by whitespace (spaces or tabs). However, the /etc/passwd file uses colons (:) to separate user information.
        - It tells awk to treat the colon (:) as the boundary between fields instead of a space.

    - BEGIN { } runs before processing, END { } runs after — great for headers, totals, and summaries
    Ex 
    ```bash
    awk 'BEGIN {print "Start"} {print $1} END {print "Done"}' file.txt
    ```

    - `$0` = entire line, `NR` = current line number, `NF` = number of fields (essential awk built-in variables)



3. `sed` — substitution, delete lines, in-place edit
    - `sed 's/old/new/'` substitutes first match per line — add `-g` flag `(s/old/new/g)` to replace all occurrences
    - `sed '3d'` deletes line 3, `sed '/pattern/d'` deletes all lines matching a pattern
    - `-i` edits the file in-place — always use `-i.bak` to keep a backup before modifying    



4. `cut` — extract columns by delimiter
    - Use for CSV files
    - `cut -d',' -f1` cuts by delimiter and extracts field number — `-d` sets delimiter, `-f` sets field(s)
    - `-f1,3` extracts multiple fields, `-f2-4` extracts a range of fields
    - `-c1-5` cuts by character position instead of delimiter — useful for fixed-width files
    - *cut is simpler than awk for basic column extraction — use awk when you need conditional logic or math*



5. `sort` — alphabetical, numerical, reverse, unique
    - Default `sort` sorts alphabetically , `-n` sorts numerically, `-r` reverses the order
    - `-u` removes duplicates while sorting (like sort | uniq combined into one)
    - `-k2` sorts by the second column/field , combine with `-t','` to set a delimiter for columns

    Ex
    ```bash
    sort -rk2 sort_test.txt     #sorted the 2nd column of list in reverse order 
    
    banana 50
    dates 40
    apple 20
    apple 20
    cherry 10
    banana 10
    ```


6. `uniq` — deduplicate, count
    - Always sort first since it only catches adjacent duplicates   (sort filename | uniq 'Any_flag')
    - uniq removes consecutive duplicate lines 
    - `-c` adds a count of how many times each line appeared — great for frequency analysis
    - `-d` prints only duplicate lines, `-u` prints only unique (non-repeated) lines

    Ex 
    ```bash
    sort sort_test.txt | uniq -d 

    50 
    apple 20

    sort sort_test.txt | uniq -c 

    1 10
    1 20
    1 25
    2 50
    1 60
    2 apple 20
    1 banana 10
    1 banana 50
    1 cherry 10
    1 date 40
    ```


7. `tr` — translate/delete characters
    - `tr 'a-z' 'A-Z'` translates characters , maps each char in set1 to corresponding char in set2

    Ex 
    ```bash
    echo "hello world" | tr 'a-z' 'A-Z'
    HELLO WORLD
    ```
    - `-d` deletes characters from input , `tr -d '0-9'` removes all digits
    - `-s` squeezes repeated characters into one , `tr -s ' '` collapses multiple spaces into one

    Ex
    ```bash
    echo "hello    world" | tr -s ' '
    hello world

    # Replace character
    echo "hello:world" | tr ':' ' '       # hello world
    ```


8. `wc` — line/word/char count
    - `wc -l` counts lines, `wc -w` counts words, `wc -c` counts bytes/characters
    - Can take multiple files — prints counts per file plus a total at the end
    - Combine with pipes — `grep "error" log.txt | wc -l` counts how many lines match a pattern

    Ex
    ```bash
    wc -l newfile.txt sort_test.txt realWorldExample.txt    
    #count lines for each file and gives a total too
     
    05 newfile.txt
    13 sort_test.txt
    06 realWorldExample.txt
    24 total

    
    # Count files in directory
    ls | wc -l
    ```

9. `head` / `tail` — first/last N lines, follow mode
    - `head -n 5` shows the first 5 lines , `tail -n 5` shows the last 5 lines — default is 10 lines
    - `tail -f` follows a file in real-time — essential for monitoring live logs as they grow
    - Combine head and tail to extract a specific line range from a file

    Ex
    ```bash
    # Follow live log (Ctrl+C to stop)
    tail -f /var/log/syslog

    # Extract specific lines (lines 5-10)
    head -n 10 file.txt | tail -n 6

    # Skip first line (header) of CSV
    tail -n +2 file.csv
    ```



---

### Useful Patterns and One-Liners


- Find and delete files older than N days
    - `find /var/logs -name "*.log" -mtime +30 -delete`
    - Remove `-delete` first to preview what will be deleted


- Count lines in all `.log` files
    - `wc -l *.log | sort -n`
    - Use `wc -l **/*.log` to search recursively (enable with `shopt -s globstar`)


- Replace a string across multiple files
    - `grep -rl "old_text" . | xargs sed -i 's/old_text/new_text/g'` 
    - grep -rl finds all files containing the string recursively
    - `xargs` passes each file to `sed -i` for in-place replacement
    - Add `.bak` → `sed -i.bak` to keep backups before replacing


- Check if a service is running
    - `systemctl is-active nginx &>/dev/null && echo "Running" || echo "Stopped"`
    - `&>/dev/null` suppresses all output — we only care about exit status
    - `&&` triggers if running, `||` triggers if stopped



- Monitor disk usage with alerts
    ```bash 
    USAGE=$(df / | awk 'NR==2 {print $5}' | tr -d '%')
    [[ "$USAGE" -gt 80 ]] && echo "Disk usage at ${USAGE}%!" || echo "Disk OK: ${USAGE}%" 
    ```


- Parse CSV or JSON from command line
    - `awk -F',' filename`
    - `-F','` sets comma as field delimiter


- Tail a log and filter for errors in real time
    - `tail -f app.log | grep --line-buffered -i "error\|warn\|critical"` 
    - `--line-buffered` forces grep to flush each line immediately (critical for piped streaming)

- Find the Top 5 Largest Files in a Directory
    - `du -ah /var/log | sort -rh | head -n 5`

- Extract Unique IPs from a Log File
    - `grep -oE '[0-9]{1,3}(\.[0-9]{1,3}){3}' access.log | sort | uniq -c | sort -nr`

- Run a Command Every N Seconds (Poor Man's Watch)
    - `while true; do clear; df -h; sleep 5; done` , replace `df -h` with any command we want to monitor


---


### Error Handling and Debugging

1. Exit codes — `$?`, `exit 0`, `exit 1`
    - `$?` holds the exit code of the last command : 0 = success, anything else = failure
    - `exit 0` explicitly ends the script with success, `exit 1` (or any non-zero) signals failure
    - Exit codes range from 0–255 : common ones ----> 1 = general error, 2 = misuse, 127 = command not found



2. `set -e` — exit on error
    - `set -e` makes the script immediately exit if any command returns a non-zero exit code
    - Without it, bash silently continues after a failed command — dangerous in production scripts
    - Use `command || true` to allow specific commands to fail without triggering `set -e`

    Ex
    ```bash
    set -e


    rm optional_file.txt || true    # won't exit even if file missing
    ```

    - Put set -e right after the shebang — it's safest to enable it from the very start of the script




3. `set -u` — treat unset variables as error
    - `set -u` causes the script to exit if you use an undefined variable — catches typos in variable names
    - Without it, unset variables expand to an empty string silently — a very common source of bugs
    - Use `${VAR:-default}` to provide a default value for variables that might not be set

    Ex
    ```bash
    set -u


    # Safe default value workaround
    ENVIRONMENT=${ENV:-"production"}   # uses "production" if ENV not set
    ```
 

4. `set -o pipefail` — catch errors in pipes
    - By default, a pipeline's exit code is the last command's exit code — earlier failures are silently ignored
    - `set -o pipefail` makes the pipeline return the exit code of the first failed command in the pipe
    - Essential when using set -e — without pipefail, `set -e` won't catch pipe failures

    Ex
    ```bash
    # Without pipefail — this would PASS even though grep fails

    # cat file.txt | grep "pattern" | sort



    # With pipefail — any failure in the pipe exits the script

    cat nonexistent.txt | grep "pattern" | sort
    # Script exits here because cat failed
    ```



5. `set -x` — debug mode (trace execution)
    - `set -x` prints every command before executing it — prefixed with `+` so you can trace exactly what ran
    - Turn it off mid-script with `set +x` — useful to debug only a specific section
    - Combine with `set -e` to see exactly which command caused the script to exit

    Ex
    ```bash
    #!/bin/bash
    set -x   # enable debug mode

    NAME="Alice"
    echo "Hello, $NAME"
    ls /tmp

    set +x   # disable debug mode
    echo "This line won't be traced"
    ```


6. Trap — `trap 'cleanup' EXIT`
    - `trap 'command' SIGNAL` runs a command when a specific signal is received — `EXIT`, `INT`, `TERM`, `ERR`
    - `trap 'cleanup' EXIT` runs your cleanup function no matter how the script exits — normally or on error
    - `trap 'cleanup' INT` catches Ctrl+C — lets you clean up temp files before the script is killed

    ```bash
    Signal      Triggered by

    EXIT        Any exit — normal or error
    ERR         Any failed command
    INT         Ctrl+C
    TERM        kill <PID>
    ```

---
