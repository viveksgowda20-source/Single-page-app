import tkinter as tk
from tkinter import messagebox
from datetime import datetime
import json
import os

# -----------------------------
# FILE FOR SAVING TASKS
# -----------------------------
DATA_FILE = "tasks.json"

tasks = []


# -----------------------------
# LOAD TASKS
# -----------------------------
def load_tasks():
    global tasks

    if os.path.exists(DATA_FILE):
        try:
            with open(DATA_FILE, "r") as file:
                tasks = json.load(file)
        except:
            tasks = []
    else:
        tasks = []


# -----------------------------
# SAVE TASKS
# -----------------------------
def save_tasks():
    with open(DATA_FILE, "w") as file:
        json.dump(tasks, file, indent=4)


# -----------------------------
# REFRESH TASK LIST
# -----------------------------
def refresh_tasks():
    task_list.delete(0, tk.END)

    search_text = search_entry.get().lower()

    for i, task in enumerate(tasks):

        if search_text not in task["title"].lower():
            continue

        status = "✓" if task["completed"] else "○"

        text = (
            f"{status}  {task['title']}   "
            f"| {task['date']} {task['time']}"
        )

        task_list.insert(tk.END, text)

        if task["completed"]:
            task_list.itemconfig(
                tk.END,
                fg="gray"
            )
        else:
            task_list.itemconfig(
                tk.END,
                fg="black"
            )

    update_counter()


# -----------------------------
# ADD TASK
# -----------------------------
def add_task():
    title = task_entry.get().strip()

    if title == "":
        messagebox.showwarning(
            "Warning",
            "Please enter a task."
        )
        return

    task = {
        "title": title,
        "date": date_entry.get(),
        "time": time_entry.get(),
        "completed": False
    }

    tasks.append(task)

    save_tasks()
    refresh_tasks()

    task_entry.delete(0, tk.END)


# -----------------------------
# DELETE TASK
# -----------------------------
def delete_task():
    selected = task_list.curselection()

    if not selected:
        messagebox.showwarning(
            "Warning",
            "Please select a task."
        )
        return

    display_index = selected[0]

    search_text = search_entry.get().lower()

    visible_tasks = []

    for i, task in enumerate(tasks):
        if search_text in task["title"].lower():
            visible_tasks.append(i)

    real_index = visible_tasks[display_index]

    tasks.pop(real_index)

    save_tasks()
    refresh_tasks()


# -----------------------------
# COMPLETE TASK
# -----------------------------
def complete_task():
    selected = task_list.curselection()

    if not selected:
        messagebox.showwarning(
            "Warning",
            "Please select a task."
        )
        return

    display_index = selected[0]

    search_text = search_entry.get().lower()

    visible_tasks = []

    for i, task in enumerate(tasks):
        if search_text in task["title"].lower():
            visible_tasks.append(i)

    real_index = visible_tasks[display_index]

    tasks[real_index]["completed"] = not tasks[real_index]["completed"]

    save_tasks()
    refresh_tasks()


# -----------------------------
# EDIT TASK
# -----------------------------
def edit_task():
    selected = task_list.curselection()

    if not selected:
        messagebox.showwarning(
            "Warning",
            "Please select a task."
        )
        return

    display_index = selected[0]

    search_text = search_entry.get().lower()

    visible_tasks = []

    for i, task in enumerate(tasks):
        if search_text in task["title"].lower():
            visible_tasks.append(i)

    real_index = visible_tasks[display_index]

    task = tasks[real_index]

    edit_window = tk.Toplevel(root)
    edit_window.title("Edit Task")
    edit_window.geometry("400x250")
    edit_window.resizable(False, False)

    tk.Label(
        edit_window,
        text="Edit Task",
        font=("Arial", 18, "bold")
    ).pack(pady=15)

    edit_entry = tk.Entry(
        edit_window,
        font=("Arial", 13),
        width=35
    )
    edit_entry.pack(pady=10)

    edit_entry.insert(0, task["title"])

    def save_edit():

        new_title = edit_entry.get().strip()

        if new_title == "":
            messagebox.showwarning(
                "Warning",
                "Task cannot be empty."
            )
            return

        tasks[real_index]["title"] = new_title

        save_tasks()
        refresh_tasks()

        edit_window.destroy()

    tk.Button(
        edit_window,
        text="Save Changes",
        command=save_edit,
        bg="#4CAF50",
        fg="white",
        font=("Arial", 11, "bold"),
        width=18
    ).pack(pady=15)


# -----------------------------
# SEARCH
# -----------------------------
def search_tasks(event=None):
    refresh_tasks()


# -----------------------------
# UPDATE COUNTER
# -----------------------------
def update_counter():

    total = len(tasks)

    completed = 0

    for task in tasks:
        if task["completed"]:
            completed += 1

    remaining = total - completed

    counter_label.config(
        text=f"Total: {total}     "
             f"Completed: {completed}     "
             f"Remaining: {remaining}"
    )


# -----------------------------
# CURRENT DATE AND TIME
# -----------------------------
def set_current_date():

    now = datetime.now()

    date_entry.delete(0, tk.END)
    date_entry.insert(
        0,
        now.strftime("%d-%m-%Y")
    )

    time_entry.delete(0, tk.END)
    time_entry.insert(
        0,
        now.strftime("%H:%M")
    )


# -----------------------------
# MAIN WINDOW
# -----------------------------
root = tk.Tk()

root.title("My To-Do App")
root.geometry("850x650")
root.minsize(750, 550)

root.configure(bg="#F4F6F8")


# -----------------------------
# HEADER
# -----------------------------
header = tk.Frame(
    root,
    bg="#263238",
    height=90
)

header.pack(
    fill="x"
)

title_label = tk.Label(
    header,
    text="MY TO-DO APP",
    bg="#263238",
    fg="white",
    font=("Arial", 26, "bold")
)

title_label.pack(pady=25)


# -----------------------------
# ADD TASK AREA
# -----------------------------
input_frame = tk.Frame(
    root,
    bg="#F4F6F8"
)

input_frame.pack(
    pady=20
)


tk.Label(
    input_frame,
    text="Task",
    bg="#F4F6F8",
    font=("Arial", 11, "bold")
).grid(
    row=0,
    column=0,
    padx=5
)


task_entry = tk.Entry(
    input_frame,
    width=35,
    font=("Arial", 12)
)

task_entry.grid(
    row=0,
    column=1,
    padx=5
)


tk.Label(
    input_frame,
    text="Date",
    bg="#F4F6F8",
    font=("Arial", 11, "bold")
).grid(
    row=0,
    column=2,
    padx=5
)


date_entry = tk.Entry(
    input_frame,
    width=12,
    font=("Arial", 11)
)

date_entry.grid(
    row=0,
    column=3,
    padx=5
)


tk.Label(
    input_frame,
    text="Time",
    bg="#F4F6F8",
    font=("Arial", 11, "bold")
).grid(
    row=0,
    column=4,
    padx=5
)


time_entry = tk.Entry(
    input_frame,
    width=8,
    font=("Arial", 11)
)

time_entry.grid(
    row=0,
    column=5,
    padx=5
)


set_current_date()


# -----------------------------
# ADD BUTTON
# -----------------------------
add_button = tk.Button(
    input_frame,
    text="＋ Add Task",
    command=add_task,
    bg="#1976D2",
    fg="white",
    font=("Arial", 11, "bold"),
    width=12
)

add_button.grid(
    row=1,
    column=1,
    pady=15
)


# -----------------------------
# SEARCH
# -----------------------------
search_frame = tk.Frame(
    root,
    bg="#F4F6F8"
)

search_frame.pack(
    pady=5
)


tk.Label(
    search_frame,
    text="Search:",
    bg="#F4F6F8",
    font=("Arial", 11, "bold")
).pack(
    side="left",
    padx=5
)


search_entry = tk.Entry(
    search_frame,
    width=40,
    font=("Arial", 12)
)

search_entry.pack(
    side="left"
)

search_entry.bind(
    "<KeyRelease>",
    search_tasks
)


# -----------------------------
# TASK LIST
# -----------------------------
list_frame = tk.Frame(
    root,
    bg="white",
    bd=1,
    relief="solid"
)

list_frame.pack(
    padx=40,
    pady=15,
    fill="both",
    expand=True
)


scrollbar = tk.Scrollbar(
    list_frame
)

scrollbar.pack(
    side="right",
    fill="y"
)


task_list = tk.Listbox(
    list_frame,
    font=("Arial", 13),
    selectmode=tk.SINGLE,
    yscrollcommand=scrollbar.set,
    bg="white",
    activestyle="none"
)

task_list.pack(
    fill="both",
    expand=True,
    padx=5,
    pady=5
)


scrollbar.config(
    command=task_list.yview
)


# -----------------------------
# BUTTON AREA
# -----------------------------
button_frame = tk.Frame(
    root,
    bg="#F4F6F8"
)

button_frame.pack(
    pady=10
)


complete_button = tk.Button(
    button_frame,
    text="✓ Complete",
    command=complete_task,
    bg="#4CAF50",
    fg="white",
    font=("Arial", 11, "bold"),
    width=14
)

complete_button.grid(
    row=0,
    column=0,
    padx=5
)
