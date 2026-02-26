## Layer 2 Active MITM – Reverse Shell Command Manipulation

### Overview

This lab demonstrates how a Layer 2 attack can escalate from passive sniffing to active session manipulation.

Using MAC flooding, the switch CAM table was exhausted, forcing the device into fail-open behavior and enabling traffic interception.
After establishing a Man-in-the-Middle position with Ettercap, a plaintext reverse shell session was intercepted and manipulated in real time.

---

### Attack Chain

#### 1. CAM Table Exhaustion

MAC flooding was performed using Ettercap, causing the switch CAM table to overflow.
As a result, the switch began broadcasting frames similarly to a hub, enabling traffic interception.

#### 2. MITM Establishment

Ettercap was used to perform ARP poisoning and intercept traffic between hosts.

This allowed visibility into:

* HTTP traffic
* Credentials
* Interactive reverse shell session

---

#### 3. Reverse Shell Interception

During interception, a plaintext reverse shell session was observed where commands and outputs were transmitted over raw TCP.

Examples:

* `ls`
* `whoami → root`

This confirmed privileged shell access over an unencrypted channel.

---

#### 4. Active MITM Command Manipulation

Using Etterfilter, inline packet modification was performed to substitute commands transmitted over the reverse shell.

Steps:

1. Compile Etterfilter C-like filter
2. Detect `whoami` command
3. Replace with alternative command
4. Validate execution via returned shell output

This demonstrated:

* Real-time command substitution
* Active control over intercepted shell session
* Manipulation of privileged execution context

---

### Key Insights

* MAC flooding enables traffic visibility beyond intended switching behavior
* Plaintext reverse shells expose full command and output streams
* MITM attacks can escalate from passive sniffing to active command injection
* Credentials and shell sessions become fully observable in plaintext protocols

---

### Passive vs Active MITM

**Passive MITM**

* Observes communication
* Extracts credentials and sensitive data
* No modification of traffic

**Active MITM**

* Modifies packets inline
* Injects or replaces commands
* Alters application behavior

---

### Mitigations

* Encrypt reverse shell traffic (SSH, TLS tunnels)
* Use VPN before establishing remote shell
* Enable switch security controls (Port Security, Dynamic ARP Inspection)
* Avoid plaintext authentication mechanisms

---

### Conclusion

This lab highlights the critical risk of plaintext interactive protocols in Layer 2 attack scenarios.
Once MITM access is achieved, attackers may transition from passive monitoring to active command manipulation, gaining control over privileged sessions.
