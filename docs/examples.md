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
