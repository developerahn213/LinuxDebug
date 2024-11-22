# outline

### handler
```
//amimouse_interrupt is the handler
 80         error = request_irq(IRQ_AMIGA_VERTB, amimouse_interrupt, 0, "amimouse",
 81                             dev);

 34 static irqreturn_t amimouse_interrupt(int irq, void *data)
 35 {
 36         struct input_dev *dev = data;
 37         unsigned short joy0dat, potgor;
 38         int nx, ny, dx, dy;
 39 
 40         joy0dat = amiga_custom.joy0dat;
```

### interrupt descriptor
```
 55 struct irq_desc {
 56         struct irq_common_data  irq_common_data;
 57         struct irq_data         irq_data;
 58         unsigned int __percpu   *kstat_irqs;
 59         irq_flow_handler_t      handle_irq;
 60         struct irqaction        *action;        /* IRQ action list */
 61         unsigned int            status_use_accessors;
 62         unsigned int            core_internal_state__do_not_mess_with_it;
 63         unsigned int            depth;          /* nested irq disables */
 64         unsigned int            wake_depth;     /* nested wake enables */
 65         unsigned int            tot_count;
 66         unsigned int            irq_count;      /* For detecting broken IRQs */
 67         unsigned long           last_unhandled; /* Aging timer for unhandled count *
 ```