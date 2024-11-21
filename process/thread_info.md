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