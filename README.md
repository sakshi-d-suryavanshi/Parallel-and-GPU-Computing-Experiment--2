# Multithreaded Programming Using Pthreads and OpenMP

## Experiment

**Develop Multithreaded Programs Using Parallel Programming Libraries to Understand Thread Creation, Management, and Coordination**

---

## 1. Aim

To develop multithreaded programs using **Pthreads and OpenMP** and understand:

- Thread creation
- Thread management
- Work distribution
- Race conditions
- Synchronization
- Thread coordination
- Performance improvement using multiple threads

---

## 2. Software Environment

The experiment was performed using:

- Windows
- WSL Ubuntu
- GCC compiler
- Pthreads
- OpenMP
- Nano editor

---

## 3. Basic Idea

A thread is an execution path inside a program.

In a sequential program, one thread performs the tasks one after another.

In a multithreaded program, multiple threads can work on different parts of the problem concurrently.

This experiment uses two parallel programming technologies:

- **Pthreads** — POSIX Threads
- **OpenMP** — Open Multi-Processing

The final part compares sequential, Pthreads, and OpenMP execution to analyze the performance improvement obtained using multiple threads.

---

# Part A — Pthreads

## 4. Create One Thread

### Program

**File:** `thread1.c`

The program creates one additional thread using `pthread_create()` and waits for it using `pthread_join()`.

### Compile

```bash
gcc thread1.c -o thread1 -pthread
```

### Run

```bash
./thread1
```

### Expected Output

```text
Hello from the thread!
Main thread finished.
```

### Concepts Demonstrated

- `pthread_t`
- `pthread_create()`
- `pthread_join()`
- Main thread
- Additional thread

---

## 5. Create Multiple Threads

### Program

**File:** `thread2.c`

Four additional threads are created using `pthread_create()`.

### Compile

```bash
gcc thread2.c -o thread2 -pthread
```

### Run

```bash
./thread2
```

### Expected Output

```text
Hello from Thread 1
Hello from Thread 2
Hello from Thread 4
Hello from Thread 3
All threads have finished.
```

The order of thread messages may change because the operating system controls thread scheduling.

### Concepts Demonstrated

- Multiple thread creation
- Thread identifiers
- Thread scheduling
- Thread joining

---

## 6. Divide Work Among Threads

### Program

**File:** `thread_sum.c`

The array

```text
10 20 30 40 50 60 70 80
```

is divided among four threads.

### Work Distribution

| Thread | Elements | Partial Sum |
|---|---|---:|
| Thread 1 | 10, 20 | 30 |
| Thread 2 | 30, 40 | 70 |
| Thread 3 | 50, 60 | 110 |
| Thread 4 | 70, 80 | 150 |
| **Total** | | **360** |

### Compile

```bash
gcc thread_sum.c -o thread_sum -pthread
```

### Run

```bash
./thread_sum
```

### Expected Result

```text
Thread 1 calculated sum = 30
Thread 2 calculated sum = 70
Thread 3 calculated sum = 110
Thread 4 calculated sum = 150
Total sum = 360
```

The order of the thread messages may vary.

### Concept Demonstrated

**Work distribution** — a large task is divided into smaller tasks and assigned to different threads.

---

## 7. Race Condition

### Program

**File:** `race.c`

Four threads increment the same shared variable:

```c
counter++;
```

Each thread performs 100,000 increments.

### Compile

```bash
gcc race.c -o race -pthread
```

### Run

```bash
./race
```

### Expected Result

The theoretical expected value is:

```text
Expected counter = 400000
```

However, the actual result may be lower and can change between executions.

Example:

```text
Expected counter = 400000
Actual counter   = 167739
```

### Reason

All threads access and modify the same shared variable simultaneously.

This can cause a **race condition**, where updates from different threads are lost.

---

## 8. Fix Race Condition Using Mutex

### Program

**File:** `mutex.c`

A Pthreads mutex is used to protect the shared counter.

### Important Section

```c
pthread_mutex_lock(&mutex);

counter++;

pthread_mutex_unlock(&mutex);
```

### Compile

```bash
gcc mutex.c -o mutex -pthread
```

### Run

```bash
./mutex
```

### Expected Output

```text
Expected counter = 400000
Actual counter   = 400000
```

### Concept Demonstrated

A **mutex** allows only one thread at a time to access the protected critical section.

---

# Part B — OpenMP

## 9. OpenMP Basic Parallel Region

### Program

**File:** `omp1.c`

The program creates an OpenMP parallel region and identifies each thread.

### Compile

```bash
gcc omp1.c -o omp1 -fopenmp
```

### Run

```bash
./omp1
```

### Example Output

```text
Hello from Thread 18 of 32
Hello from Thread 13 of 32
Hello from Thread 19 of 32
...
Hello from Thread 0 of 32
```

The exact number and order of threads depend on the system and OpenMP runtime configuration.

### Concepts Demonstrated

- `#pragma omp parallel`
- `omp_get_thread_num()`
- `omp_get_num_threads()`
- OpenMP thread team

---

## 10. OpenMP Work Sharing

### Program

**File:** `omp_sum.c`

The same array used in the Pthreads experiment is processed using OpenMP.

### Compile

```bash
gcc omp_sum.c -o omp_sum -fopenmp
```

### Run

```bash
./omp_sum
```

### Expected Result

```text
Total sum = 360
```

### Important Directive

```c
#pragma omp parallel for reduction(+:total_sum)
```

`parallel for` distributes loop iterations among OpenMP threads.

The `reduction` clause safely combines the partial sums produced by the threads.

### Concept Demonstrated

- Automatic work distribution
- Parallel loops
- Reduction

---

## 11. OpenMP Race Condition

### Program

**File:** `omp_race.c`

Multiple OpenMP threads modify the same shared counter without synchronization.

### Compile

```bash
gcc omp_race.c -o omp_race -fopenmp
```

### Run

```bash
./omp_race
```

### Expected Result

```text
Expected counter = 400000
Actual counter   = 100000
```

The exact incorrect result can vary depending on execution timing.

### Concept Demonstrated

OpenMP automatically manages threads, but shared-data operations are not automatically safe.

---

## 12. OpenMP Critical Section

### Program

**File:** `omp_critical.c`

The race condition is fixed using:

```c
#pragma omp critical
{
    counter++;
}
```

### Compile

```bash
gcc omp_critical.c -o omp_critical -fopenmp
```

### Run

```bash
./omp_critical
```

### Expected Output

```text
Expected counter = 400000
Actual counter   = 400000
```

### Concept Demonstrated

A critical section allows only one OpenMP thread at a time to execute the protected block.

---

## 13. OpenMP Barrier

### Program

**File:** `omp_barrier.c`

The program demonstrates synchronization between threads using a barrier.

### Compile

```bash
gcc omp_barrier.c -o omp_barrier -fopenmp
```

### Run

```bash
./omp_barrier
```

### Example Output

```text
Thread 0 completed Stage 1
Thread 3 completed Stage 1
Thread 1 completed Stage 1
Thread 2 completed Stage 1
Thread 3 started Stage 2
Thread 2 started Stage 2
Thread 0 started Stage 2
Thread 1 started Stage 2
```

The order may change.

However, all threads must reach the barrier before continuing to Stage 2.

### Concept Demonstrated

**Barrier synchronization** — a thread waits until all threads reach the synchronization point.

---

# Part C — Performance Analysis

## 14. Sequential Baseline

### Program

**File:** `sequential.c`

A large computational workload is executed using a single thread.

### Compile

```bash
gcc sequential.c -o sequential
```

### Run

```bash
./sequential
```

### Result

```text
Result = 499999999500.00
```

Five measured executions were:

| Run | Time (s) |
|---|---:|
| 1 | 1.353895 |
| 2 | 1.355794 |
| 3 | 1.349621 |
| 4 | 1.353422 |
| 5 | 1.353365 |

### Average Sequential Time

```text
1.353219 seconds
```

This value is used as the **sequential baseline**.

---

# 15. Pthreads Performance

### Program

**File:** `pthread_perf.c`

The same computational workload is divided among different numbers of Pthreads.

### Compile

```bash
gcc pthread_perf.c -o pthread_perf -pthread
```

### Run

```bash
./pthread_perf
```

The program accepts the number of threads as input.

### Measured Results

| Threads | Pthreads Time |
|---:|---:|
| 1 | 1.348142 s |
| 2 | 0.680737 s |
| 4 | 0.358872 s |
| 6 | 0.241345 s |
| 16 | 0.144812 s |

The execution time generally decreases as the number of threads increases.

---

# 16. OpenMP Performance

### Program

**File:** `omp_perf.c`

The same computational workload is executed using OpenMP.

### Compile

```bash
gcc omp_perf.c -o omp_perf -fopenmp
```

### Run

```bash
./omp_perf
```

### Measured Results

| Threads | OpenMP Time |
|---:|---:|
| 1 | 1.409294 s |
| 2 | 0.715560 s |
| 4 | 0.360803 s |
| 6 | 0.241608 s |
| 16 | 0.140692 s |

---

# 17. Final Performance Comparison

| Threads | Pthreads Time (s) | OpenMP Time (s) |
|---:|---:|---:|
| 1 | 1.348142 | 1.409294 |
| 2 | 0.680737 | 0.715560 |
| 4 | 0.358872 | 0.360803 |
| 6 | 0.241345 | 0.241608 |
| 16 | 0.144812 | 0.140692 |

### Observation

As the number of threads increases, the execution time generally decreases for both Pthreads and OpenMP.

The best measured execution time was obtained using **OpenMP with 16 threads**:

```text
0.140692 seconds
```

---

# 18. Speedup

Speedup is calculated using:

```text
Speedup = Sequential Time / Parallel Time
```

The sequential baseline is:

```text
1.353219 seconds
```

### Speedup Results

| Threads | Pthreads Speedup | OpenMP Speedup |
|---:|---:|---:|
| 1 | 1.004x | 0.960x |
| 2 | 1.988x | 1.891x |
| 4 | 3.771x | 3.751x |
| 6 | 5.608x | 5.601x |
| 16 | 9.345x | 9.618x |

### Observation

Speedup increases as the number of threads increases.

At 16 threads:

- Pthreads achieved approximately **9.35x speedup**
- OpenMP achieved approximately **9.62x speedup**

---

# 19. Efficiency

Efficiency is calculated using:

```text
Efficiency = (Speedup / Number of Threads) × 100
```

### Efficiency Results

| Threads | Pthreads Efficiency | OpenMP Efficiency |
|---:|---:|---:|
| 1 | 100.38% | 96.02% |
| 2 | 99.39% | 94.56% |
| 4 | 94.27% | 93.76% |
| 6 | 93.45% | 93.35% |
| 16 | 58.40% | 60.11% |

### Observation

Efficiency is relatively high at lower thread counts but decreases at 16 threads.

This shows that increasing the number of threads does not always provide proportional speedup.

---

# 20. Why Does 16 Threads Not Give 16x Speedup?

Ideally, using 16 threads might suggest a 16x speedup.

However, the measured OpenMP speedup was:

```text
9.618x
```

The difference occurs because parallel programs have overhead such as:

- Thread management
- Scheduling
- Synchronization
- Memory access
- Operating-system activity
- Non-parallel portions of the program

Therefore, adding more threads can improve performance, but the speedup is generally not perfectly linear.

---

# 21. Pthreads vs OpenMP

| Concept | Pthreads | OpenMP |
|---|---|---|
| Create threads | `pthread_create()` | `#pragma omp parallel` |
| Wait for threads | `pthread_join()` | Runtime handles team completion |
| Work distribution | Programmer explicitly divides work | `parallel for` distributes loop iterations |
| Protect shared data | Mutex | `critical` |
| Coordination | Join / synchronization mechanisms | `barrier` |
| Combine partial results | Programmer-managed | `reduction` |

---

# 22. Implemented Programs

## Pthreads

| File | Purpose |
|---|---|
| `thread1.c` | Create one thread |
| `thread2.c` | Create multiple threads |
| `thread_sum.c` | Divide work among threads |
| `race.c` | Demonstrate race condition |
| `mutex.c` | Fix race condition using mutex |
| `pthread_perf.c` | Measure Pthreads performance |

## OpenMP

| File | Purpose |
|---|---|
| `omp1.c` | Parallel region and thread identification |
| `omp_sum.c` | Work sharing and reduction |
| `omp_race.c` | Demonstrate race condition |
| `omp_critical.c` | Synchronization using critical |
| `omp_barrier.c` | Thread coordination |
| `omp_perf.c` | Measure OpenMP performance |

## Performance Analysis

- Sequential execution
- Pthreads execution
- OpenMP execution
- Execution-time comparison
- Speedup calculation
- Efficiency calculation
- Performance interpretation

---

# 23. Important Terms

### Thread

A path of execution inside a program.

### Main Thread

The thread that starts executing the `main()` function.

### Multithreading

Using multiple threads within one program.

### Work Distribution

Dividing a large task into smaller tasks and assigning them to different threads.

### Race Condition

A situation where multiple threads access or modify shared data without proper coordination, potentially producing an incorrect result.

### Mutex

A locking mechanism used in Pthreads to protect a critical section.

### Critical Section

A section of code where simultaneous execution by multiple threads must be restricted.

### Barrier

A synchronization point where threads wait until all required threads reach the same point.

### Speedup

The ratio of sequential execution time to parallel execution time.

### Efficiency

A measure of how effectively the available threads produce the measured speedup.

---

# 24. Result

The experiment successfully demonstrated:

- Thread creation using Pthreads
- Multiple thread management
- Work distribution
- Race conditions
- Mutex synchronization
- OpenMP parallel regions
- OpenMP work sharing
- Reduction
- Critical sections
- Barrier synchronization
- Performance comparison
- Speedup analysis
- Efficiency analysis

The best measured result was:

```text
OpenMP
16 Threads
Execution Time = 0.140692 seconds
Speedup = 9.618x
Efficiency = 60.11%
```

---

# 25. Conclusion

The experiment successfully implemented multithreaded programs using **Pthreads and OpenMP**.

Pthreads provides explicit control over thread creation, joining, and synchronization, while OpenMP provides a higher-level approach using parallel programming directives.

The race-condition experiments demonstrated the importance of synchronization when multiple threads access shared data. Mutexes and critical sections were used to protect shared operations.

The performance analysis showed that increasing the number of threads can significantly reduce execution time for a suitable computational workload. In the measured experiment, OpenMP with 16 threads achieved the best performance with an execution time of **0.140692 seconds** and approximately **9.62x speedup** compared with the sequential baseline.

The experiment demonstrates the practical use of multithreading for **parallel execution, synchronization, coordination, and performance improvement**.
