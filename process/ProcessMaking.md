# 'User Process' creation process

fork()->
system call(sys_clone) //_arm64_sys_clone in Rasberry pi->
kernel_clone()->
copy_process()

# 'Kernel Process' creation process
kthread_creat->
kthread_create_on_node(매크로 함수라 사실, 바로 이것이 호출 됨)->
__kthread_create_on_node(kthread를 깨움)->
==== 여기부터는 kthreadd 프로세스가 일함 ====
kthreadd->
create_kthread->
kernel_thread->
kernel_clone()->
copy_process()

kernel_clone()은 결국 공통임.
1. copy_process()를 호출해 프로세스를 생성
2. wake_up_new_take()를 호출해 생성한 프로세스를 깨움. 생성한 프로세스의 PID를 반환