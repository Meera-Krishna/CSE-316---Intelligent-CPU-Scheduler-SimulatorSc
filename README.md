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
