# grep Cheat Sheet

## Basic Syntax

```
grep [OPTIONS] PATTERN [FILE...]
```

---

## Common Options

|Option|Description|
|---|---|
|`-i`|Case-insensitive match|
|`-v`|Invert match (show non-matching lines)|
|`-n`|Show line numbers|
|`-c`|Count matching lines|
|`-l`|List only filenames with matches|
|`-L`|List only filenames without matches|
|`-r` / `-R`|Recursive search through directories|
|`-w`|Match whole words only|
|`-x`|Match whole lines only|
|`-e`|Specify multiple patterns|
|`-f FILE`|Read patterns from a file|
|`-o`|Print only the matching part of the line|
|`-q`|Quiet mode (no output, exit status only)|
|`-s`|Suppress error messages|
|`-m N`|Stop after N matches|
|`--include=PATTERN`|Search only files matching pattern|
|`--exclude=PATTERN`|Skip files matching pattern|

---

## Context Options

|Option|Description|
|---|---|
|`-A N`|Show N lines **after** match|
|`-B N`|Show N lines **before** match|
|`-C N`|Show N lines **before and after** match|

---

## Pattern Types

|Option|Description|
|---|---|
|`-E`|Extended regex (ERE) — same as `egrep`|
|`-F`|Fixed string, no regex — same as `fgrep`|
|`-P`|Perl-compatible regex (PCRE)|
|`-G`|Basic regex (default)|

---

## Basic Examples

```bash
# Search for a word in a file
grep "error" app.log

# Case-insensitive search
grep -i "error" app.log

# Show line numbers
grep -n "error" app.log

# Invert match (lines that do NOT match)
grep -v "debug" app.log

# Count matches
grep -c "error" app.log

# Search recursively in a directory
grep -r "TODO" ./src

# Match whole word only
grep -w "log" app.log

# Show 3 lines of context around each match
grep -C 3 "error" app.log
```

---

## Multiple Patterns

```bash
# Match pattern1 OR pattern2
grep -E "error|warning" app.log
grep -e "error" -e "warning" app.log

# Match lines containing both words (pipe)
grep "error" app.log | grep "timeout"
```

---

## Regex Quick Reference

|Pattern|Matches|
|---|---|
|`.`|Any single character|
|`*`|Zero or more of preceding|
|`+`|One or more of preceding (ERE)|
|`?`|Zero or one of preceding (ERE)|
|`^`|Start of line|
|`$`|End of line|
|`[abc]`|Any of a, b, or c|
|`[^abc]`|Any character except a, b, c|
|`[a-z]`|Any lowercase letter|
|`\b`|Word boundary|
|`\d`|Digit (PCRE)|
|`\w`|Word character (PCRE)|
|`\s`|Whitespace (PCRE)|
|`{n,m}`|Between n and m repetitions (ERE)|

---

## Practical Recipes

```bash
# Find files containing a pattern
grep -rl "api_key" ./config

# Search only .py files recursively
grep -r --include="*.py" "import os" .

# Exclude a directory from recursive search
grep -r --exclude-dir=".git" "TODO" .

# Extract IP addresses (PCRE)
grep -oP '\d{1,3}(\.\d{1,3}){3}' access.log

# Find empty lines
grep -c "^$" file.txt

# Lines starting with a word
grep "^ERROR" app.log

# Lines ending with a pattern
grep "\.jpg$" file.txt

# Print only the matched portion
grep -o "[0-9]*" file.txt

# Stop after first 5 matches
grep -m 5 "error" app.log

# Search stdin (pipe)
cat app.log | grep -i "warn"

# Use a pattern file
grep -f patterns.txt app.log
```

---

## Exit Codes

|Code|Meaning|
|---|---|
|`0`|Match found|
|`1`|No match found|
|`2`|Error occurred|

---

## Tips

- Quote patterns to avoid shell expansion: `grep "pattern" file`
- Use `grep -P` for advanced patterns like lookaheads: `grep -P "(?<=foo)bar"`
- Combine with `find`: `find . -name "*.log" | xargs grep "error"`
- Use `zgrep` to search inside `.gz` compressed files