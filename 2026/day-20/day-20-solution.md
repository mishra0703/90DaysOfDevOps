# Day 20 – Bash Scripting Challenge: Log Analyzer and Report Generator

### Task

```
You are a system administrator responsible for managing a network of servers. Every day, a log file is generated on each server containing important system events and error messages. Your job is to analyze these log files, identify specific events, and generate a summary report.

Write a Bash script (`log_analyzer.sh`) that automates the process of analyzing log files and generating a daily summary report.
```


---

## Challenge Tasks

### Task 1: Input and Validation
Your script should:
1. Accept the path to a log file as a command-line argument
2. Exit with a clear error message if no argument is provided
3. Exit with a clear error message if the file doesn't exist

---

### Task 2: Error Count
1. Count the total number of lines containing the keyword `ERROR` or `Failed`
2. Print the total error count to the console

---

### Task 3: Critical Events
1. Search for lines containing the keyword `CRITICAL`
2. Print those lines along with their line number

Example output:
```
--- Critical Events ---
Line 84: 2025-07-29 10:15:23 CRITICAL Disk space below threshold
Line 217: 2025-07-29 14:32:01 CRITICAL Database connection lost
```

---

### Task 4: Top Error Messages
1. Extract all lines containing `ERROR`
2. Identify the **top 5 most common** error messages
3. Display them with their occurrence count, sorted in descending order

Example output:
```
--- Top 5 Error Messages ---
45 Connection timed out
32 File not found
28 Permission denied
15 Disk I/O error
9  Out of memory
```

---

### Task 5: Summary Report
Generate a summary report to a text file named `log_report_<date>.txt` (e.g., `log_report_2026-02-11.txt`). The report should include:
1. Date of analysis
2. Log file name
3. Total lines processed
4. Total error count
5. Top 5 error messages with their occurrence count
6. List of critical events with line numbers

---

### Task 6 (Optional): Archive Processed Logs
Add a feature to:
1. Create an `archive/` directory if it doesn't exist
2. Move the processed log file into `archive/` after analysis
3. Print a confirmation message

---


```bash
#!/bin/bash


log_file=$1


if [ $# -eq 0 ]; then
        echo "Usage : Run the script with a path to a log file"
        echo "Ex : ./log_analyzer.sh <Path of a log file>"
        exit 1
fi

if [ ! -f $log_file ]; then
        echo "File does not exist"
fi




error_count() {
        echo
        echo "***** Total Error Count *****"
        grep -iE 'Error|failed' $log_file | wc -l
        echo
}


critical_log() {
        echo
        echo "***** Total Critical Error Count *****"
        grep -iE "critical" $log_file | wc -l
        echo
        echo
        echo "----- Critical Events ------"
        echo
        echo "Line - Status --------------- Description ---------------"
        echo
        awk '{for(i=6; i<=NF; i++) printf "%s ", $i; print ""}' $log_file | grep -in "critical"
        echo
}



top_errors() {
        echo
        echo "***** Top Error Messages *****"
        echo
        echo "Occurrence -- Description ----"
        grep '\[error\]' File_logs/Apace_2k_logs.log | awk '{$1=$2=$3=$4=$5=$6=""; print $0}' | sort | uniq -c | sort -rn | head -5
        echo
}


timestamp=$(date '+%Y-%m-%d-%H-%M-%S')

log_report() {
        echo
        echo "----------- Log Report ----------"
        echo
        echo "Date : $(date)"
        echo "Filename : log_report_$timestamp.txt"
        echo "Total Lines Processed : $(wc -l < $log_file)"
        echo
        echo
        error_count
        echo
        echo
        top_errors
        echo
        echo
        critical_log
        echo
        echo
}




archieve_dir="Archieves"

if [ -d "$archieve_dir" ]; then
        log_report > $archieve_dir/log_report_$timestamp.txt
        echo "Log report has been generated"
        echo "Note : You can find the report in Archieves folder"
else
        mkdir "$archieve_dir"
        log_report > $archieve_dir/log_report_$timestamp.txt
        echo "Log report has been generated"
        echo "Note : You can find the report in Archieves folder"
fi


```



