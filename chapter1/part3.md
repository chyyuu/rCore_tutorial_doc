## 移除标准库依赖

* [代码][CODE]

项目默认是链接 rust 标准库 std 的，它依赖于操作系统，因此我们需要显式将其禁用：

```rust
// src/main.rs
// 之后出现的所有代码块内的路径都放在 os 文件夹下

#![no_std]
fn main() {
    println!("Hello, world!");
}
```

我们使用  ``cargo build``  构建项目，会出现下面的错误：

> **[danger] cargo build error**
>
> ```rust
> error: cannot find macro `println` in this scope
>  --> src/main.rs:3:5
>   |
> 3 |     println!("Hello, world!");
>   |     ^^^^^^^
> error: `#[panic_handler]` function required, but not found
> error: language item required, but not found: `eh_personality
> ```

接下来，我们依次解决这些问题。

### println!

第一个错误是说  ``println!``  宏未找到，实际上这个宏属于 Rust 标准库 std，由于它被我们禁用了当然就找不到了。我们暂时将其删除，之后自己给出不依赖操作系统的实现。
> **[info] ``println!``哪里依赖了操作系统**
> 
> 这个宏会输出到**标准输出**，而这需要操作系统的支持。
> 

### panic

第二个错误是说需要一个函数作为 ``panic_handler`` ，这个函数负责在程序 ``panic`` 时调用。它默认使用标准库 std 中实现的函数，由于我们禁用了标准库，因此只能自己实现它：

```rust
// src/main.rs

use core::panic::PanicInfo;
// This function is called on panic.
#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    loop {}
}
```

> **[info] panic**
>
> panic 在 Rust 中表明程序遇到了不可恢复的错误，只能被迫停止运行。
> 

程序在 panic 后就应该结束，不过我们暂时先让这个 handler 卡在一个死循环里。因此这个 handler 不会结束，我们用``!``类型的返回值表明这个函数不会返回。

这里我们用到了核心库 ``core`` ，与标准库 ``std`` 不同，这个库不需要操作系统的支持，下面我们还会与它打交道。

### eh_personality

第三个错误提到了语义项 (language item) ，它是编译器内部所需的特殊函数或类型。刚才的 ``panic_handler`` 也是一个语义项，我们要用它告诉编译器当程序 panic 之后如何处理。

而这个错误相关语义项 ``eh_personality`` ，其中 ``eh`` 是 ``exception handling`` 的缩写，它是一个标记某函数用来实现**堆栈展开**处理功能的语义项。这个语义项也与 ``panic`` 有关。

> **[info] 堆栈展开 (stack unwinding) **
> 
> 通常，当程序出现了异常 (这里指类似 Java 中层层抛出的异常)，从异常点开始会沿着 caller 调用栈一层一层回溯，直到找到某个函数能够捕获 (catch) 这个异常。这个过程称为 堆栈展开。
>
> 当程序出现不可恢复错误时，我们需要沿着调用栈一层层回溯上去回收每个caller 中定义的局部变量**避免造成内存溢出**。这里的回收包括 C++ 的 RAII 的析构以及 Rust 的 drop。
>
> 而在 Rust 中，panic 证明程序出现了不可恢复错误，我们则会对于每个 caller 函数调用依次这个被标记为堆栈展开处理函数的函数。
>
> 这个处理函数是一个依赖于操作系统的复杂过程，在标准库中实现，我们禁用了标准库使得编译器找不到该过程的实现函数了。

简单起见，我们不用考虑内存溢出，设置当程序 panic 时不做任何清理工作，直接退出程序即可。这样堆栈展开处理函数不会被调用，编译器也就不会去寻找它的实现了。

因此，我们在项目配置文件中直接将 dev (use for `cargo build`) 和 release (use for `cargo build --release`) 的 panic 的处理策略设为 abort。

```rust
// Cargo.toml

[profile.dev]
panic = "abort"

[profile.release]
panic = "abort"
```

此时，我们 ``cargo build`` ，但是又出现了新的错误...

> **[danger] cargo build error**
> 
> ``error: requires `start` lang_item``
> 

[CODE]: https://github.com/rcore-os/rCore_tutorial/tree/ch1-pa4
