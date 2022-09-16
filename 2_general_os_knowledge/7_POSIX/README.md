# POSIX Basics

POSIX (Portable Operating System Interface) is a family of standards for maintaining compatibility between operating systems. It describes utilities, APIs, and services that a compliant OS should provide to software, thus making it easier to port programs from one system to another.

A practical example: in a Unix-like operating system, there are three standard streams, `stdin`, `stdout` and `stderr` - they are I/O connections that you will probably come across when using a terminal, as they manage the flow from the **standard input** (stdin), **standard output** (stdout) and **standard error** (stderr).

So, in this case, when we want to interact with any of these streams (through a process, for example), the POSIX operating system API makes it easier - for example, in the `<unistd.h>` C header where the stdin, stderr, and stdout are defined as `STDIN_FILENO`, `STDERR_FILENO` and `STDOUT_FILENO`.

## Most important things POSIX 7 defines

1. ### [C API](https://pubs.opengroup.org/onlinepubs/9699919799/functions/contents.html)

   Greatly [extends ANSI C](https://stackoverflow.com/questions/9376837/difference-between-c-standard-library-and-c-posix-library/37969420#37969420) with things like:

   - more file operations: `mkdir`, `dirname` `symlink`, `readlink`, `link` (hardlinks), `poll`, `stat` `sync`, `ntfw()`
   - process and threads: `fork`, `execl`, `wait`, `pipe`, semaphors, `sem *`, shared memory (`shm_*`), `kill`, scheduling params (`nice`, `sched_*`), `sleep`, `mkfifo`, `setpgid()`
   - networking: `socket()`
   - memory management: `mmap`, `mlock`, `mprotect`, `madvise`, `brk()`
   - utilities: regular expressions (`reg*`)

   Those APIs also determine underlying system concepts on which they depend, e.g `fork` requires a concept of a process

   Many [Linux system calls](https://unix.stackexchange.com/questions/421750/where-do-you-find-the-syscall-table-for-linux) exist to implement a specific POSIX C API function and make Linux compliant, e.g. sys_write, sys_read, ... Many of those syscalls also have Linux-specific extensions however.

   Major Linux desktop implementation: glibc, which in many cases just provides a shallow wrapper to system calls.

2. ### [CLI utilities](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/contents.html)

   E.g.: `cd`,`ls`,`echo`...

   Many utilities are direct shell front ends for a corresponding C API function, e.g `mkdir`.

   Major Linux desktop implementation: GNU Coreutils for the small ones, separate GNU projects for the big ones: `sed`, `grep`, `awk`, ... Some CLI utilities are implemented by Bash as [built-ins](https://unix.stackexchange.com/questions/11454/what-is-the-difference-between-a-builtin-command-and-one-that-is-not).

3. ### [Shell language](Shell language)
   E.g., `a=b; echo "$a"`
   GNU bash
4. ### [Environment values](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap08.html#tag_08)

   E.g.: `HOME`, `PATH`.

5. ### [Program exit status](http://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html#tag_18_08)

   ANSI C says 0 or `EXIT_SUCCESS` for success, `EXIT_FAILURE` for failure, and leaves the rest implementation defined.

   POSIX adds:

   - `126`: command found but not executable.
   - `127`: command not found.
   - `> 128`: terminated by a signal.

   But POSIX does not seem to specify the `128 + SIGNAL_ID` rule used by Bash: Default [exit code when process is terminated?](https://unix.stackexchange.com/questions/99112/default-exit-code-when-process-is-terminated)

6. ### [Regular expression](http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap09.html#tag_09)

   There are two types: BRE (Basic) and ERE (Extended). Basic is deprecated and only kept to not break APIs.

   Those are implemented by C API functions, and used throughout CLI utilities, e.g. `grep` accepts BREs by default, and EREs with `-E`.

   E.g.: `echo 'a.1' | grep -E 'a.[[:digit:]]'`

   Major Linux implementation: glibc implements the functions under regex.h which programs like `grep` can use as backend.

7. ### [Directory struture](http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap10.html#tag_10)

   E.g.: `/dev/null`, `/tmp`
   The Linux FHS greatly extends POSIX.

8. ### Filenames

- `/` is the path separator
- `NUL` cannot be used
- `.` is `cwd`, `..` parent
- portable filenames
  - use at most max 14 chars and 256 for the full path
  - can only contain: a-zA-Z0-9.\_-

9. ### [Command line utility API conventions](http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap12.html)

   Not mandatory, used by POSIX, but almost nowhere else, notably not in GNU. But true, it is too restrictive, e.g. single letter flags only (e.g. `-a`), no double hyphen long versions (e.g. `--all`).

   A few widely used conventions:

   - `-` means stdin where a file is expected
   - `--` terminates flags, e.g. `ls -- -l` to list a directory named `-l`

10. ### "POSIX ACLs" (Access Control Lists), e.g. as used as backend for [`setfacl`](https://askubuntu.com/questions/487527/give-specific-user-permission-to-write-to-a-folder-using-w-notation/809562#809562).

    This was [withdrawn](https://unix.stackexchange.com/questions/489820/why-was-posix-1e-withdrawn) but it was implemented in several OSes, [including in Linux with setxattr](https://www.linuxquestions.org/questions/linux-kernel-70/system-call-to-set-file-acls-in-linux-537615/#post5988355).
