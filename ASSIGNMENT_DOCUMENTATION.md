# Assignment 3 - Complete Documentation

**Student Name**: Rawan Yousef Saleh 
**Student ID**: 444052889
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

### Entry 1 - 2026-04-25, 10:00 AM
**What I implemented**: 
 Forked the repository and set up the project in VS Code. Changed student ID in SchedulerSimulationSync.java.
**Challenges encountered**: 
Git was not installed on my computer, couldn't clone the repository.
**How I solved it**: 
Downloaded and installed Git from git-scm.com, then restarted VS Code.


**Testing approach**: 
Confirmed git --version worked in terminal.
**Time spent**: 
 1 hour
---

### Entry 2 - 2026-04-26, 2:00 PM
**What I implemented**: Added ReentrantLock for counter variables (contextSwitchCount, completedProcessCount, totalWaitingTime).

**Challenges encountered**: 
Understanding where exactly to place the lock() and unlock() calls.
**How I solved it**: 
Wrapped each counter modification in try-finally blocks to ensure unlock always happens.
**Testing approach**: 
Compiled the code and checked for compilation errors.
**Time spent**: 
 1.5 hours
---

### Entry 3 -2026-04-26, 5:00 PM
**What I implemented**: 
Added ReentrantLock for executionLog ArrayList and Semaphore for CPU access.
**Challenges encountered**: 
 The locks needed to be static to be shared across all threads.
**How I solved it**: 
Changed private final to private static final for all locks and semaphore.
**Testing approach**: 
 Ran the program multiple times to verify no ConcurrentModificationException.


**Time spent**: 
1 hour
---

### Entry 4 - 2026-04-27, 7:00 PM
**What I implemented**: 
Fixed bracket placement errors and completed all synchronization tasks.
**Challenges encountered**: 
Compilation errors due to incorrect curly brace placement.
**How I solved it**: 
Restructured the code with proper indentation and matching braces.
**Testing approach**: 
Ran javac and java commands successfully.
**Time spent**: 
 1 hour
---

### Entry 5 - [Date, Time]
**What I implemented**: 

**Challenges encountered**: 

**How I solved it**: 

**Testing approach**: 

**Time spent**: 

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

Counter variables (contextSwitchCount, completedProcessCount, totalWaitingTime) : These are shared resources updated by multiple threads. Without synchronization, when two threads try to increment contextSwitchCount simultaneously, one increment could be lost because read-modify-write operations are not atomic. For example, if both threads read the same value (e.g., 5), increment to 6, and write back, the result would be 6 instead of 7.

ArrayList (executionLog) : ArrayList is not thread-safe. When multiple threads call executionLog.add() simultaneously, it can cause ConcurrentModificationException or internal corruption of the ArrayList data structure. One thread might be resizing the internal array while another is adding an element.
---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

rantLock provides mutual exclusion - only one thread can hold the lock at a time. I used ReentrantLock for protecting shared data (counters and ArrayList) because these resources need exclusive access for reading and writing.

Semaphore allows controlling how many threads can access a resource simultaneously using permits. I used a binary Semaphore with 1 permit for CPU access to ensure only one process executes on the CPU at a time.

The key difference: ReentrantLock is for mutual exclusion (protecting data), while Semaphore is for limiting concurrent access (controlling resources). I used ReentrantLock for counterLock and logLock, and Semaphore for cpuSemaphore.
---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

occurs when two or more threads are blocked forever, each waiting for a resource held by another.

Two prevention techniques I used:

try-finally blocks: Always releasing locks in finally block ensures locks are released even if exceptions occur. Example: lock() in try, unlock() in finally.

Single lock per resource: Rather than acquiring multiple locks, I used separate locks for independent resources, eliminating circular wait conditions.

I also used consistent lock ordering (always acquire counterLock before logLock when needed) and the Semaphore acquire/release pattern with try-finally.
---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

I used ONE lock (counterLock) for all three counters (coarse-grained locking). I made this choice because the three counters are updated sequentially in the code and there's no performance benefit to separate locks.

Trade-offs:

Coarse-grained (one lock): Simpler code, less overhead, but lower concurrency if counters were updated frequently.

Fine-grained (separate locks): Better concurrency if counters are independent and frequently accessed simultaneously.

Since these three counters are updated infrequently (once per context switch or process completion) and the critical sections are very short, the concurrency benefit of fine-grained locking would be negligible. Coarse-grained provides simpler, more maintainable code with no real performance penalty in this scenario.



---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: contextSwitchCount, completedProcessCount, totalWaitingTime

**Why they need protection**: 
 Multiple threads update these simultaneously, causing lost updates.
**Synchronization mechanism used**: 
ReentrantLock (counterLock)
**Code snippet**:private static final ReentrantLock counterLock = new ReentrantLock();

public static void incrementContextSwitch() {
    counterLock.lock();
    try {
        contextSwitchCount++;
    } finally {
        counterLock.unlock();
    }
}

**Justification**: 
try-finally ensures lock is always released, preventing deadlocks.


---

### Critical Section #2: Execution Log

**What resource**: 
 ArrayList<String> executionLog
**Why it needs protection**: 
ArrayList is not thread-safe; concurrent adds cause corruption
**Synchronization mechanism used**: 
 ReentrantLock (logLock)
**Code snippet**:private static final ReentrantLock logLock = new ReentrantLock();

public static void logExecution(String message) {
    logLock.lock();
    try {
        executionLog.add(message);
    } finally {
        logLock.unlock();
    }
}

**Justification**: 
Separate lock from counterLock for better concurrency.
---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**:  Limit concurrent CPU access to one process at a time.

**Number of permits and why**: 
1 permit (binary semaphore) - only one process runs at a time.
**Where implemented**: 
In Process.run() before execution and in finally block.
**Code snippet**:
private static final Semaphore cpuSemaphore = new Semaphore(1);

public void run() {
    try {
        cpuSemaphore.acquire();
        // execute quantum
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        cpuSemaphore.release();
    }
}

**Effect on program behavior**: 

 Ensures only one process executes at a time, preventing CPU overuse.

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**:  Run program 5 times to check consistent results.
**Testing procedure**: 
javac SchedulerSimulationSync.java
java SchedulerSimulationSync
# Repeated 5 times

**Results**: 
 Each run produced Total Context Switches = number of quantum executions, Total Completed Processes = number of processes (10-20), consistent with Round Robin scheduling.
**Why synchronization is necessary**:  Without synchronization, race conditions could cause incorrect counter values, ConcurrentModificationException, or lost updates.

**Conclusion**: 

---Synchronization ensures correct and consistent results.



### Test 2: Exception Testing
**What I tested**:ArrayList concurrent modification.

**Testing procedure**: 
Ran program multiple times with synchronized logLock.
**Results**: 
No ConcurrentModificationException occurred in any run.
**What this proves**: 

--- logLock successfully protects the shared ArrayList.

### Test 3: Correctness Verification
**What I tested**:  Verify final statistics.

**Expected values**: 
CompletedProcessCount should equal number of processes created.
**Actual values**: 
completedProcessCount matched numProcesses in all runs.
**Analysis**: 
All processes completed successfully with proper synchronization.
---

### Test 4: Different Scenarios
**Scenario tested**: [e.g., different time quantum, more processes, etc.]

**Purpose**: 

**Results**: 

**What I learned**: 

---

## Part 5: Reflection and Learning

### What I learned about synchronization:

i learned that race conditions occur when multiple threads access shared resources without coordination. The critical section must be protected using locks or semaphores. I learned the importance of always releasing locks in finally blocks to prevent deadlocks. I also learned the difference between ReentrantLock (mutual exclusion) and Semaphore (limiting concurrent access). Understanding lock granularity helped me make better design decisions. Synchronization adds overhead but is necessary for correctness in multithreaded programs. Testing multiple times is essential to verify synchronization works correctly.
---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: 
 Banking systems - when multiple transactions access the same account balance simultaneously, synchronization prevents incorrect balance updates.
**Example 2**:  Airline booking systems - preventing double-booking of the same seat when multiple users try to book simultaneously.



---

### How I would explain synchronization to others:

[Explain to someone who just finished Assignment 1 - use simple terms and analogies]

---

## Part 6: GitHub Repository Information

**Repository URL**: 

**Number of commits**: 3

**Commit messages**: 
1. lock /unlock
2. samephare
3. change id
4. 

---

## Summary

**Total time spent on assignment**: 
approximately 6 hours
**Key takeaways**: 
Synchronization is essential for thread-safe programs

Always use try-finally blocks when working with locks

Test multithreaded programs multiple times to verify correctness

**Most challenging aspect**: 
 Understanding lock granularity and fixing bracket placement errors.
**What I'm most proud of**: 
: The program runs consistently without race conditions or exceptions.
---

**End of Documentation**
