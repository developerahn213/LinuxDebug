# Process Stack
### stacked from the high number address to the low number address

### thread_info is located on the top of the stack
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

### The size of stack is 0x4000(16kb) in the arm64

# preempt_count
1. Set interrupt context execution or end<br>
in_interrupt()==ture //doing interrupt context

2. Set soft IRQ context execution or end<br>
in_softirq()==ture //doing soft IRQ context

3. Check weather a process is preemptible<br>
0 => preemptible

# smp_processor_id() //check the operating CPU

# thread_info initialization
```
//linux/kernel # vi fork.c

//When process is made, copy_process is executed.
2002 static __latent_entropy struct task_struct *copy_process(
2003                                         struct pid *pid,
2004                                         int trace,
2005                                         int node,
2006                                         struct kernel_clone_args *args)

//When call copy_process, below functions is called
2098         p = dup_task_struct(current, node);

//Allocate task_struct
 980         tsk = alloc_task_struct_node(node); //general
 981         if (!tsk)
 982                 return NULL;
 983 
 984         err = arch_dup_task_struct(tsk, orig); //associated with architecture
 985         if (err)
 986                 goto free_tsk;
 987 
 988         err = alloc_thread_stack_node(tsk, node);
 989         if (err)
 990                 goto free_tsk;

//Make the stack area
 311         stack = __vmalloc_node_range(THREAD_SIZE, THREAD_ALIGN,
 312                                      VMALLOC_START, VMALLOC_END,
 313                                      THREADINFO_GFP & ~__GFP_ACCOUNT,
 314                                      PAGE_KERNEL,
 315                                      0, node, __builtin_return_address(0));
====
 329         tsk->stack_vm_area = vm;
 330         stack = kasan_reset_tag(stack);
 331         tsk->stack = stack;
 332         return 0;
====
//Returns a pre-constructed task structure.
 169 static inline struct task_struct *alloc_task_struct_node(int node)
 170 {
 171         return kmem_cache_alloc_node(task_struct_cachep, GFP_KERNEL, node);
```

# Current
Macro function to access the process's task descriptor
```
linux/arch/arm64/include/asm # vi current.h
 15 static __always_inline struct task_struct *get_current(void)
 16 {
 17         unsigned long sp_el0; //The address of the task_struct will be started next
 18 
 19         asm ("mrs %0, sp_el0" : "=r" (sp_el0));
 20 
 21         return (struct task_struct *)sp_el0;
 22 }
 23 
 24 #define current get_current()
```
