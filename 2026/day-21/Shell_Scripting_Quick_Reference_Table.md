# Shell Scripting — Quick Reference Table

| # | Topic | Key Syntax | What it Does | Example |
|---|-------|-----------|--------------|---------|
| 1 | **Single Quotes** | `'$VAR'` | No expansion — prints literally | `echo '$NAME'` → `$NAME` |
| 2 | **Empty Check** | `[[ -z "$VAR" ]]` | True if string is **empty** | `name=""` → `-z` is true |
| 3 | **Non-Empty Check** | `[[ -n "$VAR" ]]` | True if string is **not empty** | `name="Prem"` → `-n` is true |
| 4 | **File Non-Empty** | `[[ -s "$FILE" ]]` | True if file exists and size > 0 | `[[ -s log.txt ]]` |
| 5 | **case block** | `case $VAR in pattern) ;; esac` | Match variable against patterns | `case "$DAY" in Mon) echo "Monday";; *) echo "Other";; esac` |
| 6 | **case default** | `*)` | Fallback if no pattern matched | `*) echo "Unknown";;` |
| 7 | **until loop** | `until [[ condition ]]; do` | Loop until condition **becomes true** | `until [[ "$X" -eq 5 ]]; do (( X++ )); done` |
| 8 | **Loop over files** | `for f in *.txt` | Iterate over file globs | `for f in *.txt; do cat "$f"; done` |
| 9 | **Loop over output** | `for x in $(command)` | Iterate over command output | `for u in $(cut -d: -f1 /etc/passwd); do echo "$u"; done` |
| 10 | **Read file line by line** | `while read LINE; do ... done < file` | Process file line by line | `while read L; do echo "$L"; done < file.txt` |
| 11 | **Return status** | `return N` | Return exit code 0–255 only | `return 0` = success, `return 1` = fail |
| 12 | **Return value** | `echo val` inside fn + `$(fn)` | Return actual string/number | `result=$(square 5)` |
| 13 | **grep ERE** | `grep -E "p1\|p2"` | Extended regex — use `+`,`?`,`\|`,`()` | `grep -E "error\|warn" log.txt` |
| 14 | **awk delimiter** | `awk -F':'` | Set custom field separator | `awk -F: '{print $1}' /etc/passwd` |
| 15 | **awk BEGIN/END** | `awk 'BEGIN{} {} END{}'` | Run code before/after processing | `awk 'BEGIN{print "Start"} END{print "Done"}' file` |
| 16 | **awk built-ins** | `$0`, `NR`, `NF` | `$0`=full line, `NR`=line no., `NF`=field count | `awk '{print NR, $0}' file` |
| 17 | **sed substitute** | `sed 's/old/new/g'` | Replace all matches per line | `sed 's/foo/bar/g' file.txt` |
| 18 | **sed in-place** | `sed -i.bak 's/old/new/g'` | Edit file directly + keep backup | `sed -i.bak 's/dev/prod/g' config.txt` |
| 19 | **cut by delimiter** | `cut -d',' -f1` | Extract field by delimiter | `cut -d',' -f1 data.csv` |
| 20 | **cut by position** | `cut -c1-5` | Extract by character position | `cut -c1-5 file.txt` → first 5 chars |
| 21 | **sort by column** | `sort -t',' -k2` | Sort by specific column | `sort -t',' -k2 -n file.csv` |
| 22 | **uniq count** | `uniq -c` | Count occurrences of each line | `sort file.txt \| uniq -c \| sort -nr` |
| 23 | **uniq filter** | `uniq -d` / `uniq -u` | `-d` = duplicates only, `-u` = unique only | `sort file.txt \| uniq -d` |
| 24 | **tr translate** | `tr 'a-z' 'A-Z'` | Map chars in set1 → set2 | `echo "hello" \| tr 'a-z' 'A-Z'` → `HELLO` |
| 25 | **tr delete** | `tr -d '0-9'` | Delete characters from input | `echo "h3llo" \| tr -d '0-9'` → `hllo` |
| 26 | **tr squeeze** | `tr -s ' '` | Collapse repeated chars into one | `echo "hi   there" \| tr -s ' '` → `hi there` |
| 27 | **wc multiple files** | `wc -l *.log` | Count lines in all matched files + total | `wc -l *.log \| sort -n` |
| 28 | **Count files** | `ls \| wc -l` | Count items in a directory | `ls /etc \| wc -l` |
| 29 | **Extract line range** | `head -n Y file \| tail -n N` | Get lines X to Y | `head -n 10 file.txt \| tail -n 6` → lines 5–10 |
| 30 | **Live log filter** | `tail -f file \| grep --line-buffered` | Stream log and filter in real time | `tail -f app.log \| grep --line-buffered -i "error\|warn"` |
| 31 | **Monitor command** | `while true; do clear; cmd; sleep N; done` | Repeat any command every N seconds | `while true; do clear; df -h; sleep 5; done` |
| 32 | **trap EXIT** | `trap 'cmd' EXIT` | Run on **any exit** (normal or error) | `trap 'rm -f /tmp/tmp_$$' EXIT` |
| 33 | **trap INT** | `trap 'cmd' INT` | Run on **Ctrl+C** | `trap 'echo "Interrupted!"' INT` |
| 34 | **trap TERM** | `trap 'cmd' TERM` | Run on **`kill <PID>`** (graceful stop) | `trap 'echo "Terminating..."; exit 1' TERM` |
| 35 | **trap ERR** | `trap 'cmd' ERR` | Run on **any failed command** | `trap 'echo "Error on line $LINENO"' ERR` |

---

### Key Combos to Memorize

| Pattern | Use Case |
|---------|----------|
| `sort \| uniq -c \| sort -nr` | Frequency count (most common first) |
| `set -euo pipefail` | Strict mode — catch all errors |
| `grep -rl "old" . \| xargs sed -i 's/old/new/g'` | Replace string across multiple files |
| `while read L; do ... done < file` | Safe line-by-line file reading |
| `result=$(function_name)` | Capture function return value |
| `trap 'cleanup' EXIT` | Always clean up on exit |
| `tail -f log \| grep --line-buffered "error"` | Live error monitoring |