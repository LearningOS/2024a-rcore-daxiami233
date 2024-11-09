## 编程作业

编写系统调用 spawn，直接创建子进程。通过读取ELF文件，新建地址空间，修改子进程列表。通过为每个进程设置一个当前 stride，表示该进程当前已经运行的“长度”。另外设置其对应的 pass 值（只与进程的优先权有关系），表示对应进程在调度后，stride 需要进行的累加值。每次需要调度时，从当前 runnable 态的进程中选择 stride 最小的进程调度。对于获得调度的进程 P，将对应的 stride 加上其对应的步长 pass。使用对进程等待列表暴力扫一遍的办法找最小值。



## 简答作业

1. stride 算法原理非常简单，但是有一个比较大的问题。例如两个 pass = 10 的进程，使用 8bit 无符号整形储存 stride， p1.stride = 255, p2.stride = 250，在 p2 执行一个时间片后，理论上下一次应该 p1 执行。

   - 实际情况是轮到 p1 执行吗？为什么？
     - `p1.stride = 255`和`p2.stride = 250`。在`p2`执行一个时间片后，理论上下一次应该`p1`执行。但实际上，由于`p2`执行后其`stride`值会加上pass ，变为260，但使用 8bit 无符号整形储存`stride` ，`stride`放生了溢出，变成5，比p1更小，所以依旧是p2执行。

   我们之前要求进程优先级 >= 2 其实就是为了解决这个问题。可以证明， **在不考虑溢出的情况下** , 在进程优先级全部 >= 2 的情况下，如果严格按照算法执行，那么 STRIDE_MAX – STRIDE_MIN <= BigStride / 2。

   - 为什么？尝试简单说明（不要求严格证明）。
     - 为了保证调度的公平性，可以要求所有进程的优先级（在这里可以理解为`stride`值的倒数）都大于或等于2。这样，即使在`stride`值减少的情况下，也能保证`STRIDE_MAX – STRIDE_MIN <= BigStride / 2`。这个结论的直观理解是，如果所有进程的`stride`值都相对较大，那么它们减少的速率会相对较慢，从而减少了因为`stride`值快速减少而导致的调度不公平性。
   - 已知以上结论，**考虑溢出的情况下**，可以为 Stride 设计特别的比较器，让 BinaryHeap<Stride> 的 pop 方法能返回真正最小的 Stride。补全下列代码中的 `partial_cmp` 函数，假设两个 Stride 永远不会相等。

   ```rust
   use core::cmp::Ordering;
   
   struct Stride(u8);
   
   impl PartialOrd for Stide {
       fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
           let diff = (i8::from(self.0) - i8::from(other.0)).abs();
           if diff == 0 {
               Some(Ordering::Equal)
           } else if diff <= (i8::MAX - i8::MIN) / 2 {
               Some(i8::from(self.0).cmp(&i8::from(other.0)))
           } else {
               Some(i8::from(other.0).cmp(&i8::from(self.0)))
           }
       }
   }
   
   impl PartialEq for Stride {
       fn eq(&self, other: &Self) -> bool {
           false
       }
   }
   ```

   TIPS: 使用 8 bits 存储 stride, BigStride = 255, 则: `(125 < 255) == false`, `(129 < 255) == true`.

   

## 荣誉准则

1. 在完成本次实验的过程（含此前学习的过程）中，我曾分别与 以下各位 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：

2. 此外，我也参考了 以下资料 ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：

《rCore-Camp-Guide-2024A 文档》

3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。

4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。