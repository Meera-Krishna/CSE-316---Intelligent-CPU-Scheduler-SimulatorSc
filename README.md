# CSE-316---Intelligent-CPU-Scheduler-Simulator
'''Developing  a simulator for CPU scheduling algorithms (FCFS, SJF, Round Robin, Priority Scheduling) with real-time visualizations. The simulator  allow users to input processes with arrival times, burst times, and priorities and visualize Gantt charts and performance metrics like average waiting time and turnaround time.'''
#code for this project
import heapq
import matplotlib.pyplot as plt

def plot_gantt_chart(schedule, title):
    fig, ax = plt.subplots()
    y_labels, start_times, durations = [], [], []
    for pid, start, duration in schedule:
        y_labels.append(f"P{pid}")
        start_times.append(start)
        durations.append(duration)
    ax.barh(y_labels, durations, left=start_times, color='skyblue', edgecolor='black')
    ax.set_xlabel("Time")
    ax.set_title(title)
    plt.show()

# First Come First Serve (FCFS)
def fcfs(processes):
    processes.sort(key=lambda x: x[1])  # Sort by Arrival Time
    time, wt, tat, schedule = 0, [], [], []
    for pid, at, bt, _ in processes:
        if time < at:
            time = at  # If CPU is idle, move to the arrival time
        wt.append(time - at)
        tat.append(wt[-1] + bt)
        schedule.append((pid, time, bt))
        time += bt
    plot_gantt_chart(schedule, "FCFS Gantt Chart")
    return wt, tat

# Shortest Job First (SJF - Non-preemptive)
def sjf(processes):
    processes.sort(key=lambda x: (x[1], x[2]))  # Sort by Arrival Time, then Burst Time
    pq, time, wt, tat, schedule = [], 0, {}, {}, []
    i, n = 0, len(processes)
    while i < n or pq:
        while i < n and processes[i][1] <= time:
            heapq.heappush(pq, (processes[i][2], processes[i]))  # Sort by Burst Time
            i += 1
        if pq:
            bt, (pid, at, bt, _) = heapq.heappop(pq)
            wt[pid] = time - at
            schedule.append((pid, time, bt))
            time += bt
            tat[pid] = time - at
        else:
            time = processes[i][1]
    plot_gantt_chart(schedule, "SJF Gantt Chart")
    return [wt[p] for p, _, _, _ in processes], [tat[p] for p, _, _, _ in processes]

# Round Robin (RR)
def round_robin(processes, quantum):
    queue, time, wt, tat, schedule = processes[:], 0, {}, {}, []
    remaining = {p[0]: p[2] for p in processes}  # Track remaining burst time
    while queue:
        pid, at, bt, _ = queue.pop(0)
        if time < at:
            time = at  # If CPU is idle, move to arrival time
        executed = min(quantum, remaining[pid])
        schedule.append((pid, time, executed))
        time += executed
        remaining[pid] -= executed
        if remaining[pid] == 0:
            tat[pid] = time - at
            wt[pid] = tat[pid] - bt
        else:
            queue.append((pid, at, bt, _))  # Re-add process if not completed
    plot_gantt_chart(schedule, "Round Robin Gantt Chart")
    return [wt[p] for p, _, _, _ in processes], [tat[p] for p, _, _, _ in processes]

# Priority Scheduling (Non-Preemptive) according to priority numbering
def priority_scheduling(processes):
    processes.sort(key=lambda x: (x[1], x[3]))  # Sort by Arrival Time, then Priority
    time, wt, tat, schedule = 0, [], [], []
    for pid, at, bt, pr in sorted(processes, key=lambda x: (x[1], x[3])):
        if time < at:
            time = at
        wt.append(time - at)
        tat.append(wt[-1] + bt)
        schedule.append((pid, time, bt))
        time += bt
    plot_gantt_chart(schedule, "Priority Scheduling Gantt Chart")
    return wt, tat

# Example Usage
processes = [(1, 0, 5, 2), (2, 1, 3, 1), (3, 2, 8, 3)]  # (ID, Arrival Time, Burst Time, Priority)
quantum = 4
print("FCFS:", fcfs(processes))
print("SJF:", sjf(processes))
print("Round Robin:", round_robin(processes, quantum))
print("Priority Scheduling:", priority_scheduling(processes))
Module 1: GUI & User Input Handling
Purpose:

Provide a user-friendly interface to input process details.

Allow users to select a scheduling algorithm.

Enable dynamic interaction with scheduling parameters.

Key Features:

Input form for process details (process ID, arrival time, burst time, priority).

Dropdown selection for scheduling algorithms.

Start/Reset button to run/restart simulations.

Example:
A user enters the following details:

Process	Arrival Time	Burst Time	Priority
P1	0	5	2
P2	1	3	1
P3	2	8	3
After selecting SJF, the GUI updates to show the Gantt chart and calculated metrics.

Module 2: Scheduling Logic & Computation Engine
Purpose:

Implement core scheduling logic.

Compute execution order and time metrics.

Handle both preemptive and non-preemptive scheduling.

Key Features:

FCFS Implementation: Sort by arrival time and execute sequentially.

SJF Implementation: Choose the shortest remaining burst time first.

Round Robin Implementation: Use a fixed time quantum for execution.

Priority Scheduling Implementation: Execute processes based on priority levels.

Calculate metrics:

Waiting Time = Turnaround Time - Burst Time

Turnaround Time = Completion Time - Arrival Time

Example Calculation for FCFS:
Given:

Process	Arrival Time	Burst Time
P1	0	4
P2	1	3
P3	2	2
Execution Order: P1 → P2 → P3
Waiting Time: P1: 0, P2: 3, P3: 5
Turnaround Time: P1: 4, P2: 6, P3: 7

Module 3: Data Visualization & Metrics Display
Purpose:

Visually represent CPU scheduling execution.

Display computed metrics.

Key Features:

Gantt Chart Representation: Uses a timeline to show execution order.

Performance Metrics Display: Displays calculated AWT, TAT.

Comparison Graphs: Bar chart to compare efficiency of algorithms.

Example:
For the above FCFS example, the Gantt chart would look like:

Copy
Edit
| P1 | P1 | P1 | P1 | P2 | P2 | P2 | P3 | P3 |
0    1    2    3    4    5    6    7    8  
Graphical outputs:

Bar Chart: Average Waiting Time for each algorithm.

Line Graph: Turnaround Time trends.


