# linux

## todos

- python: file io, argparse, main, (maybe: os stuff, threading)
- grep/awk/sed

## really nice tidbits:

- `grep -E "HEADER_PATTERN|SEARCH_PATTERN"` is a great way to find what you're looking for, and include the header line!!!
- `man` helpful scrolling tips:
    - Press `Space` to go down one page, `b` to go back one page.
    - Press `u` to go up half a page, `d` to go down half a page.
    - Press `g` to go to the beginning of the document, `G` to go to the end.
    - Press `/keyword` to search for "keyword" forward, `?keyword` to search backward.
    - Press `n` to go to the next search result, `N` to go to the previous search result.
    - Press `q` to quit.

## things i wanna know well

- threads vs processes
- fork vs exec
- inodes
- file system
- boot
- setuid, setgid
- tmux


## The major timer-based counter tools

| Tool     | Focus area           | Typical use                              |
|----------|----------------------|------------------------------------------|
| iostat -x 1 | Block I/O devices    | Disk latency, throughput, %util               |
| pidstat 1   | Tasks (per-PID)     | CPU, memory, context switches per process  |
| mpstat 1    | CPUs / cores        | CPU usage per processor or aggregate       |
| vmstat 1    | Memory, processes, I/O | Paging, run queue, swap, I/O waits         |
| sar -n DEV 1 | Network interfaces  | RX/TX packets, throughput                 |
| sar -u 1    | CPU utilization (like mpstat) | User/system/idle breakdown              |
| sar -d 1    | Block devices (like iostat) | Disk throughput and latency                 |