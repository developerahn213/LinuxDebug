# ContextSwitch
## Context Switch is defined at linux/kernel/sched/core.c
```
Context_switch(struct rq *rq, struct task_struct *prev,
 5199                struct task_struct *next, struct rq_flags *rf)
 5200 {
    ///
 5250         switch_to(prev, next, prev); //We follow this function
 5251         barrier();
 5252 
 5253         return finish_task_switch(prev);
```

### define switch_to
```
//linux/include/asm-generic
//this is macro function
 18 extern struct task_struct *__switch_to(struct task_struct *,
 19                                        struct task_struct *);
 20 
 21 #define switch_to(prev, next, last)                                     \
 22         do {                                                            \
 23                 ((last) = __switch_to((prev), (next)));                 \
 24         } while (0)
```

### last = cpu_switch_to
```
//linux/arch/arm64/kernel/process.c
517 __notrace_funcgraph __sched
518 struct task_struct *__switch_to(struct task_struct *prev,
519                                 struct task_struct *next)
520 {
521         struct task_struct *last;
    ///
550         /* the actual thread switch */
551         last = cpu_switch_to(prev, next);
```

### SYM_FUNC_START
```
//linux/arch/arm64/kernel/entry.S
//defined in assembly code. each x10, x8 etc mean register.
 829 SYM_FUNC_START(cpu_switch_to)
 830         mov     x10, #THREAD_CPU_CONTEXT //mov means copy, stores struct cpu_context's address
 831         add     x8, x0, x10
 832         mov     x9, sp
 833         stp     x19, x20, [x8], #16             // store callee-saved registers
 834         stp     x21, x22, [x8], #16
 835         stp     x23, x24, [x8], #16
 836         stp     x25, x26, [x8], #16
 837         stp     x27, x28, [x8], #16
 838         stp     x29, x9, [x8], #16
 839         str     lr, [x8] //lr stores ruturn address
 840         add     x8, x1, x10
 841         ldp     x19, x20, [x8], #16             // restore callee-saved registers
 842         ldp     x21, x22, [x8], #16
 843         ldp     x23, x24, [x8], #16
 844         ldp     x25, x26, [x8], #16
 845         ldp     x27, x28, [x8], #16
 846         ldp     x29, x9, [x8], #16
 847         ldr     lr, [x8]
 848         mov     sp, x9
 849         msr     sp_el0, x1
 850         ptrauth_keys_install_kernel x1, x8, x9, x10
 851         scs_save x0
 852         scs_load_current
 853         ret
 854 SYM_FUNC_END(cpu_switch_to)
 855 NOKPROBE(cpu_switch_to)
```

```
//Code of THREAD_CPU_CONTEXT
//linux/arch/arm64/kernel # vi asm-offsets.c
47   DEFINE(THREAD_CPU_CONTEXT,    offsetof(struct task_struct, thread.cpu_context));
```

### stuct task_struct at THREAD_CPU_CONTEXT
```
//linux/include/linux/sched.h
737 struct task_struct {
 738 #ifdef CONFIG_THREAD_INFO_IN_TASK
 739         /*
 740          * For reasons of header soup (see current_thread_info()), this
 741          * must be the first element of task_struct.
 742          */
 743         struct thread_info              thread_info;
```

### thread.cpu_context at THREAD_CPU_CONTEXT
```
//linux/arch/arm64/include/asm/processor.h
125 struct cpu_context {
126         unsigned long x19;
127         unsigned long x20;
128         unsigned long x21;
129         unsigned long x22;
130         unsigned long x23;
131         unsigned long x24;
132         unsigned long x25;
133         unsigned long x26;
134         unsigned long x27;
135         unsigned long x28;
136         unsigned long fp;
137         unsigned long sp;
138         unsigned long pc;
139 };
140 
141 istruct thread_struct {
142         struct cpu_context      cpu_context;    /* cpu context */
/
```