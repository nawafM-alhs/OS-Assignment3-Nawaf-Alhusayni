# Assignment 3 - Complete Documentation

**Student Name**: [Your Full Name]  
**Student ID**: [Your ID]  
**Date Submitted**: [Submission Date]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [Paste your personal Gmail Google Drive link here]

**Video filename**: `[YourStudentID]_Assignment3_Synchronization.mp4`

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [April 30, 2026, 6:30 PM]
**What I implemented**: 
I forked the starter repository, cloned it to my local machine, opened it in Visual Studio Code, and reviewed the main file `SchedulerSimulationSync.java`. I also updated the `studentID` variable from the default value to my actual student ID.

**Challenges encountered**: 
The main challenge was understanding how the scheduler simulation was organized and identifying which variables were shared between multiple threads.

**How I solved it**: 
I traced the code from the `main` method to the `Process` class and then to the `SharedResources` class. This helped me identify that `contextSwitchCount`, `completedProcessCount`, `totalWaitingTime`, and `executionLog` were shared resources.

**Testing approach**: 
I ran the program once before adding synchronization to observe the output structure, the number of processes, the time quantum, and the final statistics.

**Time spent**: 
1 hour

---

### Entry 2 - [April 30, 2026, 8:30 PM]
**What I implemented**: 
I added the required synchronization imports at the top of the file: `Semaphore` and `ReentrantLock`. I then added a `counterLock` inside the `SharedResources` class to protect the shared counter variables.

**Challenges encountered**: 
The challenge was deciding where the lock should be declared and how to apply it without changing the original structure of the code.

**How I solved it**: 
I declared the lock as a `public static final ReentrantLock` inside `SharedResources`, because the shared variables are also static and accessed through static methods.

**Testing approach**: 
I checked that the program still compiled after adding the imports and the new lock object.

**Time spent**: 
45 minutes

---

### Entry 3 - [May 1, 2026, 3:00 PM]
**What I implemented**: 
I protected the three shared counter methods: `incrementContextSwitch`, `incrementCompletedProcess`, and `addWaitingTime`. Each update was placed inside a lock and unlock structure using a `try-finally` block.

**Challenges encountered**: 
The main challenge was making sure the lock would always be released, even if an exception occurred during execution.

**How I solved it**: 
I used `counterLock.lock()` before the critical section and placed `counterLock.unlock()` inside the `finally` block.

**Testing approach**: 
I ran the program and checked that the final statistics were printed successfully and that the completed process count matched the number of created processes.

**Time spent**: 
1 hour

---

### Entry 4 - [ May 1, 2026, 6:00 PM]
**What I implemented**: 
I added a separate `logLock` to protect the shared `executionLog` ArrayList. I updated the `logExecution` method so that adding messages to the log is done inside a protected critical section.

**Challenges encountered**: 
The challenge was recognizing that `ArrayList` is not thread-safe and can be unsafe when multiple threads add entries concurrently.

**How I solved it**: 
I used a separate `ReentrantLock` for the execution log instead of using the same counter lock. This keeps the counter updates and log updates separated.

**Testing approach**: 
I ran the program and verified that the execution log summary appeared at the end without any `ConcurrentModificationException`.

**Time spent**: 
1 hour

---

### Entry 5 - [ May 1, 2026, 7:30 PM]
**What I implemented**: 
I added a binary semaphore named `cpuSemaphore` with one permit. I acquired the semaphore before process execution and released it inside a `finally` block. I also tested the final version and completed the documentation.

**Challenges encountered**: 
The challenge was placing the semaphore correctly so that only the CPU execution section is controlled and the permit is always released.

**How I solved it**: 
I used `SharedResources.cpuSemaphore.acquire()` before the main execution logic and `SharedResources.cpuSemaphore.release()` in the `finally` block. I also handled `InterruptedException` by interrupting the current thread and returning safely.

**Testing approach**: 
I ran the program multiple times. The final output showed that all processes completed, the completed process count matched the generated process count, and the execution log was updated successfully.

**Time spent**: 
1.5 hours

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:
The first race condition is related to the shared counter variables, especially `contextSwitchCount` and `completedProcessCount`. These variables are updated by multiple process threads using increment operations such as `contextSwitchCount++`. This operation is not atomic because it includes reading the old value, modifying it, and writing the new value back. If two threads perform the update at the same time, one update may be lost, and the final count may be incorrect.

The second race condition is related to the shared `executionLog` ArrayList. The original code allows multiple threads to add messages to the same ArrayList using `executionLog.add(message)`. Since ArrayList is not thread-safe, concurrent access can cause inconsistent log entries or runtime errors such as `ConcurrentModificationException`. This could make the execution log inaccurate or cause the program to fail during execution.

---


[Your answer here - 4-6 sentences with code examples]

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:
`ReentrantLock` is used to provide mutual exclusion for a critical section. It allows only one thread at a time to enter a protected block of code, and it is useful when protecting shared variables from race conditions. In my code, I used `ReentrantLock` to protect the shared counters and the shared execution logg.

A `Semaphore` controls access to a limited number of permits. It can allow one or more threads to access a resource depending on the number of available permits. In my code, I used a binary semaphore with one permit to control CPU access. This means only one process thread can execute its CPU quantum at a time, which matches the idea of one CPU being assigned to one process at a time in this simulation.

[Your answer here - explain your implementation choices]

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:
Deadlock occurs when two or more threads are blocked forever because each thread is waiting for a resource held by another thread. One prevention technique is to always release locks and semaphores in a `finally` block, so the resource is released even if an exception occurs. I used this technique by placing `counterLock.unlock()`, `logLock.unlock()`, and `cpuSemaphore.release()` inside `finally` blocks.

A second prevention technique is to avoid holding multiple locks unnecessarily or to use a consistent lock order when multiple locks are required. In my implementation, the counter lock and log lock are used in separate methods, and the code does not hold both locks at the same time. This reduces the chance of circular waiting and helps prevent deadlock.

[Your answer here - reference try-finally blocks, lock ordering, etc.]

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:
For Task 1, I used one `ReentrantLock` called `counterLock` to protect the three shared counters: `contextSwitchCount`, `completedProcessCount`, and `totalWaitingTime`. This is a coarse-grained locking approach. I chose this design because the counter operations are short, simple, and easy to protect using one shared lock. This also makes the code easier to read and reduces the chance of forgetting to protect one of the counters.

The trade-off is that coarse-grained locking is simpler but may reduce concurrency because only one counter update can happen at a time. Fine-grained locking, where each counter has its own lock, can provide better concurrency because independent counters can be updated at the same time. Since the three counters are independent, fine-grained locking would theoretically provide better concurrency. However, for this assignment, the operations are very small, so one counter lock is acceptable and keeps the implementation clear and safe.


[Your answer here - explain coarse-grained vs fine-grained locking, independence of counters, concurrency implications. Show understanding of when to use each approach. 5-8 sentences expected.]

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: 
`contextSwitchCount`, `completedProcessCount`, and `totalWaitingTime`.
**Why they need protection**: 
These variables are shared by multiple process threads. Increment and addition operations are not atomic, so concurrent updates can lead to lost updates and incorrect final statistics.

**Synchronization mechanism used**: 
I used `ReentrantLock` named `counterLock`.

**Code snippet**:
```java
// Lock used to protect shared counter variables
public static final ReentrantLock counterLock = new ReentrantLock();

**Justification**: 
The lock makes each update execute as a critical section. This prevents two threads from modifying the same shared counter at the same time and ensures that the final statistics are accurate.
---

### Critical Section #2: Execution Log

**What resource**: 
The shared executionLog ArrayList.
**Why it needs protection**: 
ArrayList is not thread-safe. If multiple threads add log entries at the same time, the log may become inconsistent or the program may throw a ConcurrentModificationException.
**Synchronization mechanism used**: 
I used a separate ReentrantLock named logLock.
**Code snippet**:
```java
// Lock used to protect the shared execution log
public static final ReentrantLock logLock = new ReentrantLock();

public static void logExecution(String message) {
    logLock.lock();
    try {
        executionLog.add(message);
    } finally {
        logLock.unlock();
    }
}
**Justification**: 
Using a separate lock for the log keeps log updates independent from counter updates. This protects the ArrayList while keeping the design organized and easy to understand.
---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: 
The semaphore controls access to the simulated CPU execution section.
**Number of permits and why**: 
I used one permit because the assignment requires a binary semaphore. One permit means only one process can execute on the CPU at a time.
**Where implemented**: 
The semaphore is declared inside SharedResources and used inside the run method before the process starts executing its quantum. It is released inside the finally block.
**Code snippet**:
```java
// Paste your implementation here
```

**Effect on program behavior**: 

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: 
```bash
# javac SchedulerSimulationSync.java
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSyncCommands used (run the program at least 5 times)
```

**Results**: 
The program completed successfully each time. In the provided run, the program generated 16 processes and the final output showed Total Completed Processes: 16. This means all generated processes were completed. The output also showed Total Context Switches: 35, Total Waiting Time: 1227080ms, and Total log entries: 70.
**Why synchronization is necessary**: 
Synchronization is necessary because multiple process threads access shared variables. Without locking, the counters may lose updates because increment operations are not atomic. Without protecting the execution log, multiple threads could update the ArrayList at the same time, which could lead to inconsistent data or ConcurrentModificationException.
**Conclusion**: 
The synchronized version produced correct final statistics and completed all processes successfully.
---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException
I tested whether the program runs without throwing ConcurrentModificationException or synchronization-related errors.
**Testing procedure**: 
I ran the program several times after protecting the executionLog ArrayList using logLock.
**Results**: 
The program completed successfully and printed the execution log summary. In the provided output, the final line showed Total log entries: 70.
**What this proves**: 
This proves that the shared execution log was updated safely. Protecting executionLog.add(message) with logLock prevents unsafe concurrent modifications.
---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)
I verified that the final output values are logically correct based on the simulation.v
**Expected values**: 
The total completed process count should match the number of generated processes. Since the output showed 16 processes at the beginning, the expected completed process count is 16.
**Actual values**: 
The output showed:

Processes: 16
Total Completed Processes: 16
Total Context Switches: 35
Total Waiting Time: 1227080ms
Average Waiting Time: 76692ms
Total log entries: 70
**Analysis**: 
The completed process count matches the number of created processes, so the scheduler completed all processes. The context switch count is greater than the number of processes because several processes required more than one time quantum and were returned to the ready queue. The total log entries are reasonable because each process execution and yield or completion is recorded.
---

### Test 4: Different Scenarios
**Scenario tested**: [e.g., different time quantum, more processes, etc.]
I tested the program with the deterministic random values generated from the student ID seed. This produced different burst times, priorities, and remaining times for multiple processes.
**Purpose**: 
The purpose was to verify that the synchronization logic works with many processes and different CPU burst times.
**Results**: 
The program handled 16 processes successfully. Some processes completed in one quantum, while others required multiple rounds. The scheduler continued until the ready queue became empty and then printed the final statistics.
**What I learned**: 
I learned that synchronization must protect shared data even when the program appears to run correctly. Race conditions may not appear every time, but they can still exist when shared resources are accessed by multiple threads.
---

## Part 5: Reflection and Learning

### What I learned about synchronization:
I learned that synchronization is essential when multiple threads share the same data. Even simple operations like counter++ can be unsafe because they are not atomic. I also learned that ReentrantLock is useful for protecting critical sections such as shared counters and shared lists. The assignment also showed me that Semaphore is useful when controlling access to a limited resource such as a CPU. Using try-finally is important because it guarantees that locks and semaphores are released even when an error happens. I also learned that correct multithreaded programs are not only about getting the right output once, but about producing safe and consistent behavior across repeated executions. This assignment connected the operating system concepts of processes, threads, scheduling, and synchronization in one practical program.
[6-8 sentences about key concepts, challenges, insights]

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: 
Banking systems need synchronization when multiple transactions access the same account balance. Without synchronization, two withdrawals or deposits could happen at the same time and produce an incorrect final balance.
**Example 2**: 
Online ticket booking systems need synchronization when many users try to reserve the same seat. Without synchronization, the same seat could be sold to more than one customer.
---

### How I would explain synchronization to others:
Synchronization is like allowing only one person at a time to write on a shared whiteboard. If many people write at the same time, the result may become incorrect or unreadable. In a program, shared variables are like that whiteboard. Locks make sure only one thread can update the shared data at a time. Semaphores are similar, but they control how many threads can access a resource based on the number of available permits. In this assignment, locks protected the shared counters and execution log, while the semaphore controlled access to the simulated CPU.
[Explain to someone who just finished Assignment 1 - use simple terms and analogies]

---

## Part 6: GitHub Repository Information

**Repository URL**: 
https://github.com/[nawafM-alhs]/OS-Assignment3-[Nawaf]-[Alhusayni]
**Number of commits**: 
6
**Commit messages**: 
1. Set my student ID
2. Add synchronization imports
3. Protect shared counters with ReentrantLock
4. 
Protect execution log with ReentrantLock
---

## Summary

**Total time spent on assignment**: 
4.5 hours
**Key takeaways**: 
1. Shared resources must be protected when accessed by multiple threads.
2. ReentrantLock is useful for mutual exclusion in critical sections.
3. A binary semaphore can control exclusive access to a simulated CPU.

**Most challenging aspect**: 
The most challenging aspect was identifying all shared resources and deciding which synchronization mechanism should protect each one.
**What I'm most proud of**: 
I am most proud of connecting the theory of process synchronization with a working Java scheduler simulation and verifying the results through repeated testing.
---

**End of Documentation**
