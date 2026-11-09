# Vulnserver Buffer Overflow Fuzzing Lab

Notes and fuzzing scripts from a hands-on buffer overflow exercise against
**Vulnserver**, a deliberately vulnerable Windows TCP server, using the
**Sulley fuzzing framework** on Parrot Security OS.

## Overview

This repo documents the process of:
- Connecting to Vulnserver and enumerating its commands
- Writing `.spk` Sulley fuzzing scripts for individual commands (e.g. `STATS`, `TRUN`)
- Running `generic_send_tcp` to fuzz each command
- Monitoring the target process in **Immunity Debugger**
- Identifying which command(s) crash the server and overwrite the **EIP register**

## Environment

| Component      | Details                          |
|-----------------|-----------------------------------|
| Attacker         | Parrot Security OS (VM)          |
| Target            | Windows 10/11 (VM)                |
| Vulnerable App | Vulnserver (`vulnserver.exe`)   |
| Debugger          | Immunity Debugger                 |
| Fuzzer              | Sulley (`generic_send_tcp`)     |

## Fuzzing Script Template

Each command gets its own `.spk` file, following this pattern:

\`\`\`
s_readline();
s_string("COMMANDNAME ");
s_string_variable("0");
\`\`\`

## Workflow

1. Enumerate server commands via `nc <target-ip> <port>` → `HELP`
2. Create a `.spk` fuzz script per command
3. Run the fuzzer:
   \`\`\`
   generic_send_tcp <target-ip> <port> <command>.spk 0 0
   \`\`\`
4. Check Immunity Debugger:
   - **Running** → not vulnerable, move to next command
   - **Paused / crashed (e.g. EIP = 41414141)** → vulnerable function found

## Results

| Command | Result            |
|---------|--------------------|
| STATS  | Not vulnerable (process stayed Running) |
| TRUN   | **Vulnerable** — crashed, EIP overwritten with `41414141` (A's) |

## Disclaimer

This is a lab exercise conducted in an isolated virtual environment for
educational purposes (CEH-style training). Do not run these techniques
against systems you don't own or have explicit permission to test.
