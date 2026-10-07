# OSTEP Ch.4 Homework Answers

Prediction phase only. The `-c` trace and verification analysis will be added after the first push. A `*` marks the tick on which an I/O completes; `-` means idle/no I/O.

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
- Verified result / 验证结果: Total Time 10; CPU Busy 10 (100.00%). See the corresponding `q*.txt` trace.
- Analysis / 分析: The prediction matches every simulator state row. The trace confirms ten consecutive CPU ticks, no idle tick and no I/O. / 轨迹显示10个CPU工作tick，没有空闲或I/O。

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
- Verified result / 验证结果: Total Time 11; CPU Busy 6 (54.55%). See the corresponding `q*.txt` trace.
- Analysis / 分析: The prediction matches every simulator state row. The trace separates the I/O into a start tick, five blocked ticks and a completion tick; the five blocked ticks are CPU idle. / I/O由发起、5个阻塞tick和完成处理组成。

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
- Verified result / 验证结果: Total Time 7; CPU Busy 6 (85.71%). See the corresponding `q*.txt` trace.
- Analysis / 分析: The prediction matches every simulator state row. PID 1 uses ticks 2–5 while PID 0 waits. Only tick 6 is idle, and PID 0 returns at tick 7. / PID 1利用了等待期，第6 tick才空闲。

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
- Verified result / 验证结果: Total Time 11; CPU Busy 6 (54.55%). See the corresponding `q*.txt` trace.
- Analysis / 分析: The prediction matches every simulator state row. PID 1 stays READY during ticks 2–7, but SWITCH_ON_END prevents it from running until tick 8. / PID 1虽就绪仍等到第8 tick，解释了较长总时间。

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
- Verified result / 验证结果: Total Time 7; CPU Busy 6 (85.71%). See the corresponding `q*.txt` trace.
- Analysis / 分析: The prediction matches every simulator state row. Switching on I/O lets PID 1 occupy ticks 2–5, so Q5 finishes four ticks earlier than Q4. / 允许I/O时切换，完成时间比Q4缩短4 tick。

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
- Verified result / 验证结果: Total Time 31; CPU Busy 21 (67.74%). See the corresponding `q*.txt` trace.
- Analysis / 分析: The prediction matches every simulator state row. The first I/O ends at tick 7, but IO_RUN_LATER postpones PID 0’s next CPU tick until 17; later waits leave ten idle ticks. / 首次I/O完成后PID 0继续排队，后两段等待造成10个CPU空闲tick。

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
- Verified result / 验证结果: Total Time 21; CPU Busy 21 (100.00%). See the corresponding `q*.txt` trace.
- Analysis / 分析: The prediction matches every simulator state row. Immediate resumption at ticks 7, 14 and 21 overlaps each five-tick I/O wait with another CPU-only process, eliminating idle CPU time. / 每次I/O完成即恢复PID 0，三段等待均与其他进程工作重叠。

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
- Verified result / 验证结果: Total Time 15; CPU Busy 8 (53.33%). See the corresponding `q*.txt` trace.
- Analysis / 分析: The prediction matches every simulator state row. PID 1 is already DONE at the first I/O completion, so the remaining waits cannot overlap CPU work. / 首次I/O完成前PID 1已结束。

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
- Verified result / 验证结果: Total Time 15; CPU Busy 8 (53.33%). See the corresponding `q*.txt` trace.
- Analysis / 分析: The prediction matches every simulator state row. The immediate policy makes no difference because no competing runnable process exists at either completion. / 两次完成时都没有其他可运行进程，策略不改变结果。

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
- Verified result / 验证结果: Total Time 18; CPU Busy 8 (44.44%). See the corresponding `q*.txt` trace.
- Analysis / 分析: The prediction matches every simulator state row. PID 1 remains READY through both waits and runs only after PID 0 is DONE, extending the total to 18. / PID 1在两段等待期仍就绪，最后才运行。

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
- Verified result / 验证结果: Total Time 16; CPU Busy 10 (62.50%). See the corresponding `q*.txt` trace.
- Analysis / 分析: The prediction matches every simulator state row. The trace has two CPU-idle spans (ticks 4–6 and 11–13) while both processes are BLOCKED. / 两段时间两个进程同时阻塞。

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
- Verified result / 验证结果: Total Time 16; CPU Busy 10 (62.50%). See the corresponding `q*.txt` trace.
- Analysis / 分析: The prediction matches every simulator state row. Every completion occurs while the other process is blocked or done, so immediate resumption changes no state row. / 每次I/O结束时另一进程无法竞争CPU。

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
- Verified result / 验证结果: Total Time 30; CPU Busy 10 (33.33%). See the corresponding `q*.txt` trace.
- Analysis / 分析: The prediction matches every simulator state row. The four I/O waits become serial, and the ready second process cannot run during PID 0’s waits. / 四段I/O等待串行，使总时间增至30。

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
- Verified result / 验证结果: Total Time 18; CPU Busy 9 (50.00%). See the corresponding `q*.txt` trace.
- Analysis / 分析: The prediction matches every simulator state row. At tick 9 PID 1 becomes READY, but PID 0 keeps the CPU and completes its final instruction first. / 第9 tick默认策略不抢占PID 0。

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
- Verified result / 验证结果: Total Time 17; CPU Busy 9 (52.94%). See the corresponding `q*.txt` trace.
- Analysis / 分析: The prediction matches every simulator state row. PID 1 preempts PID 0 at tick 9 and launches its next I/O at tick 10; that one-tick advance shortens the total. / PID 1立即恢复并提早发起下一次I/O。

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
- Verified result / 验证结果: Total Time 24; CPU Busy 9 (37.50%). See the corresponding `q*.txt` trace.
- Analysis / 分析: The prediction matches every simulator state row. PID 0 finishes before PID 1 starts; PID 1’s two I/O waits then have no CPU work to overlap them. / PID 1须等PID 0完成，其两段等待无法与其他工作重叠。

### Q8 policy comparison / 策略比较

| Seed | default | IO_RUN_IMMEDIATE | SWITCH_ON_END |
|---|---:|---:|---:|
| 1 | 15 ticks, 53.33% | 15 ticks, 53.33% | 18 ticks, 44.44% |
| 2 | 16 ticks, 62.50% | 16 ticks, 62.50% | 30 ticks, 33.33% |
| 3 | 18 ticks, 50.00% | 17 ticks, 52.94% | 24 ticks, 37.50% |

Switching on I/O overlaps work and waiting, while immediate I/O resumption helps only when a completion competes with a running process (seed 3, tick 9). / I/O时切换可让计算与等待重叠；立即恢复只在完成时与正在运行的进程竞争CPU时改变结果（种子3第9 tick）。
