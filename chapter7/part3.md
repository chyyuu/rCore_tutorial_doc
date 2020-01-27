## 线程调度之 Round Robin 算法

* [代码][CODE]

这里我们直接给出代码，有兴趣者可自行去研究算法细节。

```rust
// src/process/scheduler.rs

use alloc::vec::Vec;

#[derive(Default)]
struct RRInfo {
    valid: bool,
    time: usize,
    prev: usize,
    next: usize,
}

pub struct RRScheduler {
    threads: Vec<RRInfo>,
    max_time: usize,
    current: usize,
}

impl RRScheduler {
    // 设置每个线程连续运行的最大 tick 数
    pub fn new(max_time_slice: usize) -> Self {
        let mut rr = RRScheduler {
            threads: Vec::default(),
            max_time: max_time_slice,
            current: 0,
        };
        rr.threads.push(
            RRInfo {
                valid: false,
                time: 0,
                prev: 0,
                next: 0,
            }
        );
        rr
    }
}
impl Scheduler for RRScheduler {
    // 分为 1. 新线程 2. 时间片耗尽被切换出的线程 两种情况
    fn push(&mut self, tid : Tid) {
        let tid = tid + 1;
        if tid + 1 > self.threads.len() {
            self.threads.resize_with(tid + 1, Default::default);
        }

        if self.threads[tid].time == 0 {
            self.threads[tid].time = self.max_time;
        }

        let prev = self.threads[0].prev;
        self.threads[tid].valid = true;
        self.threads[prev].next = tid;
        self.threads[tid].prev = prev;
        self.threads[0].prev = tid;
        self.threads[tid].next = 0;
    }

    fn pop(&mut self) -> Option<Tid> {
        let ret = self.threads[0].next;
        if ret != 0 {
            let next = self.threads[ret].next;
            let prev = self.threads[ret].prev;
            self.threads[next].prev = prev;
            self.threads[prev].next = next;
            self.threads[ret].prev = 0;
            self.threads[ret].next = 0;
            self.threads[ret].valid = false;
            self.current = ret;
            Some(ret-1)
        }else{
            None
        }
    }

    // 当前线程的可用时间片 -= 1
    fn tick(&mut self) -> bool{
        let tid = self.current;
        if tid != 0 {
            self.threads[tid].time -= 1;
            if self.threads[tid].time == 0 {
                return true;
            }else{
                return false;
            }
        }
        return true;
    }

    fn exit(&mut self, tid : Tid) {
        let tid = tid + 1;
        if self.current == tid {
            self.current = 0;
        }
    }
}
```

[CODE]: https://github.com/rcore-os/rCore_tutorial/tree/ch7-pa4
