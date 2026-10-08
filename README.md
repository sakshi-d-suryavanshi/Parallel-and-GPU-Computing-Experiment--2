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

This experiment uses two parallel programming technologies:

- **Pthreads** — explicit thread creation and management
- **OpenMP** — higher-level parallel programming using compiler directives

The final part compares sequential, Pthreads, and OpenMP execution times.

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

**Why:** This command opens the Ubuntu Linux environment through WSL.

```bash
wsl
```

---

## 4.2 Create the Lab Directory

**Why:** This command creates the directory where all experiment programs will be stored.

```bash
mkdir -p ~/parallel_lab
```

**Why:** This command moves into the experiment directory.

```bash
cd ~/parallel_lab
```

**Why:** This command displays the current working directory.

```bash
pwd
```

Expected:

```text
/home/<username>/parallel_lab
```

---

## 4.3 Check GCC

**Why:** This command checks whether the GCC C compiler is installed.

```bash
gcc --version
```

---

## 4.4 Check OpenMP Support

**Why:** This command checks GCC with the OpenMP compilation option.

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

To understand:

- Thread creation
- Thread execution
- Waiting for a thread using `pthread_join()`

## 5.1 Create the Program

**Why:** This command opens Nano to create the first Pthreads program.

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

## 5.2 Compile

**Why:** This command compiles the program and links the Pthreads library.

```bash
gcc thread1.c -o thread1 -pthread
```

## 5.3 Run

**Why:** This command executes the compiled program.

```bash
./thread1
```

Expected output:

```text
Hello from the thread!
Main thread finished.
```

### Key Functions

`pthread_create()` creates an additional thread.

```c
pthread_create(&thread, NULL, thread_function, NULL);
```

`pthread_join()` makes the main thread wait until the created thread finishes.

```c
pthread_join(thread, NULL);
```

Therefore:

```text
pthread_create() → CREATE
pthread_join()   → WAIT
```

---

# 6. Create Multiple Threads

## 6.1 Create the Program

**Why:** This command opens Nano to create a program that creates four Pthreads.

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

## 6.2 Compile

**Why:** This command compiles the multiple-thread program using Pthreads.

```bash
gcc thread2.c -o thread2 -pthread
```

## 6.3 Run

**Why:** This command executes the program and creates four additional threads.

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

The order may be different because the operating system controls thread scheduling.

---

# 7. Divide Work Among Threads

The array used is:

```text
10 20 30 40 50 60 70 80
```

The work is divided among four threads.

```text
Thread 1 → 10 + 20 = 30
Thread 2 → 30 + 40 = 70
Thread 3 → 50 + 60 = 110
Thread 4 → 70 + 80 = 150

Total = 360
```

## 7.1 Create the Program

**Why:** This command opens Nano to create the program that divides array-sum work among threads.

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

## 7.2 Compile

**Why:** This command compiles the work-distribution program using Pthreads.

```bash
gcc thread_sum.c -o thread_sum -pthread
```

## 7.3 Run

**Why:** This command executes the program and calculates the partial and total sums.

```bash
./thread_sum
```

Expected values:

```text
Thread 1 → 30
Thread 
