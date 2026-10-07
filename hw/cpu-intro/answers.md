# OSTEP Ch.4 Homework Answers

The predictions were recorded before running `-c`, then checked against the saved traces. The original predictions are unchanged. A `*` marks an I/O completion; `-` means idle/no I/O.

## Q1
- Prediction / 预测: total time **10 ticks**; CPU busy **10/10 = 100.00%**. State table:

```text
Time          PID 0         PID 1         CPU           IOs
1             RUN:cpu       READY         1             -
2             RUN:cpu       READY         1             -
3             RUN:cpu       READY         1             -
4             RUN:cpu       READY         1             -
5             RUN:cpu       READY         1             -
6             DONE          RUN:cpu       1             -
7             DONE          RUN:cpu       1             -
8             DONE          RUN:cpu       1             -
9             DONE          RUN:cpu       1             -
10            DONE          RUN:cpu       1             -
```

- Reasoning / 理由: PID 0 uses five CPU ticks and finishes before PID 1 starts its five CPU ticks. There is no I/O wait, so the CPU should never be idle. / 两个进程各连续执行5个CPU指令，没有I/O阻塞。
- Verified result / 验证结果: 10 ticks; CPU busy 10/10 (100.00%), as shown in `q1.txt`.
- Analysis / 分析: PID 0 finishes at tick 5 and PID 1 starts at tick 6. The switch adds no idle tick, which is why the CPU stays busy the whole time. / 两个进程直接接续执行，切换没有额外空闲时间。

## Q2
- Prediction / 预测: total time **11 ticks**; CPU busy **6/11 = 54.55%**. State table:

```text
Time          PID 0         PID 1         CPU           IOs
1             RUN:cpu       READY         1             -
2             RUN:cpu       READY         1             -
3             RUN:cpu       READY         1             -
4             RUN:cpu       READY         1             -
5             DONE          RUN:io        1             -
6             DONE          BLOCKED       -             1
7             DONE          BLOCKED       -             1
8             DONE          BLOCKED       -             1
9             DONE          BLOCKED       -             1
10            DONE          BLOCKED       -             1
11*           DONE          RUN:io_done   1             -
```

- Reasoning / 理由: PID 1 cannot issue its I/O until PID 0 completes at tick 4. Starting the I/O costs tick 5, the five blocked ticks are 6–10, and completion handling costs tick 11. / PID 1须等PID 0完成才发起I/O；发起和处理完成各占1 tick，中间等待5 tick。
- Verified result / 验证结果: 11 ticks; CPU busy 6/11 (54.55%), from `q2.txt`.
- Analysis / 分析: The five idle ticks are 6–10, after PID 1 starts its I/O. Counting the start and completion as CPU work explains why CPU busy is 6 rather than 4. / 发起与完成I/O也占用CPU，真正空闲的是第6至10 tick。

## Q3
- Prediction / 预测: total time **7 ticks**; CPU busy **6/7 = 85.71%**. State table:

```text
Time          PID 0         PID 1         CPU           IOs
1             RUN:io        READY         1             -
2             BLOCKED       RUN:cpu       1             1
3             BLOCKED       RUN:cpu       1             1
4             BLOCKED       RUN:cpu       1             1
5             BLOCKED       RUN:cpu       1             1
6             BLOCKED       DONE          -             1
7*            RUN:io_done   DONE          1             -
```

- Reasoning / 理由: PID 0 starts I/O first, and the default switch-on-I/O policy lets PID 1 execute four CPU instructions during the wait. Tick 6 remains idle before PID 0 handles completion at tick 7. / I/O等待与PID 1的CPU工作重叠，只有第6 tick空闲。
- Verified result / 验证结果: 7 ticks; CPU busy 6/7 (85.71%), from `q3.txt`.
- Analysis / 分析: Reversing the process order from Q2 makes a large difference: PID 1 can use ticks 2–5 while PID 0 waits for I/O. Only tick 6 is left idle. / 与Q2相比，先发起I/O让另一个进程利用了等待时间。

## Q4
- Prediction / 预测: total time **11 ticks**; CPU busy **6/11 = 54.55%**. State table:

```text
Time          PID 0         PID 1         CPU           IOs
1             RUN:io        READY         1             -
2             BLOCKED       READY         -             1
3             BLOCKED       READY         -             1
4             BLOCKED       READY         -             1
5             BLOCKED       READY         -             1
6             BLOCKED       READY         -             1
7*            RUN:io_done   READY         1             -
8             DONE          RUN:cpu       1             -
9             DONE          RUN:cpu       1             -
10            DONE          RUN:cpu       1             -
11            DONE          RUN:cpu       1             -
```

- Reasoning / 理由: With SWITCH_ON_END, issuing I/O does not switch to ready PID 1. The CPU sits idle for five ticks, then PID 0 handles completion before PID 1 runs. / 等待I/O时不切换，所以PID 1虽就绪，CPU仍空闲。
- Verified result / 验证结果: 11 ticks; CPU busy 6/11 (54.55%), from `q4.txt`.
- Analysis / 分析: PID 1 is READY throughout PID 0's wait, yet the CPU is idle on ticks 2–6. This shows that READY does not mean the process will run if the switching policy forbids it. / PID 1虽已就绪，但切换策略让CPU在等待期间保持空闲。

## Q5
- Prediction / 预测: total time **7 ticks**; CPU busy **6/7 = 85.71%**. State table:

```text
Time          PID 0         PID 1         CPU           IOs
1             RUN:io        READY         1             -
2             BLOCKED       RUN:cpu       1             1
3             BLOCKED       RUN:cpu       1             1
4             BLOCKED       RUN:cpu       1             1
5             BLOCKED       RUN:cpu       1             1
6             BLOCKED       DONE          -             1
7*            RUN:io_done   DONE          1             -
```

- Reasoning / 理由: SWITCH_ON_IO allows PID 1 to work while PID 0 is blocked. Compared with Q4, four CPU ticks overlap the I/O wait, reducing elapsed time from 11 to 7. / 与Q4相比，切换让CPU工作与I/O等待重叠。
- Verified result / 验证结果: 7 ticks; CPU busy 6/7 (85.71%), from `q5.txt`.
- Analysis / 分析: The workload is the same as Q4. Allowing a switch when I/O starts moves PID 1's four CPU ticks into PID 0's waiting period, cutting the total time by four ticks. / 工作量没有变，只是利用了原本空闲的四个tick。

## Q6
- Prediction / 预测: total time **31 ticks**; CPU busy **21/31 = 67.74%**. State table:

```text
Time          PID 0         PID 1         PID 2         PID 3         CPU           IOs
1             RUN:io        READY         READY         READY         1             -
2             BLOCKED       RUN:cpu       READY         READY         1             1
3             BLOCKED       RUN:cpu       READY         READY         1             1
4             BLOCKED       RUN:cpu       READY         READY         1             1
5             BLOCKED       RUN:cpu       READY         READY         1             1
6             BLOCKED       RUN:cpu       READY         READY         1             1
7*            READY         DONE          RUN:cpu       READY         1             -
8             READY         DONE          RUN:cpu       READY         1             -
9             READY         DONE          RUN:cpu       READY         1             -
10            READY         DONE          RUN:cpu       READY         1             -
11            READY         DONE          RUN:cpu       READY         1             -
12            READY         DONE          DONE          RUN:cpu       1             -
13            READY         DONE          DONE          RUN:cpu       1             -
14            READY         DONE          DONE          RUN:cpu       1             -
15            READY         DONE          DONE          RUN:cpu       1             -
16            READY         DONE          DONE          RUN:cpu       1             -
17            RUN:io_done   DONE          DONE          DONE          1             -
18            RUN:io        DONE          DONE          DONE          1             -
19            BLOCKED       DONE          DONE          DONE          -             1
20            BLOCKED       DONE          DONE          DONE          -             1
21            BLOCKED       DONE          DONE          DONE          -             1
22            BLOCKED       DONE          DONE          DONE          -             1
23            BLOCKED       DONE          DONE          DONE          -             1
24*           RUN:io_done   DONE          DONE          DONE          1             -
25            RUN:io        DONE          DONE          DONE          1             -
26            BLOCKED       DONE          DONE          DONE          -             1
27            BLOCKED       DONE          DONE          DONE          -             1
28            BLOCKED       DONE          DONE          DONE          -             1
29            BLOCKED       DONE          DONE          DONE          -             1
30            BLOCKED       DONE          DONE          DONE          -             1
31*           RUN:io_done   DONE          DONE          DONE          1             -
```

- Reasoning / 理由: PID 0 issues I/O at tick 1, but IO_RUN_LATER leaves it READY when that I/O finishes at tick 7; PIDs 2 and 3 complete before PID 0 runs again at tick 17. Its later I/Os find no CPU work to overlap. / 首次I/O结束后PID 0排队到其余CPU进程之后；后两次I/O期间CPU空闲。
- Verified result / 验证结果: 31 ticks; CPU busy 21/31 (67.74%), from `q6.txt`.
- Analysis / 分析: PID 0's first I/O is done at tick 7, but it does not run again until tick 17. By then the CPU-only processes have finished, so its next two I/O waits account for all ten idle ticks. / PID 0恢复得太晚，后两次I/O已没有其他进程可重叠执行。

## Q7
- Prediction / 预测: total time **21 ticks**; CPU busy **21/21 = 100.00%**. State table:

```text
Time          PID 0         PID 1         PID 2         PID 3         CPU           IOs
1             RUN:io        READY         READY         READY         1             -
2             BLOCKED       RUN:cpu       READY         READY         1             1
3             BLOCKED       RUN:cpu       READY         READY         1             1
4             BLOCKED       RUN:cpu       READY         READY         1             1
5             BLOCKED       RUN:cpu       READY         READY         1             1
6             BLOCKED       RUN:cpu       READY         READY         1             1
7*            RUN:io_done   DONE          READY         READY         1             -
8             RUN:io        DONE          READY         READY         1             -
9             BLOCKED       DONE          RUN:cpu       READY         1             1
10            BLOCKED       DONE          RUN:cpu       READY         1             1
11            BLOCKED       DONE          RUN:cpu       READY         1             1
12            BLOCKED       DONE          RUN:cpu       READY         1             1
13            BLOCKED       DONE          RUN:cpu       READY         1             1
14*           RUN:io_done   DONE          DONE          READY         1             -
15            RUN:io        DONE          DONE          READY         1             -
16            BLOCKED       DONE          DONE          RUN:cpu       1             1
17            BLOCKED       DONE          DONE          RUN:cpu       1             1
18            BLOCKED       DONE          DONE          RUN:cpu       1             1
19            BLOCKED       DONE          DONE          RUN:cpu       1             1
20            BLOCKED       DONE          DONE          RUN:cpu       1             1
21*           RUN:io_done   DONE          DONE          DONE          1             -
```

- Reasoning / 理由: IO_RUN_IMMEDIATE resumes PID 0 at ticks 7 and 14 so it can issue the next I/O before another five-CPU-tick process runs. All three I/O waits are covered by PIDs 1–3, keeping the CPU busy throughout. / 立即恢复I/O进程，使三段等待分别与其余进程的CPU工作重叠。
- Verified result / 验证结果: 21 ticks; CPU busy 21/21 (100.00%), from `q7.txt`.
- Analysis / 分析: Here PID 0 resumes at ticks 7 and 14 and starts another I/O at 8 and 15. That timing lets PIDs 2 and 3 use the next waiting periods. Q6 and Q7 do the same amount of CPU work, but only Q7 has no idle tick. / 两题CPU工作量相同；立即恢复PID 0使后两段等待也被利用。

## Q8

The instruction lists from the non-`-c` run were: seed 1: P0 `cpu, io, io`, P1 `cpu, cpu, cpu`; seed 2: P0 `io, io, cpu`, P1 `cpu, io, io`; seed 3: P0 `cpu, io, cpu`, P1 `io, io, cpu`. Each `io` also requires a later `io_done` CPU tick.

### Seed 1 — default
- Prediction / 预测: total **15**, CPU busy **8/15 = 53.33%**.

```text
Time          PID 0         PID 1         CPU           IOs
1             RUN:cpu       READY         1             -
2             RUN:io        READY         1             -
3             BLOCKED       RUN:cpu       1             1
4             BLOCKED       RUN:cpu       1             1
5             BLOCKED       RUN:cpu       1             1
6             BLOCKED       DONE          -             1
7             BLOCKED       DONE          -             1
8*            RUN:io_done   DONE          1             -
9             RUN:io        DONE          1             -
10            BLOCKED       DONE          -             1
11            BLOCKED       DONE          -             1
12            BLOCKED       DONE          -             1
13            BLOCKED       DONE          -             1
14            BLOCKED       DONE          -             1
15*           RUN:io_done   DONE          1             -
```

- Reasoning / 理由: PID 1 finishes before PID 0’s first I/O completes, so two later idle stretches remain. / PID 1先完成，后续仍有两段CPU空闲。
- Verified result / 验证结果: 15 ticks; CPU busy 8/15 (53.33%), from `q8-s1.txt` (default).
- Analysis / 分析: PID 1 finishes at tick 5, while PID 0's first I/O ends at tick 8. There is no CPU-only work left to cover the remaining wait or the second I/O. / PID 1结束得早，后面的I/O等待无法再被利用。

### Seed 1 — -I IO_RUN_IMMEDIATE
- Prediction / 预测: total **15**, CPU busy **8/15 = 53.33%**.

```text
Time          PID 0         PID 1         CPU           IOs
1             RUN:cpu       READY         1             -
2             RUN:io        READY         1             -
3             BLOCKED       RUN:cpu       1             1
4             BLOCKED       RUN:cpu       1             1
5             BLOCKED       RUN:cpu       1             1
6             BLOCKED       DONE          -             1
7             BLOCKED       DONE          -             1
8*            RUN:io_done   DONE          1             -
9             RUN:io        DONE          1             -
10            BLOCKED       DONE          -             1
11            BLOCKED       DONE          -             1
12            BLOCKED       DONE          -             1
13            BLOCKED       DONE          -             1
14            BLOCKED       DONE          -             1
15*           RUN:io_done   DONE          1             -
```

- Reasoning / 理由: At both I/O completions there is no other runnable process to preempt, so immediate and default behavior coincide. / 两次I/O结束时均无其他可运行进程，因此轨迹相同。
- Verified result / 验证结果: 15 ticks; CPU busy 8/15 (53.33%), from `q8-s1.txt` (IO_RUN_IMMEDIATE).
- Analysis / 分析: The policy changes when an I/O completion can affect scheduling. In this seed, PID 1 has already finished before either completion, so there is nobody for PID 0 to preempt. / 没有其他可运行进程，立即恢复策略不会改变轨迹。

### Seed 1 — -S SWITCH_ON_END
- Prediction / 预测: total **18**, CPU busy **8/18 = 44.44%**.

```text
Time          PID 0         PID 1         CPU           IOs
1             RUN:cpu       READY         1             -
2             RUN:io        READY         1             -
3             BLOCKED       READY         -             1
4             BLOCKED       READY         -             1
5             BLOCKED       READY         -             1
6             BLOCKED       READY         -             1
7             BLOCKED       READY         -             1
8*            RUN:io_done   READY         1             -
9             RUN:io        READY         1             -
10            BLOCKED       READY         -             1
11            BLOCKED       READY         -             1
12            BLOCKED       READY         -             1
13            BLOCKED       READY         -             1
14            BLOCKED       READY         -             1
15*           RUN:io_done   READY         1             -
16            DONE          RUN:cpu       1             -
17            DONE          RUN:cpu       1             -
18            DONE          RUN:cpu       1             -
```

- Reasoning / 理由: The OS does not run ready PID 1 during either I/O wait; it starts PID 1 only after PID 0 finishes. / 两段I/O等待均未切换，PID 1最后才运行。
- Verified result / 验证结果: 18 ticks; CPU busy 8/18 (44.44%), from `q8-s1.txt` (SWITCH_ON_END).
- Analysis / 分析: PID 1 has work ready from the start but gets the CPU only on ticks 16–18. The two I/O waits therefore become idle time instead of overlapping with PID 1's work. / PID 1一直等到PID 0结束才运行，等待期没有重叠。

### Seed 2 — default
- Prediction / 预测: total **16**, CPU busy **10/16 = 62.50%**.

```text
Time          PID 0         PID 1         CPU           IOs
1             RUN:io        READY         1             -
2             BLOCKED       RUN:cpu       1             1
3             BLOCKED       RUN:io        1             1
4             BLOCKED       BLOCKED       -             2
5             BLOCKED       BLOCKED       -             2
6             BLOCKED       BLOCKED       -             2
7*            RUN:io_done   BLOCKED       1             1
8             RUN:io        BLOCKED       1             1
9*            BLOCKED       RUN:io_done   1             1
10            BLOCKED       RUN:io        1             1
11            BLOCKED       BLOCKED       -             2
12            BLOCKED       BLOCKED       -             2
13            BLOCKED       BLOCKED       -             2
14*           RUN:io_done   BLOCKED       1             1
15            RUN:cpu       BLOCKED       1             1
16*           DONE          RUN:io_done   1             -
```

- Reasoning / 理由: The two processes have overlapping I/Os, but ticks 4–6 and 11–13 still have neither process runnable. / 两进程I/O有重叠，但两段时间都阻塞，CPU仍空闲。
- Verified result / 验证结果: 16 ticks; CPU busy 10/16 (62.50%), from `q8-s2.txt` (default).
- Analysis / 分析: The processes overlap their I/O, but overlapping I/O alone does not keep the CPU busy. Both are BLOCKED on ticks 4–6 and 11–13, leaving six idle ticks. / 两次同时阻塞造成六个CPU空闲tick。

### Seed 2 — -I IO_RUN_IMMEDIATE
- Prediction / 预测: total **16**, CPU busy **10/16 = 62.50%**.

```text
Time          PID 0         PID 1         CPU           IOs
1             RUN:io        READY         1             -
2             BLOCKED       RUN:cpu       1             1
3             BLOCKED       RUN:io        1             1
4             BLOCKED       BLOCKED       -             2
5             BLOCKED       BLOCKED       -             2
6             BLOCKED       BLOCKED       -             2
7*            RUN:io_done   BLOCKED       1             1
8             RUN:io        BLOCKED       1             1
9*            BLOCKED       RUN:io_done   1             1
10            BLOCKED       RUN:io        1             1
11            BLOCKED       BLOCKED       -             2
12            BLOCKED       BLOCKED       -             2
13            BLOCKED       BLOCKED       -             2
14*           RUN:io_done   BLOCKED       1             1
15            RUN:cpu       BLOCKED       1             1
16*           DONE          RUN:io_done   1             -
```

- Reasoning / 理由: Each I/O finishes while the other process is BLOCKED, so immediate resumption changes no tick. / 每次I/O结束时另一进程都阻塞，立即恢复不改变轨迹。
- Verified result / 验证结果: 16 ticks; CPU busy 10/16 (62.50%), from `q8-s2.txt` (IO_RUN_IMMEDIATE).
- Analysis / 分析: This trace is identical to the default one. At each I/O completion (ticks 7, 9, 14 and 16), the other process is blocked or done, so immediate resumption has no scheduling choice to change. / 每次I/O完成时都没有正在运行的竞争进程。

### Seed 2 — -S SWITCH_ON_END
- Prediction / 预测: total **30**, CPU busy **10/30 = 33.33%**.

```text
Time          PID 0         PID 1         CPU           IOs
1             RUN:io        READY         1             -
2             BLOCKED       READY         -             1
3             BLOCKED       READY         -             1
4             BLOCKED       READY         -             1
5             BLOCKED       READY         -             1
6             BLOCKED       READY         -             1
7*            RUN:io_done   READY         1             -
8             RUN:io        READY         1             -
9             BLOCKED       READY         -             1
10            BLOCKED       READY         -             1
11            BLOCKED       READY         -             1
12            BLOCKED       READY         -             1
13            BLOCKED       READY         -             1
14*           RUN:io_done   READY         1             -
15            RUN:cpu       READY         1             -
16            DONE          RUN:cpu       1             -
17            DONE          RUN:io        1             -
18            DONE          BLOCKED       -             1
19            DONE          BLOCKED       -             1
20            DONE          BLOCKED       -             1
21            DONE          BLOCKED       -             1
22            DONE          BLOCKED       -             1
23*           DONE          RUN:io_done   1             -
24            DONE          RUN:io        1             -
25            DONE          BLOCKED       -             1
26            DONE          BLOCKED       -             1
27            DONE          BLOCKED       -             1
28            DONE          BLOCKED       -             1
29            DONE          BLOCKED       -             1
30*           DONE          RUN:io_done   1             -
```

- Reasoning / 理由: With no switch on I/O, PID 0 must finish before PID 1 can begin; all four five-tick waits occur serially. / 不在I/O时切换，四次等待串行发生。
- Verified result / 验证结果: 30 ticks; CPU busy 10/30 (33.33%), from `q8-s2.txt` (SWITCH_ON_END).
- Analysis / 分析: PID 0 has to finish both I/Os before PID 1 starts. PID 1 then has two I/Os of its own, so the four five-tick waits happen one after another: 20 idle ticks in total. / 四段等待依次发生，20个tick里CPU都没有工作。

### Seed 3 — default
- Prediction / 预测: total **18**, CPU busy **9/18 = 50.00%**.

```text
Time          PID 0         PID 1         CPU           IOs
1             RUN:cpu       READY         1             -
2             RUN:io        READY         1             -
3             BLOCKED       RUN:io        1             1
4             BLOCKED       BLOCKED       -             2
5             BLOCKED       BLOCKED       -             2
6             BLOCKED       BLOCKED       -             2
7             BLOCKED       BLOCKED       -             2
8*            RUN:io_done   BLOCKED       1             1
9*            RUN:cpu       READY         1             -
10            DONE          RUN:io_done   1             -
11            DONE          RUN:io        1             -
12            DONE          BLOCKED       -             1
13            DONE          BLOCKED       -             1
14            DONE          BLOCKED       -             1
15            DONE          BLOCKED       -             1
16            DONE          BLOCKED       -             1
17*           DONE          RUN:io_done   1             -
18            DONE          RUN:cpu       1             -
```

- Reasoning / 理由: When PID 1’s I/O completes at tick 9, IO_RUN_LATER leaves PID 0 running its final CPU instruction; PID 1 resumes afterward. / 第9 tick的I/O完成不会抢占正在运行的PID 0。
- Verified result / 验证结果: 18 ticks; CPU busy 9/18 (50.00%), from `q8-s3.txt` (default).
- Analysis / 分析: PID 1's I/O finishes at tick 9, but PID 0 uses that tick for its final CPU instruction. PID 1 resumes at tick 10 and cannot start its second I/O until tick 11. / 默认策略让PID 0先完成，PID 1的下一次I/O到第11 tick才开始。

### Seed 3 — -I IO_RUN_IMMEDIATE
- Prediction / 预测: total **17**, CPU busy **9/17 = 52.94%**.

```text
Time          PID 0         PID 1         CPU           IOs
1             RUN:cpu       READY         1             -
2             RUN:io        READY         1             -
3             BLOCKED       RUN:io        1             1
4             BLOCKED       BLOCKED       -             2
5             BLOCKED       BLOCKED       -             2
6             BLOCKED       BLOCKED       -             2
7             BLOCKED       BLOCKED       -             2
8*            RUN:io_done   BLOCKED       1             1
9*            READY         RUN:io_done   1             -
10            READY         RUN:io        1             -
11            RUN:cpu       BLOCKED       1             1
12            DONE          BLOCKED       -             1
13            DONE          BLOCKED       -             1
14            DONE          BLOCKED       -             1
15            DONE          BLOCKED       -             1
16*           DONE          RUN:io_done   1             -
17            DONE          RUN:cpu       1             -
```

- Reasoning / 理由: PID 1 preempts PID 0 at tick 9 and starts its second I/O at tick 10; PID 0 can finish during that wait. / PID 1立即运行并更早发起下一次I/O，总时间缩短1 tick。
- Verified result / 验证结果: 17 ticks; CPU busy 9/17 (52.94%), from `q8-s3.txt` (IO_RUN_IMMEDIATE).
- Analysis / 分析: This is the one seed where immediate resumption helps. PID 1 takes tick 9 and starts its next I/O at tick 10, one tick earlier than in the default run; PID 0 finishes while that I/O is in progress. / PID 1提前发起I/O，PID 0的最后一步便可与等待重叠。

### Seed 3 — -S SWITCH_ON_END
- Prediction / 预测: total **24**, CPU busy **9/24 = 37.50%**.

```text
Time          PID 0         PID 1         CPU           IOs
1             RUN:cpu       READY         1             -
2             RUN:io        READY         1             -
3             BLOCKED       READY         -             1
4             BLOCKED       READY         -             1
5             BLOCKED       READY         -             1
6             BLOCKED       READY         -             1
7             BLOCKED       READY         -             1
8*            RUN:io_done   READY         1             -
9             RUN:cpu       READY         1             -
10            DONE          RUN:io        1             -
11            DONE          BLOCKED       -             1
12            DONE          BLOCKED       -             1
13            DONE          BLOCKED       -             1
14            DONE          BLOCKED       -             1
15            DONE          BLOCKED       -             1
16*           DONE          RUN:io_done   1             -
17            DONE          RUN:io        1             -
18            DONE          BLOCKED       -             1
19            DONE          BLOCKED       -             1
20            DONE          BLOCKED       -             1
21            DONE          BLOCKED       -             1
22            DONE          BLOCKED       -             1
23*           DONE          RUN:io_done   1             -
24            DONE          RUN:cpu       1             -
```

- Reasoning / 理由: PID 1 waits until PID 0 finishes, then its two I/O waits have no other process to overlap them. / PID 1等PID 0结束后才开始，两次I/O等待无法与CPU工作重叠。
- Verified result / 验证结果: 24 ticks; CPU busy 9/24 (37.50%), from `q8-s3.txt` (SWITCH_ON_END).
- Analysis / 分析: PID 1 waits until tick 10 to start. Once PID 0 is done, nothing else can run during PID 1's two I/O waits, which accounts for most of the extra time. / PID 1开始得晚，其两次I/O等待都没有其他工作可填补。

### Q8 policy comparison / 策略比较

| Seed | default | IO_RUN_IMMEDIATE | SWITCH_ON_END |
|---|---:|---:|---:|
| 1 | 15 ticks, 53.33% | 15 ticks, 53.33% | 18 ticks, 44.44% |
| 2 | 16 ticks, 62.50% | 16 ticks, 62.50% | 30 ticks, 33.33% |
| 3 | 18 ticks, 50.00% | 17 ticks, 52.94% | 24 ticks, 37.50% |

Switching on I/O overlaps work and waiting, while immediate I/O resumption helps only when a completion competes with a running process (seed 3, tick 9). / I/O时切换可让计算与等待重叠；立即恢复只在完成时与正在运行的进程竞争CPU时改变结果（种子3第9 tick）。
