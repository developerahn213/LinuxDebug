# 
```
//linux/drivers/mailbox # vi bcm2835-mailbox.c

/sys/kernel/tracing # cat available_filter_functions | grep bcm2835_mbox_irq
bcm2835_mbox_irq

//executed from an idle
 491           <idle>-0       [000] d.h1.   327.007803: irq_handler_entry: irq=14 name=fe00b880.mailbox
  492           <idle>-0       [000] d.h1.   327.007806: bcm2835_mbox_irq+0x4/0xc0 <-__handle_irq_event_percpu+0x60/0x250
  493           <idle>-0       [000] d.h1.   327.007812: <stack trace>
  494  => bcm2835_mbox_irq+0x8/0xc0
  495  => __handle_irq_event_percpu+0x60/0x250
  496  => handle_irq_event+0x54/0xd0
  497  => handle_fasteoi_irq+0xac/0x1f0
  498  => generic_handle_domain_irq+0x34/0x50
  499  => gic_handle_irq+0x4c/0xe0
  500  => call_on_irq_stack+0x24/0x50
  501  => do_interrupt_handler+0x88/0x90
  502  => el1_interrupt+0x34/0x70
  503  => el1h_64_irq_handler+0x18/0x2c
  504  => el1h_64_irq+0x64/0x68
  505  => arch_cpu_idle+0x18/0x2c
  506  => default_idle_call+0x54/0x190
  507  => do_idle+0x23c/0x270
  508  => cpu_startup_entry+0x40/0x4c
  509  => rest_init+0xf8/0x100
  510  => arch_post_acpi_subsys_init+0x0/0x28
  511  => start_kernel+0x6f4/0x734
  512  => __primary_switched+0xbc/0xc4

//executed from a kworker
 7038     kworker/u8:0-9       [000] d.h..   337.398802: bcm2835_mbox_irq+0x4/0xc0 <-__handle_irq_event_percpu+0x60/0x250
 7039     kworker/u8:0-9       [000] d.h..   337.398809: <stack trace>
   // ↓interrupt context
 7040  => bcm2835_mbox_irq+0x8/0xc0
 7041  => __handle_irq_event_percpu+0x60/0x250
 7042  => handle_irq_event+0x54/0xd0
 7043  => handle_fasteoi_irq+0xac/0x1f0
 7044  => generic_handle_domain_irq+0x34/0x50
 7045  => gic_handle_irq+0x4c/0xe0
 7046  => call_on_irq_stack+0x24/0x50
 7047  => do_interrupt_handler+0x88/0x90
 7048  => el1_interrupt+0x34/0x70
 7049  => el1h_64_irq_handler+0x18/0x2c
 7050  => el1h_64_irq+0x64/0x68 //interrupt handler executed
  // ↑interrupt context
 7051  => wq_worker_running+0x0/0x94
 7052  => worker_thread+0xf4/0x444
 7053  => kthread+0x114/0x120
 7054  => ret_from_fork+0x10/0x20
```

### el1h_64_irq
exception level 1(kernel level)<br>
irq(interrupt request)

