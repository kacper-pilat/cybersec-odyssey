# OverTheWire: Bandit — Write-ups

Notes from working through the Bandit wargame, documenting the reasoning behind each solution rather than just the final command. Passwords are intentionally omitted - publishing solutions defeats the purpose of the challenge for other learners.

---

## Level 0 → 1

**Category:** Linux fundamentals — navigation & file reading

**Problem:** The password for `bandit1` is stored in a file called `readme` in the home directory.

**Approach:** Listed the contents of the home directory to confirm the file was there, then printed its contents.

**Solution:**
```bash
ls
cat readme
```

**What I learned:** `ls` lists the contents of a directory. `cat` prints a text file's contents straight to the terminal, no decryption or processing involved, it's a plain read.

---

## Level 1 → 2

**Category:** Linux fundamentals — special characters in filenames

**Problem:** The password is stored in a file literally named `-` (a single dash), in the home directory.

**Approach:** `ls` revealed the file, but `cat -` didn't return anything useful — the shell interprets a leading dash as the start of a command-line option rather than a filename, so `cat` was waiting on stdin instead of reading the file.

**Solution:**
```bash
cat ./-
```

**What I learned:** Prefixing a filename with `./` tells the command "this is a path in the current directory," which removes the ambiguity. Any filename starting with a dash will cause this same issue with most Linux commands.

---

## Level 2 → 3

**Category:** Linux fundamentals — spaces and special characters combined

**Problem:** The password is stored in a file named `--spaces in this filename--`.

**Approach:** This one stacked two separate problems on top of each other: the leading dashes (same issue as Level 1) and the spaces inside the name, which the shell splits into separate arguments by default. I tried each fix on its own before realizing both were needed at the same time.

**Solution:**
```bash
cat "./--spaces in this filename--"
```

**What I learned:** Quoting a filename groups everything inside the quotes into a single argument, which solves the spaces problem. That's a different fix from the `./` prefix, which solves the leading-dash problem. Filenames that break normal conventions often need more than one fix applied together.

---

## Level 3 → 4

**Category:** Linux fundamentals — hidden files & recursive search

**Problem:** The password is stored somewhere inside the `inhere` directory, in a file with a deliberately obscure name.

**Approach:** A plain `ls` inside `inhere` wasn't enough to spot it. Switched to `find` to list everything recursively, which surfaced a hidden file with a suspicious name that `ls` hadn't shown by default.

**Solution:**
```bash
find .
cat "./inhere/...Hiding-From-You"
```

**What I learned:** `find` traverses directories recursively and will surface hidden files (names starting with a dot) that a plain `ls` skips unless you pass extra flags. Useful whenever a file is deliberately tucked away.

---

## Level 4 → 5

**Category:** Linux fundamentals — filtering by file type

**Problem:** The password is stored in `inhere`, mixed in among several decoy files with different encodings. Only one of them is plain, human-readable text.

**Approach:** Similar starting point to Level 3→4 — `find` to see everything inside the directory — but the real work here was going through the candidates and identifying which one was actually readable text rather than binary or garbled data.

**Solution:**
```bash
find inhere
cat ./inhere/-file07
```

**What I learned:** Not every file that "looks" like a candidate actually holds the answer — part of the job is filtering out noise, not just locating files.

---

## Level 5 → 6

**Category:** Linux fundamentals — combining `find` with size filters

**Problem:** The password is stored somewhere under `inhere` (multiple nested subdirectories) in a file that is human-readable, exactly 1033 bytes, and not executable.

**Approach:** This one took the most trial and error. My first attempts got the size unit wrong — `find`'s `-size` option takes a suffix letter, and it's easy to assume `b` means "bytes." It doesn't: `b` defaults to 512-byte blocks, `c` is the one that means bytes. I also initially tried `cd`-ing into `inhere` before running `find`, which wasn't necessary — `find` accepts a starting path as its first argument and searches recursively from there on its own.

**Solution:**
```bash
find inhere -size 1033c
cat ./inhere/[path-returned-above]
```

**What I learned:** Two things stuck: first, read flag documentation carefully — similarly-named suffixes (`b` vs `c`) can mean very different things and it's a common trap. Second, `find` doesn't require navigating into a directory first; passing the path as an argument and layering filters (`-size`, `-type`, etc.) on top is the more efficient pattern. This was the most satisfying level so far — the first one that genuinely required combining several pieces of `find`'s syntax rather than applying one fix.
