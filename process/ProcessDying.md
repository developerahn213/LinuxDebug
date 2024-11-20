# In user level

## 2 flows
1. call 'exit()' function from application
exit()->sys_exit_group[__arm64_sys_exit_group]->do_group_exit->do_exit()

2. get kill signal
kill->do_notify_resume->get_signal->do_group_exit

## call exit()

### Check whether 'do_exit' funtion can use ftrace or not
```
/sys/kernel/debug/tracing # cat available_filter_functions | grep do_exit
do_exit 
/sys/kernel/debug/tracing # 
//posible
```

### ftrace setting
```
 echo do_exit exit_group > /sys/kernel/debug/tracing/set_ftrace_filter
 sleep 1
 echo "set ftrace filter enabled"
 //Add do_exit function and exit_group function
```

### execution
```
./set_ftrace.sh
/project/test procTest2
./get_ftrace.sh do_exit1
```

### test result
```
24213        procTest2-2260    [001] ..... 17499.596429: __arm64_sys_exit_group+0x4/0x20       <-invoke_syscall+0x50/0x120
24214        procTest2-2260    [001] ..... 17499.596436: <stack trace>
24215  => __arm64_sys_exit_group+0x8/0x20
24216  => invoke_syscall+0x50/0x120
24217  => el0_svc_common.constprop.0+0x4c/0xf4
24218  => do_el0_svc+0x34/0xd0
24219  => el0_svc+0x2c/0x84
24220  => el0t_64_sync_handler+0xb8/0xc0
24221  => el0t_64_sync+0x18c/0x190
24222        procTest2-2260    [001] ..... 17499.596441: do_exit+0x4/0x9a4 <-do_group_ex      it+0x3c/0xa0
24223        procTest2-2260    [001] ..... 17499.596443: <stack trace>
24224  => do_exit+0x8/0x9a4
24225  => do_group_exit+0x3c/0xa0
24226  => __wake_up_parent+0x0/0x40
24227  => invoke_syscall+0x50/0x120
24228  => el0_svc_common.constprop.0+0x4c/0xf4
24229  => do_el0_svc+0x34/0xd0
24230  => el0_svc+0x2c/0x84
24231  => el0t_64_sync_handler+0xb8/0xc0
24232  => el0t_64_sync+0x18c/0x190
```

### addition information
```
// At project/linuxSrc/out folder, input 'objdump -D vmlinux > tmp.txt
 151383 ffffffc0080911a0 <__arm64_sys_exit_group>:
 151384 ffffffc0080911a0:       d503201f        nop
 151385 ffffffc0080911a4:       d503201f        nop
 151386 ffffffc0080911a8:       d503233f        paciasp
 151387 ffffffc0080911ac:       a9bf7bfd        stp     x29, x30, [sp, #-16]!
 151388 ffffffc0080911b0:       910003fd        mov     x29, sp
 151389 ffffffc0080911b4:       f9400000        ldr     x0, [x0]
 151390 ffffffc0080911b8:       53181c00        ubfiz   w0, w0, #8, #8
 151391 ffffffc0080911bc:       97ffffd1        bl      ffffffc008091100 <do_group_exit>
 151392 
 151393 ffffffc0080911c0 <__wake_up_parent>:
 151394 ffffffc0080911c0:       d503201f        nop

// '__wake_up_parent' is equal to 'do_group_exit'
//Needed to -4byte at arm core
```

# In kernel level

### execution
```
./set_ftrace.sh
./project/test procTest
//infinite excution
kill -9 PID
./get_ftrace.sh do_exit2
```

### test result
```
 7977         procTest-2340    [001] ..... 19503.269362: do_exit+0x4/0x9a4 <-do_group_exit+0x3c/0xa0
 7978         procTest-2340    [001] ..... 19503.269370: <stack trace>
 7979  => do_exit+0x8/0x9a4
 7980  => do_group_exit+0x3c/0xa0
 7981  => get_signal+0x8b8/0x930
 7982  => do_notify_resume+0x178/0x1280
 7983  => el0_svc+0x74/0x84
 7984  => el0t_64_sync_handler+0xb8/0xc0
 7985  => el0t_64_sync+0x18c/0x190
```

# After do_exit
```
 __schedule()
 context_switch()
 finish_task_switch()
```

1. The process which is will be finished, return almost resources at the do_exit() function
 State will be TASK_DEAD
2. Context Switching

3.The process executed by context switching, execute the finish_task_switch().
 if the state of previous process is TASK_DEAD, free the stack area.

## test finish_task_switch

### check
```
/sys/kernel/debug/tracing # cat available_filter_functions | grep finish_task_switch
finish_task_switch.isra.0
```