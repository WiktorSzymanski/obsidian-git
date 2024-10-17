---
class: KSCh
---
# Unikraft
---
- *micro-liblary operating system*
- modularyzacja jądra SO
	- *memory allocators*
	- *schedulers*
	- *network stacks*
- dobór komponentów do potrzeb (API jako *micro-library*)
- biblioteka `musl libc` + *POSIX syscall shim layer micro-liblary*
- #TODO