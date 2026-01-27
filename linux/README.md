# Linux Interview Questions

## Linux

### Q1. What is the difference between a Hard Link and a Soft (Symbolic) Link?

A **Soft Link** points to a filename (like a shortcut in Windows). If you delete the original file, the link breaks.

A **Hard Link** points to the specific **inode** (data location) on the disk. If you delete the original file, the hard link still works because the data is still there.

---

### Q2. What is an Inode?

An **Inode** is a data structure that stores metadata about a file (permissions, owner, size, location on disk), but not the filename or the actual data content.

---

### Practice 1: The "Broken Link"

```bash
# Create a dummy file
echo "this is important data" > original.txt

# Create soft and hard links
ln -s original.txt soft.lnk
ln original.txt hard.lnk

# View files with inode numbers
ls -li
```

---

### Q3. Explain the permission 755 vs 644

> **Note:** Permission values are calculated by adding:
>
> | Permission  | Value |
> | ----------- | ----- |
> | Execute (x) | 1     |
> | Write (w)   | 2     |
> | Read (r)    | 4     |

- **755** (`rwxr-xr-x`): Owner can Read/Write/Execute. Everyone else can Read/Execute. _(Standard for scripts/directories)_
- **644** (`rw-r--r--`): Owner can Read/Write. Everyone else can only Read. _(Standard for text files/configs)_

---

### Q4. How do I switch to the root user? What is the difference between `su -` and `sudo -i`?

- **`su -`**: Switches you to root but requires the **root password**.
- **`sudo -i`**: Switches you to root using **your own password** (if you are in the sudoers group).

---

### Practice 2: The "Secret Project"

**Scenario:** You need to create a setup where a specific group can work on files, but outsiders cannot see them.

```bash
# Step 1: Create two users with home directories and bash shell
sudo useradd -m -s /bin/bash joe
sudo useradd -m -s /bin/bash jane

# Step 2: Create a new group for the team
sudo groupadd devteam

# Step 3: Add both users to the group
sudo usermod -aG devteam joe
sudo usermod -aG devteam jane

# Step 4: Create the project directory
sudo mkdir /opt/project

# Step 5: Set ownership (root owns it, devteam group has access)
sudo chown root:devteam /opt/project

# Step 6: Set permissions so only owner and group can access
sudo chmod 770 /opt/project
```

---

### Q5. What is the difference between `>` and `>>`?

- **`>` (Redirect)**: Overwrites the file. If the file had data, it's gone.
- **`>>` (Append)**: Adds to the end of the file, keeping existing data safe.

```bash
# Example: Overwrite
echo "First line" > file.txt    # file.txt contains: "First line"
echo "Second line" > file.txt   # file.txt contains: "Second line" (First line is gone!)

# Example: Append
echo "First line" > file.txt    # file.txt contains: "First line"
echo "Second line" >> file.txt  # file.txt contains: "First line\nSecond line"
```

---

### Q6. What is `grep` used for? Can you give an example?

**`grep`** (Global Regular Expression Print) is used to search for patterns in text or files. It returns lines that match the pattern.

```bash
# Search for "error" in a log file
grep "error" /var/log/syslog

# Case-insensitive search
grep -i "error" /var/log/syslog

# Show line numbers
grep -n "error" /var/log/syslog

# Search recursively in a directory
grep -r "TODO" /home/user/project/
```

---

### Q7. What is the Pipe `|` operator?

The **pipe** (`|`) takes the output of one command and sends it as input to another command. It allows you to chain commands together.

```bash
# Count how many lines contain "error"
grep "error" /var/log/syslog | wc -l

# Find running processes and filter for "nginx"
ps aux | grep nginx

# List files and show only the first 5
ls -la | head -5

# Chain multiple pipes: find large files and sort by size
du -h /var/log | sort -h | tail -10
```

> **Key Concept:** Without pipes, you'd need to save intermediate results to files. Pipes let you process data in a single line, making shell scripting powerful and efficient.

---

### Q8. What is `sed` and how do you use it?

**`sed`** (Stream Editor) is a command-line tool for parsing and transforming text. It's most commonly used for find-and-replace operations.

```bash
# Basic syntax: sed 's/old/new/' file

# Replace first occurrence of "foo" with "bar" on each line
sed 's/foo/bar/' file.txt

# Replace ALL occurrences (global flag)
sed 's/foo/bar/g' file.txt

# Edit file in-place (modify the actual file)
sed -i 's/foo/bar/g' file.txt

# Delete lines containing a pattern
sed '/error/d' file.txt

# Delete line 5
sed '5d' file.txt

# Print only lines 10-20
sed -n '10,20p' file.txt
```

| Flag         | Purpose                        |
| ------------ | ------------------------------ |
| `s/old/new/` | Substitute first match         |
| `g`          | Global (all matches on line)   |
| `-i`         | In-place edit                  |
| `-n`         | Suppress output (use with `p`) |
| `d`          | Delete matching lines          |

---

### Q9. What is `awk` and how do you use it?

**`awk`** is a powerful text-processing language. It's excellent for working with columnar data (like CSV, logs, or command output).

```bash
# Basic syntax: awk '{action}' file
# $1, $2, $3... = columns, $0 = entire line

# Print the first column
awk '{print $1}' file.txt

# Print first and third columns
awk '{print $1, $3}' file.txt

# Use a custom delimiter (e.g., colon for /etc/passwd)
awk -F: '{print $1, $7}' /etc/passwd

# Print lines where column 3 is greater than 100
awk '$3 > 100' file.txt

# Sum values in column 2
awk '{sum += $2} END {print sum}' file.txt

# Print line numbers with content
awk '{print NR, $0}' file.txt
```

| Variable    | Meaning                    |
| ----------- | -------------------------- |
| `$0`        | Entire line                |
| `$1, $2...` | Column 1, 2, etc.          |
| `NR`        | Current line number        |
| `NF`        | Number of fields (columns) |
| `-F`        | Set field delimiter        |

> **sed vs awk:** Use `sed` for simple find/replace. Use `awk` when you need to work with columns or do calculations.

---

### Practice 3: The "Config Fixer" (Sed & Grep)

**Scenario:** You have a configuration file with the wrong database IP, and you need to fix it without opening a text editor.

```bash
# Step 1: Create a sample config file
echo "db_host=192.168.1.10" > config.app
echo "db_port=5432" >> config.app
echo "app_mode=production" >> config.app

# Step 2: Verify the current db_host value
cat config.app | grep -i db_host

# Step 3: Replace the old IP with the new one (in-place)
sed -i 's/192.168.1.10/10.0.0.5/g' config.app

# Step 4: Verify the change
cat config.app
```

> **Result:** The database IP is changed from `192.168.1.10` to `10.0.0.5` without ever opening a text editor—perfect for scripting and automation!

---

### Q10. What is a Zombie Process?

A **Zombie Process** is a process that has finished execution (died) but still has an entry in the process table because its parent process hasn't "read" its exit status yet.

- Uses almost no resources but takes up a PID slot
- Shows as `Z` or `defunct` in `ps` output
- Fixed by killing the parent process or having the parent call `wait()`

```bash
# Find zombie processes
ps aux | grep -w Z
```

---

### Q11. What is the difference between SIGTERM (15) and SIGKILL (9)?

| Signal      | Number | Behavior                                                                                                        |
| ----------- | ------ | --------------------------------------------------------------------------------------------------------------- |
| **SIGTERM** | 15     | The polite way to ask a process to stop. Allows the process to save data, close files, and clean up gracefully. |
| **SIGKILL** | 9      | The nuclear option. Kills the process instantly. No cleanup, no saving—use as last resort.                      |

```bash
# Graceful stop (SIGTERM) - preferred
kill 1234
kill -15 1234

# Force kill (SIGKILL) - last resort
kill -9 1234
```

> **Best Practice:** Always try `SIGTERM` first. Only use `SIGKILL` if the process doesn't respond.

---

### Q12. How do I run a command in the background?

Add `&` at the end of the command to run it in the background.

```bash
# Run a command in the background
./long_running_script.sh &

# See background jobs
jobs

# Bring a background job to foreground
fg %1

# Send a running foreground process to background
# Press Ctrl+Z to pause, then:
bg
```

| Command           | Purpose                            |
| ----------------- | ---------------------------------- |
| `command &`       | Start command in background        |
| `jobs`            | List background jobs               |
| `fg %n`           | Bring job n to foreground          |
| `bg %n`           | Resume paused job in background    |
| `nohup command &` | Run in background, survives logout |
| `disown`          | Detach job from terminal           |

---

### Practice 4: The "Background Job"

**Scenario:** You will start a "long-running" process, put it in the background, and then manage it.

```bash
# Step 1: Start a long-running process
sleep 1000

# Step 2: Pause the process (press Ctrl+Z)
# This suspends the process and returns you to the shell
# You'll see: [1]+  Stopped    sleep 1000

# Step 3: View paused/background jobs
jobs

# Step 4: Resume it in the background
bg %1

# Step 5: Verify it's running
ps aux | grep sleep

# Step 6: Kill the process (replace with actual PID from step 5)
kill -9 <PID>
# Or kill by job number:
kill %1
```
