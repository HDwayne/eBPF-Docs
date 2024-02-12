# Examples

## Inter-process Communication

Attach a program to an XDP-event. Transformed C sources code into eBPF bytecode and then compiled to machine code. Load the program in the kernel and attach it to event.

hello.bpf.c
```c
#include <linux/bpf.h>
#include <bpf/bpf_helpers.h>

int counter = 0;

SEC("xdp")
int hello(struct xdp_md *ctx) {
    bpf_printk("Hello World %d", counter);
    counter++; 
    return XDP_PASS;
}

char LICENSE[] SEC("license") = "Dual BSD/GPL";

```

hello-func.bpf.c

```c
#include <linux/bpf.h>
#include <bpf/bpf_helpers.h>

static __attribute((noinline)) int get_opcode(struct bpf_raw_tracepoint_args *ctx) {
    return ctx->args[1];
}

SEC("raw_tp/")
int hello(struct bpf_raw_tracepoint_args *ctx) {
    int opcode = get_opcode(ctx);
    bpf_printk("Syscall: %d", opcode);
    return 0;
}

char LICENSE[] SEC("license") = "Dual BSD/GPL";
```

- Compilation in eBPF bytecode \
make

- Loading the program in the kernel\
sudo bpftool prog load hello.bpf.o /sys/fs/bpf/hello

(hello is the program's name)

- Check if the program is in the kernel \
ls /sys/fs/bpf

- Some information of the program \
sudo bpftool prog list

- Attach the program to an event \
sudo bpftool net attach xdp id 540 dev lo

- Check the output \
sudo cat /sys/kernel/debug/tracing/trace_pipe

- Detach the program \
sudo bpftool net detach xdp dev lo

- Remove the program to the kernel \
sudo rm /sys/fs/bpf/hello



eBPF program :

```c

#include "vmlinux.h"
#include <bpf/bpf_helpers.h>
#include <bpf/bpf_core_read.h>
#include "simple.h"

char LICENSE[] SEC("license") = "GPL";


int count=0;


struct {
    __uint(type, BPF_MAP_TYPE_HASH);
    __uint(max_entries, 40000);
    __type(key, u32);
    __type(value, struct typetest);
} my_map SEC(".maps");
	


SEC("tp/signal/signal_deliver")
int test(void *params)
{
    
    struct typetest t;
    t.pid =  bpf_get_current_pid_tgid() >> 32;
    t.count = count;
    t.uid = bpf_get_current_uid_gid() & 0xFFFFFFFF;
    if( ! bpf_map_lookup_elem(&my_map,&t.pid) ){
        bpf_map_update_elem(&my_map,&t.pid,&t,BPF_ANY);
    }
    count ++;
    return 0;
}

```


User space program :


```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/resource.h>
#include <unistd.h>
#include "simple.h"
#include "simple.skel.h"


void print_map_array(struct simple *skel){

	struct typetest t;
	int *cur_key= NULL;
	int next_key;
	int err;

	if( (err = bpf_map__get_next_key(skel->maps.my_map, cur_key, &next_key,sizeof(int)))==0){

		printf("-----------------------------------------------------------------------------------\n");
		for (;err == 0;) {

                bpf_map__lookup_elem(skel->maps.my_map, &next_key,sizeof(int), &t,sizeof(struct typetest),BPF_ANY);
				printf("PID = %d,  count = %d  , UID = %d\n",t.pid,t.count,t.uid);


                cur_key = &next_key;
				err = bpf_map__get_next_key(skel->maps.my_map, cur_key, &next_key,sizeof(int));
    	}
		printf("-----------------------------------------------------------------------------------\n");
	}
}

int main(void)
{

    struct simple *skel = simple__open();
    simple__load(skel);
    simple__attach(skel);


	while(true){
		sleep(1);
		print_map_array(skel);
	}

    return 0;
}

```

Marche bien mais je conseille de tester en changeant d'évènement

+ structure de données utilisé ( à mettre dans un .h ):

```c
struct typetest {
    int pid;
    int count;
    int uid;
};
```

en gros, le programme affiche le contenu de la Hash-map toute les 1 seconde
( la Hash-map contient de éléments de type struct typeset)



