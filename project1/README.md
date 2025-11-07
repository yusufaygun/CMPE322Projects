
# 🐚 MyShell — A Minimal Unix-like Shell

`myshell` is a small Unix-style shell implemented in C. It supports **command execution via `$PATH`**, **background jobs (`&`)**, **output redirection (`>`, `>>`)**, a custom **invert-and-append redirection (`>>>`)**, **persistent aliases**, and a built-in **`bello`** command that prints session/user info. High-level behavior is also summarized in the accompanying code report. 

---

## ✨ Features

* **Command execution through `$PATH`**
  Resolves executables by scanning directories in `PATH` and runs them with `execv`.

* **Background execution**
  Append `&` to run a process without blocking the shell.

* **Output redirection**

  * `cmd > file` – truncate and write
  * `cmd >> file` – append
  * `cmd >>> file` – **invert** (reverse) command output and append to `file` (implemented via a pipe + post-processing).

* **Persistent aliases**

  * Define with: `alias name="actual command and args"`
  * Aliases are stored in `AliasFile.txt` (append-only), and looked up on each command.

* **Built-in: `bello`**
  Prints username, hostname, last executed command, TTY, shell name, home directory, local time, and load average’s first window.

* **Friendly prompt**
  `username@hostname CWD --- `

---

## 🗂️ Project Structure

```
project1/
├─ myshell.c          # Shell implementation
├─ Makefile           # Build rules (gcc)
└─ code_report.pdf    # Short report on design & flow
```

---

## 🛠️ Build

Requirements: **gcc** (C11), a POSIX-like environment (Linux/macOS).

```bash
cd project1
make        # builds ./myshell
make clean  # removes objects and binary
```

---

## ▶️ Run

```bash
./myshell
```

You’ll see a prompt like:

```
user@host /current/dir --- 
```

Type commands just like a normal shell (with the supported subset below).

---

## 🧑‍💻 Usage & Examples

### 1) Regular commands

```bash
ls -la
date
uname -a
```

### 2) Background jobs

```bash
sleep 5 &
```

### 3) Redirection

```bash
echo hello > out.txt         # overwrite
echo world >> out.txt        # append
echo reverse-me >>> out.txt  # append the REVERSED output ("em-esrever")
```

> `>>>` works by routing the child’s stdout into a pipe, the parent **reverses** the captured buffer, and appends it to the file.

### 4) Aliases (persistent)

```bash
alias ll="ls -la"
ll
```

* Definitions are appended to `AliasFile.txt`.
* On each command, the shell checks if the first token is an alias and expands it before execution.

### 5) `bello` — session snapshot

```bash
bello
```

Outputs (for example):

```
1. Username: yusuf
2. Hostname: myhost
3. Last Executed Command: ls
4. TTY: /dev/pts/0
5. Current Shell Name: /bin/bash
6. Home Location: /home/yusuf
7. Current Time and Date: Fri Nov 7 20:03:21 2025
8. Current number of processes being executed: 0.42
```

---

## 🧩 Implementation Notes

* **Main loop**
  Prints the prompt, reads a line with `getline`, tokenizes, handles `exit`, `alias`, alias expansion, background mark (`&`), `bello`, and delegates to `executeCommand(...)`. 

* **Aliases**

  * `addAlias(name, cmd)` writes `name=cmd` to `AliasFile.txt`.
  * `findAlias(name)` scans the file, returning the first matching command string.
  * If the typed word matches an alias, the shell **replaces** the first token with the stored command and re-parses the line. 

* **Command resolution & exec**

  * Iterates directories in `$PATH`, builds `<dir>/<argv[0]>`, and uses `access(..., X_OK)` to locate the executable.
  * Spawns a child with `fork`, does redirection (or pipe for `>>>`), and calls `execv`. Parent `waitpid`’s unless background was requested. 

* **Invert redirection (`>>>`)**

  * Child writes stdout to pipe.
  * Parent reads into a buffer, **reverses** via `reverseBuffer(...)`, then appends to the output file. 

---

## ⚠️ Notes & Limitations

* **Quoting/escaping**: Tokenization is whitespace-based; complex quoting/escaping is limited.
* **Alias parsing**: `alias name="cmd args"` is expected; malformed lines may not be handled fully.
* **Buffers**: Fixed buffer sizes (e.g., for reversing output). Extremely large outputs may be truncated before reversal.
* **`errno` usage**: The program occasionally checks/sets `errno` as a simple status indicator; this is not a robust error-propagation method.
* **No piping between external commands (e.g., `ls | grep`)**: Only the special internal pipe for `>>>` is implemented.

---

## 🔮 Possible Enhancements

* Proper quoting/escaping and globbing.
* Real pipelines (`|`) and input redirection (`<`).
* History and arrow-key navigation (readline).
* Config file for aliases (load on startup, avoid linear search on every command).
* More robust error handling and safer buffer management.

---
