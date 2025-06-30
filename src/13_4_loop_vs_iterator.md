### loop vs iterator

* **Performance:** In Rust, iterators and explicit `for` loops have nearly identical performance. Benchmarks show that the difference in speed is negligible.
* **Zero-Cost Abstraction:** Iterators in Rust are a "zero-cost abstraction." This means that using this high-level feature doesn't add any runtime overhead. The compiler optimizes the code to be as efficient as if you had written the lower-level equivalent yourself.
* **Compiler Optimizations:** Rust's compiler can perform advanced optimizations on iterator-based code, such as loop unrolling. This can lead to highly efficient assembly code that avoids the typical overhead of loops and bounds-checking.
* **Real-World Application:** The audio decoder example demonstrates how a chain of iterator methods (`.iter()`, `.zip()`, `.map()`, `.sum()`) can be compiled into highly optimized, low-level code without any actual looping instructions in the final assembly.

In short, you can and should use iterators and closures in Rust without being concerned about performance penalties. They offer more concise and expressive code without sacrificing speed.