# System Call

```
 //linux/arch/arm64/kernel # vi entry-common.c
649 asmlinkage void noinstr el0t_64_sync_handler(struct pt_regs *regs)
650 {
651         unsigned long esr = read_sysreg(esr_el1);
652 
653         switch (ESR_ELx_EC(esr)) {
654         case ESR_ELx_EC_SVC64:
655                 el0_svc(regs);
656                 break;
657         case ESR_ELx_EC_DABT_LOW:
658                 el0_da(regs, esr);
659                 break;
660         case ESR_ELx_EC_IABT_LOW:
661                 el0_ia(regs, esr);
662                 break;
663         case ESR_ELx_EC_FP_ASIMD:
664                 el0_fpsimd_acc(regs, esr);
665                 break;
666         case ESR_ELx_EC_SVE:
667                 el0_sve_acc(regs, esr);

//Transfer register adderess to do_e10_svc
633 static void noinstr el0_svc(struct pt_regs *regs)
634 {
635         enter_from_user_mode(regs);
636         cortex_a76_erratum_1463225_svc_handler();
637         do_el0_svc(regs);
638         exit_to_user_mode(regs);
639 }

//linux/arch/arm64/kernel # vi syscall.c
203 void do_el0_svc(struct pt_regs *regs)
204 {
205         fp_user_discard();
206         el0_svc_common(regs, regs->regs[8], __NR_syscalls, sys_call_table); //__NR_syscalls is '451'
207 }

 81 static void el0_svc_common(struct pt_regs *regs, int scno, int sc_nr,
 82                            const syscall_fn_t syscall_table[])
 83 {
 84         unsigned long flags = read_thread_flags();
 85 
 86         regs->orig_x0 = regs->regs[0];
 87         regs->syscallno = scno;

//system call
147         invoke_syscall(regs, scno, sc_nr, syscall_table);
```
## test1
```
19837      syscalltest-6013    [002] d.... 57262.926612: do_el0_svc+0x4/0xd0 <-el0_svc+0      x2c/0x84
19838      syscalltest-6013    [002] d.... 57262.926616: <stack trace>
19839  => do_el0_svc+0x8/0xd0
19840  => el0_svc+0x2c/0x84
19841  => el0t_64_sync_handler+0xb8/0xc0
19842  => el0t_64_sync+0x18c/

20473      syscalltest-6013    [002] ..... 57262.928538: do_sys_openat2+0x4/0x174 <-__ar      m64_sys_openat+0x6c/0xb4
20474      syscalltest-6013    [002] ..... 57262.928540: <stack trace>
20475  => do_sys_openat2+0x8/0x174
20476  => __arm64_sys_openat+0x6c/0xb4
20477  => invoke_syscall+0x50/0x120
20478  => el0_svc_common.constprop.0+0x4c/0xf4
20479  => do_el0_svc+0x34/0xd0
20480  => el0_svc+0x2c/0x84
20481  => el0t_64_sync_handler+0xb8/0xc0
20482  => el0t_64_sync+0x18c/0x190

```

##test2
```
//linux/arch/arm64/kernel # vi syscall.c
//set printk
 82 static void el0_svc_common(struct pt_regs *regs, int scno, int sc_nr,
 83                            const syscall_fn_t syscall_table[])
 84 {
 85         unsigned long flags = read_thread_flags();
 86 
 87         regs->orig_x0 = regs->regs[0];
 88         regs->syscallno = scno;
 89 
 90         if(7989 == raspbian_debug_state){
 91                 printk(KERN_INFO "AHN_System call number: %d\n", scno);
 92                 dump_stack();
 93 
 94         }

//result
/sys/kernel/debug/rpi_debug # dmesg | grep AHN
[  122.352307] AHN_System call number: 63
[  122.352358] AHN_System call number: 64
[  122.352399] AHN_System call number: 98
[  122.352440] AHN_System call number: 98
[  122.352493] AHN_System call number: 178

//system call number check
//
205 #define __NR_read 63
206 __SYSCALL(__NR_read, sys_read)
207 #define __NR_write 64
208 __SYSCALL(__NR_write, sys_write)
321 #define __NR_futex 98
322 __SC_3264(__NR_futex, sys_futex_time32, sys_futex)
535 #define __NR_sysinfo 179
536 __SC_COMP(__NR_sysinfo, sys_sysinfo, compat_sys_sysinfo)

```