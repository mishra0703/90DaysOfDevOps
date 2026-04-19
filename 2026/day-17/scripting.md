# Day 17 – Shell Scripting: Loops, Arguments & Error Handling

### Task : Level up your scripting — use loops, handle arguments, and deal with errors.


---

## Challenge Tasks

### Task 1: For Loop
1. Create `for_loop.sh` that:
   - Loops through a list of 5 fruits and prints each one

    ```bash
    for fruit in apple mango banana grapes pineapple
    do
        echo $fruit
    done
    ```

2. Create `count.sh` that:
   - Prints numbers 1 to 10 using a for loop

    ```bash
    for i in {1..10}
    do
        echo $i
    done
    ```

---

### Task 2: While Loop
1. Create `countdown.sh` that:
   - Takes a number from the user
   - Counts down to 0 using a while loop
   - Prints "Done!" at the end

    ```bash
    read -p "Enter a number : " num

    while (( $num >= 0 ))
    do
        echo $num
        num=$(( num - 1 ))
    done
    ```

*Note* : In Bash there are two ways to write conditions, and each has its own syntax:
 
- Using (( )) — Arithmetic context → use `<=`, `>=`, `<`, `>`, `==`, `!=` all work here

- Using [ ] — Test context → use `-le`, `-ge`, `-lt`, `-gt`, `-eq`, `-ne` work here

---

### Task 3: Command-Line Arguments
1. Create `greet.sh` that:
   - Accepts a name as `$1`
   - Prints `Hello, <name>!`
   - If no argument is passed, prints "Usage: ./greet.sh <name>"

    ```bash
    if (( $# == 0 )); then
            echo "Usage: ./greetings.sh <name>"
    else
            echo "Hello, $1"
    fi
    ```

2. Create `args_demo.sh` that:
   - Prints total number of arguments (`$#`)
   - Prints all arguments (`$@`)
   - Prints the script name (`$0`)

    ```bash
    echo "Total arguments : $#"
    echo "All arguments : $@"
    echo "Script Name : $0"
    ```

---

### Task 4: Install Packages via Script
1. Create `install_packages.sh` that:
   - Defines a list of packages: `nginx`, `curl`, `wget`
   - Loops through the list
   - Checks if each package is installed (use `dpkg -s` or `rpm -q`)
   - Installs it if missing, skips if already present
   - Prints status for each package

    ```bash
    pkgs=("nginx" "docker" "curl" "wget")


    for pkg in "${pkgs[@]}"
    do
            if dpkg -s "$pkg" >/dev/null 2>&1; then
                    echo "$pkg is installed"
            else
                    echo "$pkg is not installed"

                    read -p "Do you want to install $pkg (Y/N)" ans

                    if [[ "$ans" == "Y" || "$ans" == "y" ]]; then

                            echo "***************Installing $pkg***************"
                            sudo apt install $pkg -y

                    elif [[ "$ans" == "N" || "$ans" == "n" ]]; then

                            continue
                    fi
            fi
    done
    ```


---

### Task 5: Error Handling
1. Create `safe_script.sh` that:
   - Uses `set -e` at the top (exit on error)
   - Tries to create a directory `/tmp/devops-test`
   - Tries to navigate into it
   - Creates a file inside
   - Uses `||` operator to print an error if any step fails

    ```bash
    set -e

    mkdir /tmp/devops-test || echo "Failed to create folder as it already exists"
    ## if mkdir fails we will see "Failed to create..."

    touch /tmp/devops-test/test.txt || echo "Failed to create file as it already exists"
    ## this will never fail as running touch again n again only updates timestamps

    echo "Hello I am from test.txt file" >> /tmp/devops-test/test.txt
    ## this will run weather mkdir fails or not 
    ```

*Note* : 

- set -e is useful for unexpected/unhandled failures — things you did NOT anticipate.
- Without set -e, the script keeps running even after those unexpected failures — which can cause bigger damage.
- *set -e is your safety net for everything you didn't think to handle. if/else is for logic you consciously control. You need both in a well-written script.*

---

2. Modify your `install_packages.sh` to check if the script is being run as root — exit with a message if not.

    ```bash
    if [ "$EUID" -ne 0 ]; then 
        echo "Run as root"; exit 1; 
    fi
    ```

    ### OR

    ```bash
        set -e  #at the top of the script

            if [[ "$ans" == "Y" || "$ans" == "y" ]]; then

                echo "***************Installing $pkg***************"
                apt install $pkg -y || { echo " Warning ";  echo "Usage : sudo ./script.sh"; echo " Exiting "; exit 1; }
    ```

---

