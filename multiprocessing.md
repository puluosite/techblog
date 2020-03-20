# C++ MultiProcessing Developing Notes
1. [GDB Commands](#GDB-Commands)


## GDB Commands
```
set follow-fork-mode parent/child # stick to parent or child process
set detach-on-fork on/off # on: one process is in GDB, others run freely; off: one process in GDB, others suspended

info inferiors # show all processes
inferiors xxx # select inferior
detach inferiors xxx # to detach one process
kill inferiors xxx # to kill one process
```

What I will do usually is to first turn off detach-on-fork and follow parent. 
Hit the place where I want to debug the MP session. Then enable the break points, which will be hit 
in child process. Then, set detach-on-fork off, and follow-fork-mode child. And let it hit.
