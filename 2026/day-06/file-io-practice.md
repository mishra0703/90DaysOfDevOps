# Day 06 – Linux Fundamentals: Read and Write Text Files


## - *Creating a file*

- `touch file_name.ext` : Will create a file with .ext (extension)


## - *Writing text in the file*

- `vim file.txt` : A text editor will get opened 

- `press i` : To enter Insert Mode and write anything you want.

- `press Esc` : To exit from Insert Mode

- `:wq` : Write and quit -> saves the file and whatever text you entered in it.


## - *Appending lines into the file*

-  `echo "Hello World" > file.txt` : If something was there in the file , that will get removed and "Hello World" get wrote in the file. Basically it overwrites the file

-  `echo "Hello World" >> file.txt` : It appends as new text in new line in the file , the old data was remains safe

## - *Reading a file*

- `cat file.txt` : Show everything in the file

- `head -n 5 file.txt` : Show first 5 lines of the file

- `tail -n 5 file.txt` : Show last 5 lines of the file


## - *Write and Display at the same time*

- `echo "Hello Dost" | tee -a file.txt` : Will append the text in the file and show it in the terminal too ; echo don't show after running

