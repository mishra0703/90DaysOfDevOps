# Day 18 – Shell Scripting: Functions & intermediate Concepts

### Task : Write cleaner, reusable scripts — learn functions, strict mode, and real-world patterns.


### Task 1: Basic Functions
1. Create `functions.sh` with:
   - A function `greet` that takes a name as argument and prints `Hello, <name>!`
   - A function `add` that takes two numbers and prints their sum
   - Call both functions from the script

    ```bash
    greet() {
        echo "Hello, $1"
    }
    
    
    if (( $# == 0 )); then
        echo "WARNING : Give an argument"
        echo "Usage : ./greet_func.sh <Name>"
    else
        greet $1    # we passeed global argument as local argument to function
    fi
    ```

    ```bash
    sum() {
            ans=$(( $1 + $2 ))
            echo "Sum is $ans"
    }


    if (( $# == 0 )); then
            echo "WARNING : Pass two numbers as arguments"
            echo "Usage : ./sum_of_two.sh <num1> <num2>"
    else
            sum $1 $2       # Did the same thing here as well
    fi
    ```

---

### Task 2: Functions with Return Values
1. Create `disk_check.sh` with:
   - A function `check_disk` that checks disk usage of `/` using `df -h`
   - A function `check_memory` that checks free memory using `free -h`
   - A main section that calls both and prints the results

    ```bash
    check_disk_usage() {
            echo "********** Disk Usage **********"
            df -h | awk 'NR<=2 {print $2,$3,$4}'
    }


    check_memory_usage() {
            echo "********** Memory Usage **********"
            free -h | awk 'NR==2 {print $2,$3,$4}'
    }



    check_disk_usage
    check_memory_usage
    ```


---

### Task 3: Strict Mode — `set -euo pipefail`
1. Create `strict_demo.sh` with `set -euo pipefail` at the top
2. Try using an **undefined variable** — what happens with `set -u`?
3. Try a command that **fails** — what happens with `set -e`?
4. Try a **piped command** where one part fails — what happens with `set -o pipefail`?


    ```bash
    set -euo pipefail
    
    
    #Undefind Variables
    echo "$name"
    
    
    #Error Handling
    failed=false
    $failed || echo "This command got failed"
    
    

    cat fakefile.txt | grep "hello" | sort
    
    # If above command got failed thn we will get exit 1 and none of the code written below work
    # And the script stopped right here

    if (( $?==0 )); then
           echo "Above code runs successfully"
    fi
    ```



**Document:** What does each flag do?
- `set -u` → 
    - We will get --> name: unbound variable  
    - But if we don't use set -u and still use an undefined variable it will not show any error you'll just get blank space instead


- `set -e` → 
    - When you expect `0` or `false` or `Error` from a command you use `||` there 
    - The command in the right of `||` will run if you get any `Error` , `0` or `false` value from the command in the left of `||`    
    - `error detected` || Fallback logic to handle error (can be multiple lines too)


- `set -o pipefail` →
    - Without pipefail , Even though one command fails, $? shows 0 because maybe other command gets succeed. So, Script will keep running.
    - With set -o pipefail  , If any one of the command fails then the entire pipeline fails.
    - Basically It protects from pipeline failures.


---

### Task 4: Local Variables
1. Create `local_demo.sh` with:
   - A function that uses `local` keyword for variables
   - Show that `local` variables don't leak outside the function
   - Compare with a function that uses regular variables


```bash    
    # ─────────── Function with local keyword ───────────
    function local_demo() {
        local name="Prem"       # local → only lives inside this function
        echo "Inside local_demo : $name"
    }

    local_demo
    echo "Outside local_demo : $name"    # prints nothing — variable is gone

    # ─────────── Function without local ───────────
    function regular_demo() {
        name="Raj"               # It's a global variable by-default,which will leaks outside
        echo "Inside regular_demo : $name"
    }

    regular_demo
    echo "Outside regular_demo : $name"  # prints Raj — variable leaked out
```

---

### Task 5: Build a Script — System Info Reporter
Create `system_info.sh` that uses functions for everything:
1. A function to print **hostname and OS info**
2. A function to print **uptime**
3. A function to print **disk usage** (top 5 by size)
4. A function to print **memory usage**
5. A function to print **top 5 CPU-consuming processes**
6. A `main` function that calls all of the above with section headers
7. Use `set -euo pipefail` at the top

```bash
    hostInfo() {
            echo
            echo "***** Hostname *****"
            hostname
            echo
            echo "***** OS Info *****"
            cat /etc/os-release | awk 'NR>=2&&NR<=3'
            echo
    }

    systemUptime() {
            echo
            echo "***** System uptime *****"
            uptime | awk '{print $2,$3,$4,$5}'
            echo
    }


    diskUsage() {
            echo
            echo "***** Disk Usage (Top 5 by size) *****"
            df -h | awk 'NR==1'
            df -h | awk 'NR>=2&&NR<=5' | sort -h
            echo
     }



    check_memory_usage() {
            echo
            echo "********** Memory Usage **********"
            echo "Total Used Free"
            free -h | awk 'NR==2 {print $2,$3,$4}'
            echo
    }



    cpu_usage() {
            echo
            echo "***** CPU Usage *****"
            ps aux --sort=-%cpu | awk 'NR<=6 {print $1,$2,$3,$11}'
            echo
    }




    system_info() {
            echo "==============================="
            echo "      SYSTEM HEALTH REPORT     "
            echo "==============================="
            echo
    
            hostInfo
            echo
    
            systemUptime
            echo
    
            diskUsage
            echo
    
            check_memory_usage
            echo
    
            cpu_usage
            echo
    
            echo "==============================="
            echo "         END OF REPORT         "
            echo "==============================="
    }
    
    
    system_info
```
