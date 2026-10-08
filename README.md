# Multithreaded Programming Using Pthreads and OpenMP

## Experiment

Develop Multithreaded Programs Using Parallel Programming Libraries to Understand Thread Creation, Management, and Coordination.

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

## 2. Basic Idea

A **thread** is an execution path inside a program.

A sequential program uses one thread to perform all tasks:

```text
One Worker
    |
    |---- Task 1
    |---- Task 2
    |---- Task 3
    |---- Task 4
```

A multithreaded program divides work among multiple threads:

```text
                 Program
                    |
          ---------------------
          |     |     |       |
       Thread1 Thread2 Thread3 Thread4
          |     |     |       |
        Work   Work  Work     Work
```

This experiment uses:

- **Pthreads** — explicit thread creation and management
- **OpenMP** — higher-level parallel programming using compiler directives

---

## 3. Software Environment

| Software | Used For |
|---|---|
| Windows | Host operating system |
| WSL Ubuntu | Linux environment |
| GCC | C compiler |
| Pthreads | Thread programming |
| OpenMP | Parallel programming |
| Nano | C program editing |

---

# 4. Setup

## 4.1 Start WSL

Opens the Ubuntu Linux environment through WSL.

```bash
wsl
```

## 4.2 Create the Lab Directory

Creates the directory where all experiment programs will be stored.

```bash
mkdir -p ~/parallel_lab
```

Moves into the experiment directory.

```bash
cd ~/parallel_lab
```

Displays the current working directory.

```bash
pwd
```

Expected:

```text
/home/<username>/parallel_lab
```

## 4.3 Check GCC

Checks whether the GCC C compiler is installed.

```bash
gcc --version
```

## 4.4 Check OpenMP Support

Checks GCC with the OpenMP compilation option.

```bash
gcc -fopenmp --version
```

---

# PART A — PTHREADS

Pthreads stands for **POSIX Threads**.

Pthreads provides explicit control over:

- Thread creation
- Thread joining
- Synchronization
- Mutex locking
- Work distribution

Important functions:

```text
pthread_create()
pthread_join()
pthread_mutex_lock()
pthread_mutex_unlock()
```

---

# 5. Create One Thread

## Objective

To understand thread creation, thread execution, and waiting for a thread using `pthread_join()`.

### Create the Program

Opens Nano to create the first Pthreads program.

```bash
nano thread1.c
```

Paste:

```c
#include <stdio.h>
#include <pthread.h>

void *thread_function(void *arg)
{
    printf("Hello from the thread!\n");
    return NULL;
}

int main()
{
    pthread_t thread;

    pthread_create(&thread, NULL, thread_function, NULL);

    pthread_join(thread, NULL);

    printf("Main thread finished.\n");

    return 0;
}
```

Save using:

```text
Ctrl + O
Enter
Ctrl + X
```

### Compile

Compiles the program and links the Pthreads library.

```bash
gcc thread1.c -o thread1 -pthread
```

### Run

Executes the compiled Pthreads program.

```bash
./thread1
```

Expected output:

```text
Hello from the thread!
Main thread finished.
```

---

# 6. Create Multiple Threads

### Create the Program

Opens Nano to create a program that creates four Pthreads.

```bash
nano thread2.c
```

Paste:

```c
#include <stdio.h>
#include <pthread.h>

void *thread_function(void *arg)
{
    int thread_id = *(int *)arg;

    printf("Hello from Thread %d\n", thread_id);

    return NULL;
}

int main()
{
    pthread_t threads[4];
    int thread_ids[4];

    for (int i = 0; i < 4; i++)
    {
        thread_ids[i] = i + 1;

        pthread_create(
            &threads[i],
            NULL,
            thread_function,
            &thread_ids[i]
        );
    }

    for (int i = 0; i < 4; i++)
    {
        pthread_join(threads[i], NULL);
    }

    printf("All threads have finished.\n");

    return 0;
}
```

Save:

```text
Ctrl + O
Enter
Ctrl + X
```

### Compile

Compiles the multiple-thread program using Pthreads.

```bash
gcc thread2.c -o thread2 -pthread
```

### Run

Executes the program and creates four threads.

```bash
./thread2
```

Example output:

```text
Hello from Thread 1
Hello from Thread 3
Hello from Thread 2
Hello from Thread 4
All threads have finished.
```

The order may change because the operating system controls thread scheduling.

---

# 7. Divide Work Among Threads

The array used is:

```text
10 20 30 40 50 60 70 80
```

Work distribution:

```text
Thread 1 → 10 + 20 = 30
Thread 2 → 30 + 40 = 70
Thread 3 → 50 + 60 = 110
Thread 4 → 70 + 80 = 150

Total = 360
```

### Create the Program

Opens Nano to create the program that divides array-sum work among threads.

```bash
nano thread_sum.c
```

Paste:

```c
#include <stdio.h>
#include <pthread.h>

#define NUM_THREADS 4
#define ARRAY_SIZE 8

int array[ARRAY_SIZE] = {10, 20, 30, 40, 50, 60, 70, 80};

int partial_sum[NUM_THREADS];

void *calculate_sum(void *arg)
{
    int thread_id = *(int *)arg;

    int start = thread_id * (ARRAY_SIZE / NUM_THREADS);
    int end = start + (ARRAY_SIZE / NUM_THREADS);

    partial_sum[thread_id] = 0;

    for (int i = start; i < end; i++)
    {
        partial_sum[thread_id] += array[i];
    }

    printf("Thread %d calculated sum = %d\n",
           thread_id + 1,
           partial_sum[thread_id]);

    return NULL;
}

int main()
{
    pthread_t threads[NUM_THREADS];
    int thread_ids[NUM_THREADS];

    for (int i = 0; i < NUM_THREADS; i++)
    {
        thread_ids[i] = i;

        pthread_create(
            &threads[i],
            NULL,
            calculate_sum,
            &thread_ids[i]
        );
    }

    for (int i = 0; i < NUM_THREADS; i++)
    {
        pthread_join(threads[i], NULL);
    }

    int total_sum = 0;

    for (int i = 0; i < NUM_THREADS; i++)
    {
        total_sum += partial_sum[i];
    }

    printf("Total sum = %d\n", total_sum);

    return 0;
}
```

Save:

```text
Ctrl + O
Enter
Ctrl + X
```

### Compile

Compiles the work-distribution program using Pthreads.

```bash
gcc thread_sum.c -o thread_sum -pthread
```

### Run

Executes the program and calculates the partial and total sums.

```bash
./thread_sum
```

Expected:

```text
Thread 1 → 30
Thread 2 → 70
Thread 3 → 110
Thread 4 → 150
Total → 360
```

---

# 8. Demonstrate a Race Condition

A race condition occurs when multiple threads access or modify shared data at the same time without proper synchronization.

### Create the Program

Opens Nano to create a program that demonstrates a race condition.

```bash
nano race.c
```

Paste:

```c
#include <stdio.h>
#include <pthread.h>

#define NUM_THREADS 4
#define INCREMENTS 100000

int counter = 0;

void *increment_counter(void *arg)
{
    for (int i = 0; i < INCREMENTS; i++)
    {
        counter++;
    }

    return NULL;
}

int main()
{
    pthread_t threads[NUM_THREADS];

    for (int i = 0; i < NUM_THREADS; i++)
    {
        pthread_create(
            &threads[i],
            NULL,
            increment_counter,
            NULL
        );
    }

    for (int i = 0; i < NUM_THREADS; i++)
    {
        pthread_join(threads[i], NULL);
    }

    printf("Expected counter = %d\n",
           NUM_THREADS * INCREMENTS);

    printf("Actual counter   = %d\n",
           counter);

    return 0;
}
```

Save:

```text
Ctrl + O
Enter
Ctrl + X
```

### Compile

Compiles the race-condition program using Pthreads.

```bash
gcc race.c -o race -pthread
```

### Run

Executes the program to observe the effect of unsynchronized shared data.

```bash
./race
```

Expected counter:

```text
400000
```

The actual value may be lower and may change between runs.

Example:

```text
Expected counter = 400000
Actual counter   = 167739
```

---

# 9. Fix Race Condition Using Mutex

A mutex allows only one thread at a time to enter the protected section.

### Create the Program

Opens Nano to create the program that protects the shared counter using a mutex.

```bash
nano mutex.c
```

Paste:

```c
#include <stdio.h>
#include <pthread.h>

#define NUM_THREADS 4
#define INCREMENTS 100000

int counter = 0;

pthread_mutex_t mutex;

void *increment_counter(void *arg)
{
    for (int i = 0; i < INCREMENTS; i++)
    {
        pthread_mutex_lock(&mutex);

        counter++;

        pthread_mutex_unlock(&mutex);
    }

    return NULL;
}

int main()
{
    pthread_t threads[NUM_THREADS];

    pthread_mutex_init(&mutex, NULL);

    for (int i = 0; i < NUM_THREADS; i++)
    {
        pthread_create(
            &threads[i],
            NULL,
            increment_counter,
            NULL
        );
    }

    for (int i = 0; i < NUM_THREADS; i++)
    {
        pthread_join(threads[i], NULL);
    }

    pthread_mutex_destroy(&mutex);

    printf("Expected counter = %d\n",
           NUM_THREADS * INCREMENTS);

    printf("Actual counter   = %d\n",
           counter);

    return 0;
}
```

Save:

```text
Ctrl + O
Enter
Ctrl + X
```

### Compile

Compiles the mutex synchronization program using Pthreads.

```bash
gcc mutex.c -o mutex -pthread
```

### Run

Executes the synchronized program and checks the final counter.

```bash
./mutex
```

Expected:

```text
Expected counter = 400000
Actual counter   = 400000
```

---

# PART B — OPENMP

OpenMP provides a higher-level approach to parallel programming.

Important directives and functions:

```text
#pragma omp parallel
#pragma omp parallel for
#pragma omp critical
#pragma omp barrier
omp_get_thread_num()
omp_get_num_threads()
```

---

# 10. OpenMP Basic Parallel Region

### Create the Program

Opens Nano to create the basic OpenMP parallel-region program.

```bash
nano omp1.c
```

Paste:

```c
#include <stdio.h>
#include <omp.h>

int main()
{
    #pragma omp parallel
    {
        int thread_id = omp_get_thread_num();
        int total_threads = omp_get_num_threads();

        printf("Hello from Thread %d of %d\n",
               thread_id,
               total_threads);
    }

    return 0;
}
```

Save:

```text
Ctrl + O
Enter
Ctrl + X
```

### Compile

Compiles the program and enables OpenMP support.

```bash
gcc omp1.c -o omp1 -fopenmp
```

### Run

Executes the OpenMP program and displays thread information.

```bash
./omp1
```

---

# 11. OpenMP Work Sharing

### Create the Program

Opens Nano to create the OpenMP work-sharing program.

```bash
nano omp_sum.c
```

Paste:

```c
#include <stdio.h>
#include <omp.h>

#define ARRAY_SIZE 8

int array[ARRAY_SIZE] = {10, 20, 30, 40, 50, 60, 70, 80};

int main()
{
    int total_sum = 0;

    #pragma omp parallel for reduction(+:total_sum)
    for (int i = 0; i < ARRAY_SIZE; i++)
    {
        int thread_id = omp_get_thread_num();

        printf("Thread %d processing array[%d] = %d\n",
               thread_id,
               i,
               array[i]);

        total_sum += array[i];
    }

    printf("Total sum = %d\n", total_sum);

    return 0;
}
```

Save:

```text
Ctrl + O
Enter
Ctrl + X
```

### Compile

Compiles the OpenMP work-sharing program.

```bash
gcc omp_sum.c -o omp_sum -fopenmp
```

### Run

Executes the program and displays the distributed array processing.

```bash
./omp_sum
```

Expected:

```text
Total sum = 360
```

---

# 12. OpenMP Race Condition

### Create the Program

Opens Nano to create the OpenMP race-condition program.

```bash
nano omp_race.c
```

Paste:

```c
#include <stdio.h>
#include <omp.h>

#define NUM_THREADS 4
#define INCREMENTS 100000

int counter = 0;

int main()
{
    omp_set_num_threads(NUM_THREADS);

    #pragma omp parallel
    {
        for (int i = 0; i < INCREMENTS; i++)
        {
            counter++;
        }
    }

    printf("Expected counter = %d\n",
           NUM_THREADS * INCREMENTS);

    printf("Actual counter   = %d\n",
           counter);

    return 0;
}
```

Save:

```text
Ctrl + O
Enter
Ctrl + X
```

### Compile

Compiles the OpenMP race-condition program.

```bash
gcc omp_race.c -o omp_race -fopenmp
```

### Run

Executes the program to observe the race condition.

```bash
./omp_race
```

Expected:

```text
Expected counter = 400000
Actual counter   = ...
```

---

# 13. OpenMP Critical Section

### Create the Program

Opens Nano to create the OpenMP program that fixes the race condition using a critical section.

```bash
nano omp_critical.c
```

Paste:

```c
#include <stdio.h>
#include <omp.h>

#define NUM_THREADS 4
#define INCREMENTS 100000

int counter = 0;

int main()
{
    omp_set_num_threads(NUM_THREADS);

    #pragma omp parallel
    {
        for (int i = 0; i < INCREMENTS; i++)
        {
            #pragma omp critical
            {
                counter++;
            }
        }
    }

    printf("Expected counter = %d\n",
           NUM_THREADS * INCREMENTS);

    printf("Actual counter   = %d\n",
           counter);

    return 0;
}
```

Save:

```text
Ctrl + O
Enter
Ctrl + X
```

### Compile

Compiles the OpenMP critical-section program.

```bash
gcc omp_critical.c -o omp_critical -fopenmp
```

### Run

Executes the synchronized OpenMP program.

```bash
./omp_critical
```

Expected:

```text
Expected counter = 400000
Actual counter   = 400000
```

---

# 14. OpenMP Barrier

A barrier is a synchronization point where threads wait until all threads reach the same point.

### Create the Program

Opens Nano to create the OpenMP barrier coordination program.

```bash
nano omp_barrier.c
```

Paste:

```c
#include <stdio.h>
#include <omp.h>

int main()
{
    omp_set_num_threads(4);

    #pragma omp parallel
    {
        int thread_id = omp_get_thread_num();

        printf("Thread %d completed Stage 1\n", thread_id);

        #pragma omp barrier

        printf("Thread %d started Stage 2\n", thread_id);
    }

    return 0;
}
```

Save:

```text
Ctrl + O
Enter
Ctrl + X
```

### Compile

Compiles the OpenMP barrier program.

```bash
gcc omp_barrier.c -o omp_barrier -fopenmp
```

### Run

Executes the program and demonstrates thread coordination using a barrier.

```bash
./omp_barrier
```

---

# PART C — PERFORMANCE ANALYSIS

The performance section compares:

1. Sequential execution
2. Pthreads execution
3. OpenMP execution

---

# 15. Sequential Baseline

### Create the Program

Opens Nano to create the sequential program used as the performance baseline.

```bash
nano sequential.c
```

Paste:

```c
#include <stdio.h>
#include <time.h>

#define N 1000000000L

double get_time()
{
    struct timespec ts;

    clock_gettime(CLOCK_MONOTONIC, &ts);

    return ts.tv_sec + ts.tv_nsec / 1e9;
}

int main()
{
    double sum = 0.0;

    double start = get_time();

    for (long i = 0; i < N; i++)
    {
        sum += (double)i * 0.000001;
    }

    double end = get_time();

    printf("Result = %.2f\n", sum);
    printf("Execution time = %.6f seconds\n",
           end - start);

    return 0;
}
```

Save:

```text
Ctrl + O
Enter
Ctrl + X
```

### Compile

Compiles the sequential baseline program.

```bash
gcc sequential.c -o sequential
```

### Run

Executes the sequential program and measures its execution time.

```bash
./sequential
```

Measured average:

```text
1.353219 seconds
```

---

# 16. Pthreads Performance

### Create the Program

Opens Nano to create the Pthreads performance program.

```bash
nano pthread_perf.c
```

Paste:

```c
#include <stdio.h>
#include <pthread.h>
#include <time.h>

#define N 1000000000L

double partial_sum[32];

typedef struct
{
    int thread_id;
    long start;
    long end;
} ThreadData;

double get_time()
{
    struct timespec ts;

    clock_gettime(CLOCK_MONOTONIC, &ts);

    return ts.tv_sec + ts.tv_nsec / 1e9;
}

void *calculate(void *arg)
{
    ThreadData *data = (ThreadData *)arg;

    double sum = 0.0;

    for (long i = data->start; i < data->end; i++)
    {
        sum += (double)i * 0.000001;
    }

    partial_sum[data->thread_id] = sum;

    return NULL;
}

int main()
{
    int num_threads;

    printf("Enter number of threads: ");
    scanf("%d", &num_threads);

    if (num_threads < 1 || num_threads > 32)
    {
        printf("Please enter a value between 1 and 32.\n");
        return 1;
    }

    pthread_t threads[num_threads];
    ThreadData data[num_threads];

    long chunk = N / num_threads;

    double start_time = get_time();

    for (int i = 0; i < num_threads; i++)
    {
        data[i].thread_id = i;
        data[i].start = i * chunk;

        if (i == num_threads - 1)
            data[i].end = N;
        else
            data[i].end = (i + 1) * chunk;

        pthread_create(
            &threads[i],
            NULL,
            calculate,
            &data[i]
        );
    }

    for (int i = 0; i < num_threads; i++)
    {
        pthread_join(threads[i], NULL);
    }

    double total_sum = 0.0;

    for (int i = 0; i < num_threads; i++)
    {
        total_sum += partial_sum[i];
    }

    double end_time = get_time();

    printf("Result = %.2f\n", total_sum);
    printf("Execution time = %.6f seconds\n",
           end_time - start_time);

    return 0;
}
```

Save:

```text
Ctrl + O
Enter
Ctrl + X
```

### Compile

Compiles the Pthreads performance program and links the Pthreads library.

```bash
gcc pthread_perf.c -o pthread_perf -pthread
```

### Run

Runs the Pthreads program for different thread counts.

```bash
./pthread_perf
```

Enter:

```text
1
```

Repeat for:

```text
2
4
6
16
```

### Pthreads Results

| Threads | Execution Time |
|---:|---:|
| 1 | 1.348142 s |
| 2 | 0.680737 s |
| 4 | 0.358872 s |
| 6 | 0.241345 s |
| 16 | 0.144812 s |

---

# 17. OpenMP Performance

### Create the Program

Opens Nano to create the OpenMP performance program.

```bash
nano omp_perf.c
```

Paste:

```c
#include <stdio.h>
#include <omp.h>
#include <time.h>

#define N 1000000000L

double get_time()
{
    struct timespec ts;

    clock_gettime(CLOCK_MONOTONIC, &ts);

    return ts.tv_sec + ts.tv_nsec / 1e9;
}

int main()
{
    double sum = 0.0;
    int num_threads;

    printf("Enter number of threads: ");
    scanf("%d", &num_threads);

    if (num_threads < 1 || num_threads > 32)
    {
        printf("Please enter a value between 1 and 32.\n");
        return 1;
    }

    omp_set_num_threads(num_threads);

    double start_time = get_time();

    #pragma omp parallel for reduction(+:sum)
    for (long i = 0; i < N; i++)
    {
        sum += (double)i * 0.000001;
    }

    double end_time = get_time();

    printf("Result = %.2f\n", sum);
    printf("Execution time = %.6f seconds\n",
           end_time - start_time);

    return 0;
}
```

Save:

```text
Ctrl + O
Enter
Ctrl + X
```

### Compile

Compiles the OpenMP performance program with OpenMP support.

```bash
gcc omp_perf.c -o omp_perf -fopenmp
```

### Run

Runs the OpenMP program for different thread counts.

```bash
./omp_perf
```

Enter:

```text
1
```

Repeat for:

```text
2
4
6
16
```

### OpenMP Results

| Threads | Execution Time |
|---:|---:|
| 1 | 1.409294 s |
| 2 | 0.715560 s |
| 4 | 0.360803 s |
| 6 | 0.241608 s |
| 16 | 0.140692 s |

---

# 18. Final Performance Comparison

Sequential baseline:

```text
1.353219 seconds
```

| Threads | Pthreads Time (s) | OpenMP Time (s) |
|---:|---:|---:|
| 1 | 1.348142 | 1.409294 |
| 2 | 0.680737 | 0.715560 |
| 4 | 0.358872 | 0.360803 |
| 6 | 0.241345 | 0.241608 |
| 16 | 0.144812 | 0.140692 |

### Observation

Execution time generally decreases as the number of threads increases.

The lowest measured execution time was:

```text
Pthreads → 0.144812 s
OpenMP   → 0.140692 s
```

with 16 threads.

---

# 19. Speedup

Formula:

```text
Speedup = Sequential Time / Parallel Time
```

| Threads | Pthreads Speedup | OpenMP Speedup |
|---:|---:|---:|
| 1 | 1.004x | 0.960x |
| 2 | 1.988x | 1.891x |
| 4 | 3.771x | 3.751x |
| 6 | 5.608x | 5.601x |
| 16 | 9.345x | 9.618x |

For OpenMP with 16 threads:

```text
Speedup = 1.353219 / 0.140692
        ≈ 9.62x
```

---

# 20. Efficiency

Formula:

```text
Efficiency = (Speedup / Number of Threads) × 100
```

| Threads | Pthreads Efficiency | OpenMP Efficiency |
|---:|---:|---:|
| 1 | 100.38% | 96.02% |
| 2 | 99.39% | 94.56% |
| 4 | 94.27% | 93.76% |
| 6 | 93.45% | 93.35% |
| 16 | 58.40% | 60.11% |

For 16-thread OpenMP:

```text
Efficiency = 9.62 / 16 × 100
           ≈ 60.1%
```

---

# 21. Why 16 Threads Do Not Give 16x Speedup

Ideal execution time:

```text
1.353219 / 16 ≈ 0.0846 seconds
```

Measured OpenMP time:

```text
0.140692 seconds
```

The speedup is not perfectly linear because of:

- Thread management
- Scheduling
- Synchronization
- Memory access
- Operating-system activity
- Non-parallel portions of the program

Therefore:

```text
More threads ≠ Perfect linear speedup
```

---

# 22. Pthreads vs OpenMP

| Concept | Pthreads | OpenMP |
|---|---|---|
| Thread creation | `pthread_create()` | `#pragma omp parallel` |
| Thread waiting | `pthread_join()` | Runtime handles completion |
| Work distribution | Programmer controlled | `parallel for` |
| Shared-data protection | Mutex | `critical` |
| Coordination | Join / synchronization | `barrier` |
| Combining results | Programmer managed | `reduction` |

---

# 23. Implemented Programs

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
| `omp_critical.c` | Synchronization using critical section |
| `omp_barrier.c` | Thread coordination |
| `omp_perf.c` | Measure OpenMP performance |

---

# 24. Important Terms

| Term | Meaning |
|---|---|
| **Thread** | A path of execution inside a program |
| **Main Thread** | The thread that starts execution of `main()` |
| **Multithreading** | Using multiple threads in one program |
| **Parallel Programming** | Dividing work among multiple execution units |
| **Work Distribution** | Dividing a task among threads |
| **Race Condition** | A problem caused by unsynchronized access to shared data |
| **Mutex** | A locking mechanism used to protect shared data |
| **Critical Section** | A section where simultaneous execution is restricted |
| **Barrier** | A synchronization point where threads wait for each other |
| **Speedup** | How much faster parallel execution is than sequential execution |
| **Efficiency** | Speedup obtained per thread |

---

# 25. Conclusion

The experiment demonstrates multithreaded programming using **Pthreads and OpenMP**.

Pthreads provides explicit control over thread creation, joining, work distribution, and mutex-based synchronization. OpenMP provides a higher-level programming model using parallel regions, work sharing, critical sections, barriers, and reductions.

The experiments demonstrate that race conditions can occur when multiple threads access shared data without synchronization. Pthread mutexes and OpenMP critical sections are used to solve this problem.

The performance results show that increasing the number of threads generally reduces execution time for the selected workload.

For 16 threads:

```text
Pthreads → 9.345x speedup
OpenMP   → 9.618x speedup
```

The corresponding efficiency was:

```text
Pthreads → 58.40%
OpenMP   → 60.11%
```

The results show that speedup is not perfectly linear because parallel execution introduces additional overhead.

Overall, the experiment demonstrates:

```text
Create Threads
      ↓
Manage Threads
      ↓
Divide Work
      ↓
Handle Shared Data
      ↓
Identify Race Conditions
      ↓
Synchronize Threads
      ↓
Coordinate Threads
      ↓
Measure Performance
      ↓
Calculate Speedup
      ↓
Calculate Efficiency
      ↓
Analyze Results
```
