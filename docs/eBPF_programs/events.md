IT WILL CONTAIN ALL EVENTS AVAILABLE AND RELEVANT TO THE PROJECT


POSSIBLE :

- sysinfo : This system call is used by the '-m' option of Mojito/S to retrieve memory information. Two events are associated with it: sys_exit_sysinfo and sys_enter_sysinfo. They are enabled by default and are generated regularly by the system.
- sched : To access scheduling infos

- vmalloc and kmalloc: To access memory allocations infos

- percpu : To access memory allocations info for cpu ( need to be verified )

IMPOSSIBLE : 
