# Day 03 – Linux Commands Practice

## Cheat sheet of commands focused on:

### - *File management*

- **ls** : List the names of files and subfolders the current directory.
- **cd 'Dir_Name'** : To change the directory
- **cd ..** : Up one level to enclosing folder or parent directory
- **cd** : To the $HOME directory
- **diff 'file_name' 'file_name'** : Compare two files A and B for differ Outputs the difference.
- **pwd** : Display the path of the current working directory
- **mkdir X** : Make a new directory named X inside the current directory.
- **mv A B** : Move a file from path A to path B. Also used for renaming files.
    Examples:
    - Moving between directories folder1 and folder2: `mv ./folder1/file.txt ./folder2`
  The file name will remain unchanged, and its new path will be `./folder2/file.txt`
    - Renaming a file: `mv new_doc.txt expenses.txt`
  The new file name will be `expenses.txt`

- **cp A B** : Copy a file from path A to path B. Used similarly to `mv`, use for both -> for copying to a new directory and simultaneously renaming the file in its new location.
    Example:
    - `cp ./f1/file.txt ./f2/expenses.txt` simultaneously copies the file `file.txt` to the new location with a new name `expenses.txt`

- **cp -r Y Z** : Recursively copy a directory Y and its contents to Z. If Z exists, copy source Y into it; otherwise, create Z and Y becomes its subdirectory with Y’s contents

- **rm X** : Remove/Delete (file name X) permanently .

- **rm -r Y** : Remove/Delete a directory Y and it's all content.

- **touch X** : Create an empty file X or update the access and modification times of X.

- **cat X** : View contents of X

- **cat -b X** : Also display line number as well

- **wc X** : Display word count of X

- **less X** : Work same as cat but lets you scroll and navigate comfortably, making it ideal for large files

---

### - *Process Management*

- **&** : Add this character to the end of a command/process to run it in the background. 
    Ex : `ping google.com > ping_output.txt &` this will run in the background

- **ps** : Show process status. Often used with grep , Ex `ps aux`    

- **ps -eo pid,user,command** : Display only the columns pid , user and command in ps output

- **top** : Display sorted information about processes

- **kill 'PID_Num'** : Kill a process specified by it's process id (PID), which you obtain using the ps command

- **killall 'Process_Name'** : Kill all processes containing 'Process_Name' in their name. Ex `killall ping` will kill all process containing ping in their name such as ping google.com ping youtube.com etc.

- **jobs** : See all background jobs/process you started

- **fg** : Bring the background process to front and show its live output. Use `ctrl + z` to pause it or `ctrl + c` to stop it 

- **df -h** : Show how full the disks are, in a human-readable format. `-h` = "human-readable" —converts raw bytes into KB, MB, GB automatically.


---

### - *Networking troubleshooting*

- **ifconfig** : Display all network interfaces with IP addresses 

- **ip a** : Another way to display all network interfaces with IP addresses

- **netstat** : Print open sockets of network connections, routing tables, interface statistics, masquerade connections, and multicast memberships.Pipe with the less command: e.g., `netstat -a

- **ping 'host'** : Send ICMP echo request to host, which may be a symbolic name, domain name or IP address. Ex : `ping google.com`

- **whois 'domain'** : Display whois information for domain In short --> who owns this domain, when it was registered, and when it expires

- **dig 'domain'** : Display DNS information for domain

- **host 'domain'** : Display DNS IP address for domain

- **wget 'Link'** : Download from location 'LINK'/Web-address and save in current directory. For ex : `https://file-examples.com/wp-content/storage/2017/10/file-example_PDF_1MB.pdf`

- **curl 'Link'** : Display the HTML source of LINK on terminal. Basically fetch and show content, great for APIs and testing

- **nmcli** : Command-line tool for managing network connections

- **nmcli device show** : Displays detailed network information about all network interfaces on your system.





This is the command toolkit you will reuse for years.
