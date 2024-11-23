Do not call scheduling functions when doing interrupt context.<br>

# 
```
//head.S is the start point of the kernel
//linux/arch/arm64/kernel # vi head.S
420 SYM_FUNC_START_LOCAL(__primary_switched)
421         adr_l   x4, init_task
422         init_cpu_task x4, x5, x6
423 
424         adr_l   x8, vectors                     // load VBAR_EL1 with virtual
425         msr     vbar_el1, x8                    // vector table address

//vector table
//linux/arch/arm64/kernel # vi entry.S
 506 SYM_CODE_START(vectors)
 507         kernel_ventry   1, t, 64, sync          // Synchronous EL1t
 508         kernel_ventry   1, t, 64, irq           // IRQ EL1t
 509         kernel_ventry   1, t, 64, fiq           // FIQ EL1t
 510         kernel_ventry   1, t, 64, error         // Error EL1t
 511 
 512         kernel_ventry   1, h, 64, sync          // Synchronous EL1h
 513         kernel_ventry   1, h, 64, irq           // IRQ EL1h
 514         kernel_ventry   1, h, 64, fiq           // FIQ EL1h
 515         kernel_ventry   1, h, 64, error         // Error EL1h
 516 
 517         kernel_ventry   0, t, 64, sync          // Synchronous 64-bit EL0
 518         kernel_ventry   0, t, 64, irq           // IRQ 64-bit EL0
 519         kernel_ventry   0, t, 64, fiq           // FIQ 64-bit EL0
 520         kernel_ventry   0, t, 64, error         // Error 64-bit EL0
 521 
 522         kernel_ventry   0, t, 32, sync          // Synchronous 32-bit EL0
 523         kernel_ventry   0, t, 32, irq           // IRQ 32-bit EL0
 524         kernel_ventry   0, t, 32, fiq           // FIQ 32-bit EL0
 525         kernel_ventry   0, t, 32, error         // Error 32-bit EL0
 526 SYM_CODE_END(vectors)

//definition of kernel_ventry which is macro function.
  38         .macro kernel_ventry, el:req, ht:req, regsize:req, label:req //here is important
  39         .align 7
  40 .Lventry_start\@:
  41         .if     \el == 0
  42         /*
  43          * This must be the first instruction of the EL0 vector entries. It is
  44          * skipped by the trampoline vectors, to trigger the cleanup.
  45          */
  46         b       .Lskip_tramp_vectors_cleanup\@
  47         .if     \regsize == 64
  48         mrs     x30, tpidrro_el0
  49         msr     tpidrro_el0, xzr
  50         .else
  51         mov     x30, xzr
  52         .endif
  53 .Lskip_tramp_vectors_cleanup\@:
  54         .endif
  55 
  56         sub     sp, sp, #PT_REGS_SIZE
  57 #ifdef CONFIG_VMAP_STACK
  58         /*
  59          * Test whether the SP has overflowed, without corrupting a GPR.
  60          * Task and IRQ stacks are aligned so that SP & (1 << THREAD_SHIFT)
  61          * should always be zero.
  62          */
  63         add     sp, sp, x0                      // sp' = sp + x0
  65         tbnz    x0, #THREAD_SHIFT, 0f
  68         b       el\el\ht\()_\regsize\()_\label
  69 
  70 0:
  71         /*
  72          * Either we've just detected an overflow, or we've taken an exception
  73          * while on the overflow stack. Either way, we won't return to
  74          * userspace, and can clobber EL0 registers to free up GPRs.
  75          */
  76 
  77         /* Stash the original SP (minus PT_REGS_SIZE) in tpidr_el0. */
  78         msr     tpidr_el0, x0
  79 
  80         /* Recover the original x0 value and stash it in tpidrro_el0 */
  81         sub     x0, sp, x0
  82         msr     tpidrro_el0, x0
  83 
  84         /* Switch to the overflow stack */
  85         adr_this_cpu sp, overflow_stack + OVERFLOW_STACK_SIZE, x0
  86 
  87         /*
  88          * Check whether we were already on the overflow stack. This may happen
  89          * after panic() re-enables interrupts.
  90          */
  91         mrs     x0, tpidr_el0                   // sp of interrupted context
  92         sub     x0, sp, x0                      // delta with top of overflow stack
  93         tst     x0, #~(OVERFLOW_STACK_SIZE - 1) // within range?
  94         b.ne    __bad_stack                     // no? -> bad stack pointer
  95 
  96         /* We were already on the overflow stack. Restore sp/x0 and carry on. */
  97         sub     sp, sp, x0
  98         mrs     x0, tpidrro_el0
  99 #endif
 100         b       el\el\ht\()_\regsize\()_\label
 101 .org .Lventry_start\@ + 128     // Did we overflow the ventry slot?
 102         .endm

//above codes head toward below line.
 100         b       el\el\ht\()_\regsize\()_\label

// label defined
 560 SYM_CODE_START_LOCAL(el\el\ht\()_\regsize\()_\label)
 561         kernel_entry \el, \regsize
 562         mov     x0, sp
 563         bl      el\el\ht\()_\regsize\()_\label\()_handler //here is important
 564         .if \el == 0
 565         b       ret_to_user
 566         .else
 567         b       ret_to_kernel
 568         .endif
 569 SYM_CODE_END(el\el\ht\()_\regsize\()_\label)
 570         .endm

 572 /*
 573  * Early exception handlers
 574  */
 575         entry_handler   1, t, 64, sync
 576         entry_handler   1, t, 64, irq
 577         entry_handler   1, t, 64, fiq
 578         entry_handler   1, t, 64, error
 579 
 580         entry_handler   1, h, 64, sync
 581         entry_handler   1, h, 64, irq
 582         entry_handler   1, h, 64, fiq
 583         entry_handler   1, h, 64, error
 584 
 585         entry_handler   0, t, 64, sync
 586         entry_handler   0, t, 64, irq
 587         entry_handler   0, t, 64, fiq
 588         entry_handler   0, t, 64, error
 589 
 590         entry_handler   0, t, 32, sync
 591         entry_handler   0, t, 32, irq
 592         entry_handler   0, t, 32, fiq
 593         entry_handler   0, t, 32, error

 560 SYM_CODE_START_LOCAL(el\el\ht\()_\regsize\()_\label)
 561         kernel_entry \el, \regsize
 562         mov     x0, sp
 563         bl      el\el\ht\()_\regsize\()_\label\()_handler
 564         .if \el == 0
 565         b       ret_to_user
 566         .else
 567         b       ret_to_kernel
 568         .endif
 569 SYM_CODE_END(el\el\ht\()_\regsize\()_\label)
 570         .endm

 //linux/arch/arm64/kernel # vi entry-common.c
420 asmlinkage void noinstr el1h_64_sync_handler(struct pt_regs *regs)
421 {
422         unsigned long esr = read_sysreg(esr_el1);
423 
424         switch (ESR_ELx_EC(esr)) {
425         case ESR_ELx_EC_DABT_CUR:
426         case ESR_ELx_EC_IABT_CUR:
427                 el1_abort(regs, esr);
428                 break;

1. Indicate a base in the head.S file. In this case, the base is vbar_el1 
2. Follow vectors that SYM_CODE_START indicate.
3. SYM_CODE_START_LOCAL define entry_handler
4. Enter b el1h_64_sync_lable
5. b el1h_64_sync_handler
```