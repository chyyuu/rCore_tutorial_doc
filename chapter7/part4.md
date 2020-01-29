## 线程调度测试

* [代码][CODE]

我们终于可以来测试一下这一章的代码实现的有没有问题了！

```rust
// src/process/mod.rs

use scheduler::RRScheduler;
use thread_pool::ThreadPool;
use alloc::boxed::Box;

pub fn init() {
    // 使用 Round Robin Scheduler
    let scheduler = RRScheduler::new(1);
    // 新建线程池
    let thread_pool = ThreadPool::new(100, Box::new(scheduler));
	// 新建内核线程 idle ，其入口为 Processor::idle_main
    let idle = Thread::new_kernel(Processor::idle_main as usize);
    // 我们需要传入 CPU 的地址作为参数
    idle.append_initial_arguments([&CPU as *const Processor as usize, 0, 0]);
    // 初始化 CPU
    CPU.init(idle, Box::new(thread_pool));
    
    // 依次新建 5 个内核线程并加入调度单元
    for i in 0..5 {
        CPU.add_thread({
            let thread = Thread::new_kernel(hello_thread as usize);
            // 传入一个编号作为参数
            thread.append_initial_arguments([i, 0, 0]);
            thread
        });
    }
    println!("++++ setup process!   ++++");
}

pub fn run() {
    CPU.run();
}

// src/process/processor.rs

impl Processor {  
	pub fn run(&self) {
        // 运行，也就是从启动线程切换到调度线程 idle
        Thread::get_boot_thread().switch_to(&mut self.inner().idle);
    }
}
```

内核线程的入口点是：

```rust
// src/process/mod.rs

#[no_mangle]
pub extern "C" fn hello_thread(arg: usize) -> ! {
    println!("begin of thread {}", arg);
    for i in 0..800 {
        print!("{}", arg);
    }
    println!("\nend  of thread {}", arg);
    // 通知 CPU 自身已经退出
    CPU.exit(0);
    loop {}
}
```

随后我们在``rust_main``主函数里添加调用``crate::process::init()``函数和``crate::process::run()``函数：

```rust
// src/init.rs

#[no_mangle]
pub extern "C" fn rust_main() -> ! {
    crate::interrupt::init();

	extern "C" {
		fn end();
	}
	crate::memory::init(
        ((end as usize - KERNEL_BEGIN_VADDR + KERNEL_BEGIN_PADDR) >> 12) + 1,
        PHYSICAL_MEMORY_END >> 12
    );
	crate::process::init();
    crate::timer::init();
	crate::process::run();
    loop {}
}

```

``make run`` 一下，终于可以看到结果了！

这里开始就已经没有确定性的运行显示结果了，一个参考结果如下：

> **[success] 线程调度成功**
> 
> ```rust
> ++++ setup interrupt! ++++
> switch satp from 0x8000000000080221 to 0x8000000000080a37
> ++++ setup memory!    ++++
> ++++ setup process!   ++++
> ++++ setup timer!     ++++
> 
> >>>> will switch_to thread 0 in idie_main!
> begin of thread 0
> 0000000000000000000000000000000000000000000000000000000000000000000000000
> 0000000000000000000000000000000000000000000000000000000000000000000000000
> 0000000000000000000000000000000000000000000000000000000000000000000000000
> 0000000000000000000000000000000000000000000000000000000000000000000000000
> 0000000000000000000000000000000000000000000000000000000000000000000000000
> 000000000000
> <<<< switch_back to idle in idle_main!
> 
> >>>> will switch_to thread 1 in idie_main!
> begin of thread 1
> 1111111111111111111111111111111111111111111111111111111111111111111111111
> 1111111111111111111111111111111111111111111111111111111111111111111111111
> 1111111111111111111111111111111111111111111111111111111111111111111111111
> 1111111111111111111111111111111111111111111111111111111111111111111111111
> 1111111111111111111111111111111111111111111111111111111111111111111111111
> 1111111111111111111111111111111111111111111111111111111111111111111111111
> 11111111111111111111111
> <<<< switch_back to idle in idle_main!
> ......
> ```

我们可以清楚的看到在每一个时间片内每个线程所做的事情。

如果结果不对的话，[这里][CODE]可以看到至今的所有代码。

[CODE]: https://github.com/rcore-os/rCore_tutorial/tree/ch7-pa4
