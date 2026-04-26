# Assignment 3 - Complete Documentation

**Student Name**: Hind Mohammed
**Student ID**: 443052334  
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
### Entry 1 - April 21, 2026
**What I implemented**: Cloned the repository, set up the project locally, and updated my Student ID (443052334).
**Challenges encountered**: Encountered a Git 403 Permission Denied error when trying to push due to cached Windows credentials.
**How I solved it**: Updated the remote URL using `git remote set-url origin` to explicitly include my GitHub username.
**Testing approach**: Ran `git push` to verify successful connection to my repository.
**Time spent**: 45 minutes

### Entry 2 - April 22, 2026
**What I implemented**: Task 1: Added `ReentrantLock` to protect shared counter variables (`contextSwitchCount`, `completedProcessCount`, `totalWaitingTime`).
**Challenges encountered**: Ensuring that the lock is always released even if an error occurs.
**How I solved it**: Implemented the strict `try-finally` block pattern for every critical section.
**Testing approach**: Reviewed code to ensure every `lock.lock()` has a corresponding `lock.unlock()` in a `finally` block.
**Time spent**: 30 minutes

### Entry 3 - April 22, 2026
**What I implemented**: Task 2: Applied `ReentrantLock` to protect the `executionLog` ArrayList from concurrent modification.
**Challenges encountered**: Understanding why ArrayLists fail in multithreaded environments.
**How I solved it**: Wrapped the `executionLog.add()` method inside a `try-finally` block using the same `ReentrantLock`.
**Testing approach**: Checked for `ConcurrentModificationException` during program execution.
**Time spent**: 20 minutes

### Entry 4 - April 22, 2026
**What I implemented**: Task 3: Added a Binary `Semaphore` (`cpuSemaphore`) to control CPU access in `run()` and `runToCompletion()`.
**Challenges encountered**: Handling `InterruptedException` when acquiring the semaphore.
**How I solved it**: Used a `try-catch` block for `acquire()`, and placed `release()` inside a main `finally` block to prevent deadlocks.
**Testing approach**: Ran the simulation to verify that only one process executes its quantum at a time.
**Time spent**: 45 minutes

### Entry 5 - April 23, 2026
**What I implemented**: Task 4: Completed the technical documentation, ran final consistency checks, and recorded the demonstration video.
**Challenges encountered**: Terminal output had character encoding issues displaying emojis.
**How I solved it**: Compiled the code explicitly using `javac -encoding UTF-8 SchedulerSimulationSync.java`.
**Testing approach**: Ran the compiled program multiple times to ensure outputs matched expected logic.
**Time spent**: 60 minutes


---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

1. **Counter Variables Increment:** The operation `completedProcessCount++` is a read-modify-write operation. If two threads read the variable at the exact same time, both will increment the same original value and write it back, resulting in a lost increment (e.g., total processes might show 14 instead of 15).
2. **ArrayList Modification:** The `executionLog.add(message)` method modifies the internal structure of an `ArrayList`, which is not thread-safe. If concurrent access happens, the list might throw a `ConcurrentModificationException`, or log entries might overwrite each other, leading to data loss.

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

A `ReentrantLock` provides "Mutual Exclusion" (ownership); the thread that locks it MUST be the one to unlock it, making it ideal for protecting data. A `Semaphore` is a signaling mechanism based on permits; it doesn't have an "owner" and can restrict access to a limited pool of resources. 
In my code, I used a `ReentrantLock` to protect the shared data (counters and the execution log) so only one thread updates them at a time. I used a `Semaphore` initialized to 1 (binary semaphore) to represent the single-core CPU, acting as a gatekeeper to ensure only one process executes simultaneously.

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

A deadlock is a situation where two or more threads are blocked forever, waiting for each other to release resources. 
Two prevention techniques are: 1) **Lock Ordering:** Always acquiring multiple locks in the exact same sequence. 2) **Guaranteed Release:** Ensuring locks are released even if a critical error happens. 
In my code, I prevented deadlocks by strictly using the `try-finally` construct. By placing `lock.unlock()` and `cpuSemaphore.release()` inside `finally` blocks, I guaranteed that resources are surrendered even if an `InterruptedException` or unexpected error occurs during execution.

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

I used a **coarse-grained** approach (ONE single `ReentrantLock` for all three counters and the execution log). I made this choice because the critical sections involved are extremely short and fast (simple integer increments and list additions). 
**Trade-offs:** A fine-grained approach (separate locks for each variable) provides better concurrency because a thread updating context switches wouldn't block a thread updating waiting time. However, it increases code complexity, memory usage, and the risk of deadlocks. While fine-grained is theoretically better for independent counters, the coarse-grained approach was highly effective and efficient for this specific simulation due to the negligible execution time of the operations.
---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: `contextSwitchCount`, `completedProcessCount`, `totalWaitingTime`.
**Why they need protection**: Multiple process threads attempt to read and modify these values concurrently at the end of their quantums, risking lost updates.
**Synchronization mechanism used**: `ReentrantLock`
**Code snippet**:
```java
public static void incrementCompletedProcess() {
    lock.lock();
    try {
        completedProcessCount++;
    } finally {
        lock.unlock();
    }
}

Justification: A lock ensures mutual exclusion, guaranteeing atomic updates to the integers.

---

### Critical Section #2: Execution Log

**What resource**: List<String> executionLog

**Why it needs protection**: ArrayList is not thread-safe. Concurrent additions can corrupt the internal array or throw exceptions.

**Synchronization mechanism used**: ReentrantLock

**Code snippet**:
```java
public static void logExecution(String message) {
    lock.lock();
    try {
        executionLog.add(message);
    } finally {
        lock.unlock();
    }
}
```

**Justification**: Prevents ConcurrentModificationException and ensures all logs are appended safely.

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: To limit the number of threads (processes) running on the simulated CPU at the exact same time.

**Number of permits and why**: 1 permit. Because we are simulating a single-core processor that can only handle one process quantum at a time.

**Where implemented**: Inside the run() and runToCompletion() methods of the Process class.

**Code snippet**:
```java
try {
    SharedResources.cpuSemaphore.acquire();
} catch (InterruptedException e) {
    // Exception handling
    return;
}
try {
    // Process execution logic...
} finally {
    SharedResources.cpuSemaphore.release();
}
```

**Effect on program behavior**: It completely sequentializes the execution outputs in the console. Instead of multiple progress bars printing over each other, processes neatly take turns.

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results.

**Testing procedure**: 
```bash
javac -encoding UTF-8 SchedulerSimulationSync.java
java SchedulerSimulationSync
```

**Results**: 
The program successfully finished execution every time with 15 completed processes and exactly 58 execution log entries for Student ID 443052334.

**Why synchronization is necessary**: 
 Without it, the final counter of completed processes might have been less than 15 due to race conditions.
**Conclusion**: 
he implementation is robust and thread-safe.

---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**: Observed the console for any Java stack trace errors during the heavy logging phases.

**Results**: Zero exceptions occurred

**What this proves**: The ReentrantLock correctly serialized access to the ArrayList.

---

### Test 3: Correctness Verification
**What I tested**:Verifying correct final values based on the specific random seed generated by my Student ID (443052334).

**Expected values**: 15 processes created and exactly 15 processes completed.

**Actual values**: 15 completed processes, 29 total context switches.

**Analysis**: The counters are 100% accurate, proving no increments were lost.

---

### Test 4: Different Scenarios
**Scenario tested**: Interrupting the terminal while threads were sleeping.

**Purpose**: To verify that the finally blocks actually release locks and permits during exceptions.

**Results**: Program cleanly handled the InterruptedException without hanging the console indefinitely.

**What I learned**:Strict use of try-finally is not just theoretical; it physically prevents the application from freezing during unexpected behaviors. 

---

## Part 5: Reflection and Learning

### What I learned about synchronization:

I learned that writing multithreaded code introduces severe unpredictability (race conditions) that are hard to debug because they don't always crash the program—they just produce wrong data silently. I realized the profound difference between a Mutex Lock (ownership for data protection) and a Semaphore (permits for capacity limits). Most importantly, I learned the critical habit of always using try-finally blocks; seeing the logic behind guaranteeing resource release taught me a lot about writing defensive, production-ready code.

---

### Real-world applications:

Example 1: Banking Systems: When multiple users (or automated systems) attempt to withdraw and deposit money into the same bank account simultaneously, locks are required to prevent balance miscalculation.
Example 2: Online Ticket Booking: Using semaphores or locks to ensure that the last available seat on an airplane or at a concert isn't sold to two different customers checking out at the exact same millisecond.

---

### How I would explain synchronization to others:

Imagine a lock is like the key to a single-occupancy bathroom. Only one person (thread) can have the key at a time, protecting their privacy (data). A semaphore, on the other hand, is like a bouncer at a club. The bouncer has 5 wristbands (permits). Up to 5 people can enter, but the 6th person must wait in line until someone leaves and hands their wristband back to the bouncer.

---

## Part 6: GitHub Repository Information

 Repository URL: [Paste your GitHub Repo Link Here]

Number of commits: 4

Commit messages:

Set my student ID

1- Completed Task 1: Added ReentrantLock to protect shared variables

2- Task 2 completed: Added strict protection for execution log

3- Task 3 completed: Successfully implemented Semaphore for CPU scheduling

---

## Summary

**Total time spent on assignment**: Approx. 3-4 hours

Key takeaways:

Always protect shared mutable state in Java.

try-finally is mandatory for safe lock management.

Semaphores are excellent for throttling execution flow.

Most challenging aspect: Ensuring the correct placement of brackets and scopes so the finally block successfully catches the specific try block without breaking the logic.
What I'm most proud of: Achieving a flawless run with zero race conditions and understanding exactly why the terminal output was organized.
---

**End of Documentation**
