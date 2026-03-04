# Race Condition – TOCTOU (Filesystem)

**Platform:** TryHackMe
**Room:** Race Conditions
**Lab:** anti_flag_reader

---

# Overview

In this lab we analyze a vulnerable program called **anti_flag_reader**.
The purpose of this program is to prevent users from reading the flag file.

To enforce this restriction, the program performs two checks:

1. It checks if the provided file path contains the word **"flag"**
2. It checks whether the file is a **symbolic link**

If any of these checks fail, the program refuses to open the file.

However, the program performs these checks **before opening the file**, which introduces a race condition vulnerability.

---

# Vulnerable Program

Below is the full source code of the program used in this lab.

```c
#include <stdio.h>
#include <fcntl.h>
#include <string.h>
#include <unistd.h>
#include <assert.h>
#include <sys/stat.h>

int main(int argc, char **argv, char **envp) {

    int n;
    char buf[1024];
    struct stat lstat_buf;

    if (argc != 2) {
        puts("Usage: anti_flag_reader <FILE>");
        return 1;
    }
    
    puts("Checking if 'flag' is in the provided file path...");
    int path_check = strstr(argv[1], "flag");
    puts("Checking if the file is a symlink...");
    lstat(argv[1], &lstat_buf);
    int symlink_check = (S_ISLNK(lstat_buf.st_mode));
    puts("<Press Enter to continue>");
    getchar();
    
    if (path_check || symlink_check) {
        puts("Nice try, but I refuse to give you the flag!");
        return 1;
    } else {
        puts("This file can't possibly be the flag. I'll print it out for you:\n");
        int fd = open(argv[1], 0);
        assert(fd >= 0 && "Failed to open the file");
        while((n = read(fd, buf, 1024)) > 0 && write(1, buf, n) > 0);
    }
    
    return 0;
}
```

---

# Vulnerable Logic

The core issue appears in the following logic:

```c
lstat(argv[1], &lstat_buf);   // CHECK
...
int fd = open(argv[1], 0);    // USE
```

Program flow:

1. The program checks the file using `lstat()`
2. It waits for user input (`getchar()`)
3. Then it opens the file using `open()`

This creates the following pattern:

```
CHECK → lstat()
USE   → open()
```

---

# Why This Is Vulnerable

The program validates the file **before it is opened**.

Because of this, the filesystem state may change between the check and the use.

The program even prints:

```
<Press Enter to continue>
```

This pause creates a **race window**, where the file system state could change before the `open()` call happens.

As a result, the file that was validated may not be the same file that is later opened.

This is a classic **Time Of Check To Time Of Use (TOCTOU)** race condition.

---

# Root Cause

The main issue is that the program performs security checks on the **file path**, not on the actual opened file.

When `open()` is executed, the operating system resolves the path again.

If the file changed after the validation step, the program may end up opening a different file than the one it originally checked.

---

# Vulnerability Classification

This vulnerability can be classified as:

* Race Condition
* TOCTOU (Time Of Check To Time Of Use)
* Broken Access Control

---

# Mitigation

A safer approach is to open the file first and then validate it using the file descriptor.

Example safer implementation:

```c
int fd = open(path, O_RDONLY | O_NOFOLLOW);
fstat(fd, &stat_buf);
```

Security checks should be performed on the **file descriptor**, not on the file path.

---

# Key Takeaways

* Security checks performed on file paths can introduce race conditions.
* Filesystem state may change between operations.
* TOCTOU vulnerabilities occur when validation and usage are separated.
* Always validate resources after opening them.
