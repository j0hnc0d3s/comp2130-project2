# SIGNR — Bomber Squadron Simulation

> A multi-process bomber-plane simulation in C using Unix signals for command-and-control IPC.

Built for **COMP2130 — Operating Systems / Systems Programming** at the University of the West Indies, Mona.

## Overview

SIGNR is a command-line simulation of an air-force bomber squadron. The user (command base) can launch new bomber planes, signal individual planes to drop bombs, refuel them mid-flight, or terminate the simulation. Each plane is a separate Unix process forked from the main controller, and command-and-control between base and planes is handled entirely through **Unix signals** — a creative framing for what is fundamentally a signal-based IPC exercise.

## How It Works

- **The base process** runs the user command loop and maintains an array of active plane PIDs (`planes[MAX_PLANES]`).
- **Each plane** is a child process created with `fork()`. Planes start at 100% fuel and consume 5% per second, reporting status back to base every 3 seconds. When fuel reaches 0, the plane crashes (`exit`).
- **Commands from base to planes** are delivered as signals via `kill()`:
  - `SIGUSR1` → drop bomb
  - `SIGUSR2` → refuel
  - `SIGTERM` → terminate (used during shutdown)
- **Child planes** register their own handlers for the above signals, plus `SIGCHLD` for cleanup.
- **On `q`,** the base sends `SIGTERM` to all active planes and `waitpid()`s on each one to ensure clean termination.

## Commands

| Command | Action |
|---|---|
| `l` | Launch a new bomber plane |
| `b` | Drop a bomb from a specified plane (1-indexed) |
| `r` | Refuel a specified plane (1-indexed) |
| `q` | Quit the simulation (terminates all planes cleanly) |

## Concepts Demonstrated

- Process creation with `fork()`
- Inter-process communication via Unix signals (`SIGUSR1`, `SIGUSR2`, `SIGTERM`, `SIGCHLD`)
- Signal handler registration with `signal()`
- Sending signals between processes using `kill()` and PIDs
- Parent-side child reaping with `waitpid()`
- Shared state between user input loop and child processes

## Tech Stack

- **Language:** C
- **Platform:** POSIX Unix (Linux, macOS)
- **System APIs:** `<signal.h>`, `<unistd.h>`, `<sys/types.h>`, `<sys/wait.h>`, `<string.h>`

## Build

```bash
gcc -Wall -Wextra -o signr signr.c
```

## Run

```bash
./signr
```

Example session:
```
Enter command l (for launch), b (for bomb), r (for refuel), and q (for quit): l
Plane 12345 has been launched
Enter command l (for launch), b (for bomb), r (for refuel), and q (for quit): b
Which plane should drop a bomb?
1
Bomber 12345 to base, bombs away!
```

Built by [Josiah-John Green](https://github.com/j0hnc0d3s) — Software Engineering, UWI Mona '26.
