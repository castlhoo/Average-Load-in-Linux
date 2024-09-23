
# 🖥️ Average Load in Linux
Today, I am gonna analyze Average Load. The reason why I analyze Average Load is that we can evaluate the system overall competency and status. Also we can check about the system's bottlenecks or performance issues.

Additionally, When operating a system or program, it's vital for users not to shut down during a process. An unstable operating system can lead to data loss and reduced user trust. To avoid this, we often use tools like `top` or `uptime` when our system feels slow. But do we really understand what these mean? 

🤔 Let's figure out the concepts and explore better ways to manage system performance.

![image](https://github.com/user-attachments/assets/5c0d9004-c772-4ec5-b3c7-65fcdcdfcb4b)

---

### 🔍 Understanding the Output of `uptime`
![image](https://github.com/user-attachments/assets/5ac58a56-6b14-43fd-a950-c72ae227759e)

| Time            | Description                          |
|-----------------|--------------------------------------|
| **12:30:55**    | Current system time                  |
| **up 3:35**     | System uptime (3 hours, 35 minutes)  |
| **3 users**     | Number of logged-in users            |
| **Load average**| Average system load over 1, 5, and 15 minutes |

- **Average load**: Represents the average number of processes in the runnable or uninterruptible state.
- **Runnable Process**: A process that is either using the CPU or waiting for CPU time.
![image](https://github.com/user-attachments/assets/d7892fbd-4112-4799-8abe-b3aabb67852b)
- **Uninterruptible Process**: A critical kernel process that cannot be interrupted, ensuring system protection.

---

### 🛠️ How to Check CPU Count

To find out how many CPUs your system has, use any of the following commands:

```bash
top
cat /proc/cpuinfo
nproc
lscpu
```

---

### ⚖️ Average Load vs. CPU Usage

- **Average Load**: Refers to the number of processes in the runnable and uninterruptible states. This includes processes using the CPU, waiting for the CPU, and those waiting for I/O.
- **CPU Usage**: Represents how busy the CPU is during a given time period and doesn't necessarily correlate directly to the average load.

---

## 🧪 Practical Example: Understanding Average Load

### Prerequisites
![image](https://github.com/user-attachments/assets/544a55a4-2649-4100-8f8d-7ea44039c7bf)

Install the necessary tools:
```bash
apt install stress sysstat
```

- **mpstat**: A tool for analyzing multi-core CPU performance, providing real-time metrics.
- **pidstat**: A tool for process performance, showing CPU, memory, I/O, and context switches for each process.

---

### 1️⃣ CPU-Intensive Process

We will simulate 100% CPU usage with the `stress` command.

#### Step 1: Run the stress command in the first terminal:
![image](https://github.com/user-attachments/assets/3de6de17-7043-4c7a-9767-d5d443b9b678)
```bash
stress --cpu 1 --timeout 600
```
> Set one CPU-bound task and the stress will run for 600 seconds.

#### Step 2: Check the load with `uptime` in a second terminal:
![image](https://github.com/user-attachments/assets/fbe9f0ee-0313-4c16-a1ad-e5deb4fc15c8)
```bash
uptime
```
> The load increases to 1.00 after one minute.

#### Step 3: Use `mpstat` to observe CPU usage:
![image](https://github.com/user-attachments/assets/a67e8cc5-c834-4987-a341-eac71955d343)
```bash
mpstat -P ALL 5 1
```
> One CPU is at 100%, indicating that the increased load is due to high CPU usage.
> 
> We can see status of CPU from 'mpstat'
> 
> It collects statistics of all CPU cores in the system once every 5 seconds

#### Step 4: Find the culprit process:
![image](https://github.com/user-attachments/assets/7addc2c0-e274-4fb5-aefd-53352ea6dd5e)
```bash
pidstat -u 5 1
```
> The process with PID 10867 is causing the 100% CPU usage.
>
> It monitors and reports CPU usage, it collects statistics every 5 seconds and generates an only one report that pidstat generates.

---

### 2️⃣ I/O-Intensive Process

Simulate I/O pressure using `sync`.

#### Step 1: Run `stress` to simulate I/O:
![image](https://github.com/user-attachments/assets/86bcf64d-28d3-4539-a32d-c60fd90cc47d)
```bash
stress --io 1 --timeout 600
```

#### Step 2: Check load with `uptime`:
![image](https://github.com/user-attachments/assets/403de7a3-a537-4e4c-92a8-48e484a1b2c2)
```bash
uptime
```

#### Step 3: Use `mpstat` to check CPU and I/O:
![image](https://github.com/user-attachments/assets/9963d819-74fd-41ef-9181-e269cd72f0de)
```bash
mpstat -P ALL 5 1
```
> One CPU has an I/O wait time of 66.76%, showing the load is due to I/O activity.

#### Step 4: Identify the process ID:
![image](https://github.com/user-attachments/assets/b8255d51-6c65-4ff8-9fa1-a084647d8433)
```bash
pidstat -u 5 1
```
> The stress process is causing the high I/O wait time.

---

### 3️⃣ Overloading the CPU with Processes

What happens when the number of running processes exceeds CPU capacity?

#### Step 1: Simulate 8 processes on a 2-CPU system:
![image](https://github.com/user-attachments/assets/8f759aed-93c1-41cb-b4a1-9eb7f8ff1d0e)
```bash
stress --cpu 8 --timeout 600
```

#### Step 2: Check process stats with `pidstat`:
![image](https://github.com/user-attachments/assets/eb200442-62e0-4470-9351-c1ff10749c0c)
```bash
pidstat 1
```
> The 8 processes are contending for 2 CPUs, with each process waiting for the CPU about 75% of the time.

---

### ❓ Simple Question ❓
See below pictures and explain how factors mainly affect to CPU stress. 
![image](https://github.com/user-attachments/assets/880bbe2c-0b5e-43ed-ba64-cbb734be43d9)


> Answer : This situation appears to be caused by I/O bottlenecks. CPU 1 is waiting for I/O operations (likely disk or network-related) to complete, as indicated by the high %iowait. This kind of scenario is common when the disk or network is slow, heavily loaded, or underperforming, which forces the CPU to wait for I/O resources.
![image](https://github.com/user-attachments/assets/5f5c6d08-b875-42ce-b043-2741cf708b33)

---

### 📊 Summary

- **Average load** gives a quick overview of system performance, reflecting the overall load.
- However, a high average load doesn’t directly tell you the bottleneck.
- You need tools like `mpstat` and `pidstat` to get detailed insights into CPU and I/O usage.

---

👨‍💻 **Key Takeaways**:
1. A high average load is often caused by CPU-intensive processes.
2. High load doesn’t always mean high CPU usage—it can indicate high I/O wait.
3. Tools like `mpstat` and `pidstat` can help pinpoint the source of the load.

