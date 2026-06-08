import tkinter as tk
from tkinter import messagebox
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
import matplotlib.pyplot as plt

high = ["lottery", "winner", "won", "prize", "cash", "claim",
        "urgent", "blocked", "suspended", "password reset",
        "account suspended", "aadhar card", "atm card number"]

medium = ["verify", "otp", "click", "offer", "reward",
          "coupon", "promotion", "free"]

low = ["update", "notification", "support team",
       "order", "delivery", "note", "important", "account"]

weights = {}
for w in high:
    weights[w] = 15
for w in medium:
    weights[w] = 8
for w in low:
    weights[w] = 4

def compute_score(message):
    msg = message.lower()
    score = 0
    matched = set()
    for k, wt in weights.items():
        if k in msg and k not in matched:
            score += wt
            matched.add(k)
    pct = min(score, 100)
    return pct, matched

def classify_pct(pct):
    if pct >= 60:
        return "spam"
    elif pct >= 30:
        return "confusing"
    else:
        return "ham"

def show_pie(spam_pct):
    not_pct = 100 - spam_pct
    fig, ax = plt.subplots(figsize=(3.8, 3.8))
    colors = ["#FF6B6B", "#7CE3BD"]
    ax.pie([spam_pct, not_pct],
           labels=[f"Spam {spam_pct:.1f}%", f"Not Spam {not_pct:.1f}%"],
           startangle=90,
           colors=colors,
           wedgeprops={'linewidth': 1, 'edgecolor': '#ffffff'})
    ax.set_title("Spam Probability", fontsize=12, fontweight="bold")
    fig.tight_layout()
    return fig

root = tk.Tk()
root.title("Spam Classifier - With Pie Chart")
root.geometry("820x650")
root.configure(bg="#f0f0ff")

left = tk.Frame(root, bg="#fff0f6", bd=3, relief="groove")
left.place(x=20, y=20, width=500, height=600)

lbl = tk.Label(left, text="Paste Email Content:",
               font=("Helvetica", 14, "bold"),
               bg="#fff0f6", fg="#5b0c46")
lbl.pack(pady=(12, 6))

text_box = tk.Text(left, height=20, width=55,
                   font=("Arial", 11),
                   bg="#fff8fb", bd=2, relief="solid")
text_box.pack(padx=10)

result_label = tk.Label(left, text="", font=("Arial", 18, "bold"),
                        bg="#fff0f6")
result_label.pack(pady=5)

percent_label = tk.Label(left, text="", font=("Arial", 14),
                         bg="#fff0f6")
percent_label.pack(pady=5)

right = tk.Frame(root, bg="#eef0ff", bd=3, relief="groove")
right.place(x=540, y=20, width=260, height=600)

chart_title = tk.Label(right, text="Analysis",
                       font=("Helvetica", 16, "bold"),
                       bg="#eef0ff", fg="#4b0b7a")
chart_title.pack(pady=10)

canvas_holder = tk.Frame(right, bg="#eef0ff")
canvas_holder.pack()

def display_chart(fig):
    for widget in canvas_holder.winfo_children():
        widget.destroy()
    canvas = FigureCanvasTkAgg(fig, master=canvas_holder)
    canvas.draw()
    canvas.get_tk_widget().pack(pady=10)

def classify_action():
    msg = text_box.get("1.0", tk.END).strip()
    if msg == "":
        messagebox.showerror("Error", "Paste an email first!")
        return
    pct, matched = compute_score(msg)
    category = classify_pct(pct)
    matched_keywords = ", ".join(matched) if matched else "none"

    if category == "spam":
        result_label.config(text="Result: SPAM ❌", fg="#b00000")
    elif category == "ham":
        result_label.config(text="Result: NOT SPAM ✔", fg="#008a3d")
    else:
        result_label.config(text="Result: Warning ⚠", fg="#b36f00")

    percent_label.config(text=f"Spam Score: {pct:.1f}% | Matched: {matched_keywords}")
    fig = show_pie(pct)
    display_chart(fig)

btn = tk.Button(left, text="Analyze Email",
                command=classify_action,
                bg="#9b6bff", fg="white",
                font=("Helvetica", 12, "bold"),
                bd=0, width=18)
btn.pack(pady=10)

display_chart(show_pie(0))

root.mainloop()
