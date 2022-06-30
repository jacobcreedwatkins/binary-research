# binary-research
Work day research related to binaries used in the PCTE


### Objective
- Analyze a linux binary to determine the following:- Purpose of binary- Functions of binary- Kernel Modules used- External connections and/or attempts to connect- Files utilized/written to

```
Programs used:
  gdb - gnu debugger
  strace - system call tracer
  lsof - list open files
Binary executables on most Linux distributions will have the types OUT, COFF, or ELF; a software package that supports other executable file types could however run other executables. For example, MS-DOS programs can be executed if the Operating System has Wine installed.
  To find out the file's type: file [program]
Binary files are by and large found in a */bin/ directory. By default, they can be found in the $PATH environment variable, or:
  /usr/local/sbin, /usr/local/bin, /usr/sbin, /usr/bin, /sbin, /bin, /usr/games
Finding out information on a Linux binary, such as what it is, what it does, and how to use it, requires use of local documentation (man pages, documents in the same directory, source code), reverse engineering, or searching the internet.
  man [program]
  (Note that searching an executable by name on the internet isn't 100% reliable without checksums)
Finding kernel modules for a program can be done using gdb. This returns all shared libraries in use for the running time of the program:
  sudo gdb [program]
  start
  info sharedlibrary
Finding out whether or not a program interacts with the network can be done through lsof, ss, or netstat:
  sudo lsof -i, ss -ntlp, netstat -antl
To constantly monitor a program as it interacts with the network, or examine network interactions of a preexisting program, you can use strace:
  To run the program and check for network interaction: strace -e network [program]
  To monitor an program already running: strace -e network ip [process id]
Finding any file interactions, such as general I/O or file locks, can be viewed via lsof or strace.
  To find out if a program is currently interacting with files, you can use lsof:
    sudo lsof | egrep "*[program]"
    sudo lsof | egrep "[process_id]"
  To find out if a program interacts with files during execution, you can use strace:
    strace -e trace=file -f -o mytrace.log [program]
  If you want to monitor a currently running program:
    strace -e trace=file -f -o mytrace.log -p [process_id]
(Note that just because a program was executed, doesn't mean it will interact with a file. Interaction may be required with the program in order to correctly monitor file I/O. For this, it is recommended that you use multiple avenues of monitoring for file interaction. This also applies to network interaction. When possible, simulate interaction with the program during monitoring)
STRACE SYSCALLS
  Identifying what a program is doing via strace requires reading linux system call functions. The following functions are notable:
  FILES
  1. openat() - opens a file in a certain mode (read,write,both).
    File Modes:
      O_RDONLY - Open for reading only.
      O_WRONLY - Open for writing only.
      O_RDWR - Open for reading and writing.
      O_CREAT - If the file already exists, do nothing. If it doesn't exist, create the file.
      O_EXCL - Use with O_CREAT. Creates an error if the file does exist.
    EX) openat(AT_FDCWD, "file.txt", O_RDONLY|O_CREAT|O_EXCL, 0600) = open file.txt, which is found in the current working directory, in readonly mode. If there is no file named file.txt, create it, and it must be created now or else it fails, with the permissions 0600.
  2. unlink() - remove a file from the filesystem
    EX) unlink("/home/me/file1.txt") = delete file1.txt from /home/me directory.
 NETWORK
  accept() - Accept a connection from a requesting socket. This means we are a server allowing connections from a client.
  connect() - Connects to another socket. This means we are a client connecting to a server.
  bind() - Bind to the created socket. This indicates that the program is a server.
  listen() - Listen on the binded socket. This means the program is a server and listening for connections from a client.
  socket() - Create a socket. This means the network is being interacted with.
  The first argument is the domain that the socket is being created over.
    AF_UNIX is local to the machine, such as different processes communicating with each other
    AF_INET or AF_INET6 means the program is reaching out over the internet.
EXAMPLE
  Finding information on a binary
  what it is, what it does, how to use it
  use of local documentation/source/reverse engineering, man pages, internet search.
Finding kernel modules used by a binary
    sudo gdb [program]
    start
    (if prompted for setting breakpoints, reply appropriately)
    info sharedlibrary
  The list given will provide the shared libraries that the application uses.
Finding whether or not a program is interacting with the network
  sudo lsof -i or netstat -ano or sudo ss -lnrt
  cat /proc/net/tcp
  Constantly monitor, or check by running the program:
  strace -e network [program]
  strace -e network -p [process_id]
Finding files that a program utilizes
  sudo lsof | egrep "*[program]"
  sudo lsof | egrep "[process_id]"
  strace -e trace=file -f -o mytrace.log [program]
  strace -e trace=file -f -o mytrace.log -p [process_id]
3:03
Analyze a linux binary to determine the following:
- Purpose of binary
- Functions of binary
- Kernel Modules used
- External connections and/or attempts to connect
- Files utilized/written to
Linux Binary:
most default binaries: /bin, /sbin, /usr/bin, ...  binaries by and large located in "*bin" directory
Types:  ELF - Executable and Linkable Format (widely adopted)  OUT - Assembler OUTput Format (seldom used)  MS-DOS (windows), COFF (replaced OUT) runnable on supporting distros
Finding information on a binary
what it is, what it does, how to use it  use of local documentation/source/reverse engineering, man pages, internet search.
Finding kernel modules used by a binary
sudo gdb [program]
start    (if prompted for setting breakpoints, reply appropriately)
info sharedlibrary
The list given will provide the shared libraries that the application uses.
Finding whether or not a program is interacting with the network
sudo lsof -i or netstat -ano or sudo ss -lnrt
cat /proc/net/tcp
Constantly monitor, or check by running the program:
strace -e network [program]
strace -e network -p [process_id]
Finding files that a program utilizes
sudo lsof | egrep "*[program]"
sudo lsof | egrep "[process_id]"
strace -e trace=file -f -o mytrace.log [program]
strace -e trace=file -f -o mytrace.log -p [process_id] (edited) 
```

# Things that I have done

`> sudo apt install nmap`

- this will actually install nmap to your machine 

`> which nmap`

- shows me where nmap installed on my device (in my case /usr/bin/nmap - the /usr/bin will be the default installation path for user binaries or programs that are "ready-to-run" by normal users)

`> file nmap`

- shows me basic information about the nmap binary. output of this command below:

`> nmap: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=abeaeb9c57cd40a2fca33be55267d325a72233b7, for GNU/Linux 3.2.0, stripped
`

## OSINT (google)

[nmap.org website](https://nmap.org/)

- nmap's official website is a treasure trove of information all about nmap
- the most important section is the reference guide, containing syntax examples, options and their functions, and other important information regarding the tool's usage

## Kernel Modueles

- To find the kernel modules being used by nmap, I used a tool known as GDB (GNU DeBugger)
- What does GNU actually stand for? "GNU's Not Unix", kind of funny history tidbit there.

Commands run:
```
> sudo gdb nmap
> start
> info sharedlibrary
```

- Output of `info sharedlibrary`


### What is happening here?

1. first we open up gdb with super user permissions and run nmap
2. `start` is self explanatory, we start the debugger
3. `info sharedlibrary` shows information about the loaded libraries, their addresses and state of the debugging symbols.


#### Shared vs Dynamic Libraries (additional research)
website I got the above information from: [visualgdb.com](https://visualgdb.com/gdbreference/commands/sharedlibrary)

- from u/Codo on [stackexchange](https://stackoverflow.com/questions/56899488/what-is-the-difference-between-shared-and-dynamic-libraries-in-c): "In the shared library case, you specify the shared library at compile-time. When the app is started, the operating system will load the shared library before the application starts. In the dynamic libary case, the library is not specified at compile-time, so it's not loaded by the operating system. Instead, your application will contain some code to load the library."

- in my own words: with a shared libraries, you are specifying the libraries to be used by the program at compile-time. Once the program is run, the operating system will load the shared library before the application actually starts.

- with dynamic linked libraries, the library is not specified at compile-time. instead, the application will contain some code to load the library. essentially these are two ways of performing the same task that are ever so slightly different.


- actual code output of command:
- ![image](https://user-images.githubusercontent.com/100236631/176528450-25bb1223-d8f4-4983-917e-d07038c3354b.png)

Here underneath the "shared object library" column, we can see every single shared library being used by nmap, along with the memory addresses they are found at (From / To) and the symbols being read 

#### strace
- strace is used to find system calls being performed by the binary


`> strace -e trace=file -f -o mytrace.log nmap`

![image](https://user-images.githubusercontent.com/100236631/176678939-61c3028c-3d08-4a76-b4af-360e71617cf4.png)


### Networkk Information

![image](https://user-images.githubusercontent.com/100236631/176679459-02f3c5c3-46ab-44b7-b8d8-4d519649bdf0.png)

