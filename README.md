verson one 

import tkinter as tk
from tkinter import messagebox

root = tk.Tk()
root.title("GLH Prototype V1")
root.geometry("400x400")

# simple login screen
tk.Label(root, text="Login", font=("Arial", 14)).pack(pady=10)

tk.Label(root, text="Email").pack()
email = tk.Entry(root)
email.pack()

tk.Label(root, text="Password").pack()
password = tk.Entry(root, show="*")
password.pack()

def login():
    # basic check (no database yet)
    if email.get() == "" or password.get() == "":
        messagebox.showerror("Error", "Fill all fields")
    else:
        open_dashboard()

def open_dashboard():
    dash = tk.Toplevel(root)
    dash.title("Dashboard")

    # simple product list
    products = ["Milk", "Bread", "Eggs"]

    for p in products:
        tk.Label(dash, text=p).pack()

tk.Button(root, text="Login", command=login).pack(pady=10)

root.mainloop()


version 2
import tkinter as tk
from tkinter import messagebox
import sqlite3

# database added in this version
conn = sqlite3.connect("v2.db")
cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS users (
    email TEXT,
    password TEXT
)
""")

cursor.execute("""
CREATE TABLE IF NOT EXISTS orders (
    items TEXT
)
""")

conn.commit()

root = tk.Tk()
root.title("GLH Prototype V2")
root.geometry("500x450")

cart = []

# login
tk.Label(root, text="Login").pack()

email = tk.Entry(root)
email.pack()

password = tk.Entry(root, show="*")
password.pack()

def register():
    cursor.execute("INSERT INTO users VALUES (?, ?)",
                   (email.get(), password.get()))
    conn.commit()
    messagebox.showinfo("Done", "Registered")

def login():
    cursor.execute("SELECT * FROM users WHERE email=? AND password=?",
                   (email.get(), password.get()))
    if cursor.fetchone():
        open_dashboard()
    else:
        messagebox.showerror("Error", "Invalid")

def open_dashboard():
    dash = tk.Toplevel(root)

    products = [("Milk", 1.5), ("Bread", 1.2)]

    def add_cart(name):
        cart.append(name)

    def checkout():
        cursor.execute("INSERT INTO orders VALUES (?)",
                       (",".join(cart),))
        conn.commit()
        messagebox.showinfo("Done", "Order saved")

    for p in products:
        tk.Button(dash, text=p[0],
                  command=lambda n=p[0]: add_cart(n)).pack()

    tk.Button(dash, text="Checkout", command=checkout).pack()

tk.Button(root, text="Login", command=login).pack()
tk.Button(root, text="Register", command=register).pack()

root.mainloop()


version 3
import tkinter as tk
from tkinter import messagebox
import sqlite3

# ---------------- DATABASE ----------------
conn = sqlite3.connect("glh_final.db")
cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT,
    email TEXT,
    password TEXT,
    address TEXT,
    county TEXT
)
""")

cursor.execute("""
CREATE TABLE IF NOT EXISTS orders (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_email TEXT,
    items TEXT,
    total REAL
)
""")

cursor.execute("""
CREATE TABLE IF NOT EXISTS products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT,
    price REAL,
    stock INTEGER
)
""")

conn.commit()

# ---------------- INSERT PRODUCTS ----------------
cursor.execute("SELECT COUNT(*) FROM products")
if cursor.fetchone()[0] == 0:
    cursor.executemany("""
    INSERT INTO products (name, price, stock)
    VALUES (?, ?, ?)
    """, [
        ("Milk", 1.50, 10),
        ("Bread", 1.20, 15),
        ("Eggs", 2.50, 20),
        ("Vegetables", 3.00, 12),
        ("Apple", 1.00, 25),
        ("Cheese", 2.80, 8),
        ("Juice", 1.90, 18),
        ("Chicken", 4.50, 6)
    ])
    conn.commit()

# ---------------- MAIN WINDOW ----------------
root = tk.Tk()
root.title("Greenfield Local Hub")
root.geometry("750x550")
root.configure(bg="#e8f5e9")

header = tk.Frame(root, bg="#2e7d32", height=80)
header.pack(fill="x")

tk.Label(header, text="Greenfield Local Hub",
         fg="white", bg="#2e7d32",
         font=("Arial", 18, "bold")).pack(pady=20)

card = tk.Frame(root, bg="white", bd=2, relief="raised")
card.place(relx=0.5, rely=0.5, anchor="center", width=420, height=350)

tk.Label(card, text="Welcome", font=("Arial", 14, "bold"), bg="white").pack()
tk.Label(card, text="Choose your role", bg="white").pack(pady=5)

# ---------------- IMAGE MAPPING ----------------
product_images = {
    "Milk": "images/milk.png",
    "Bread": "images/bread.png",
    "Eggs": "images/eggs.png",
    "Vegetables": "images/vegetables.png",
    "Apple": "images/apple.png",
    "Cheese": "images/cheese.png",
    "Juice": "images/juice.png",
    "Chicken": "images/chicken.png"
}

# ---------------- LOGIN ----------------
def open_login():
    win = tk.Toplevel(root)
    win.title("Login")
    win.geometry("350x300")

    tk.Label(win, text="Login", font=("Arial", 14)).pack(pady=10)

    tk.Label(win, text="Email").pack()
    email = tk.Entry(win)
    email.pack()

    tk.Label(win, text="Password").pack()
    password = tk.Entry(win, show="*")
    password.pack()

    def login():
        e = email.get().strip().lower()
        p = password.get().strip()

        cursor.execute("SELECT * FROM users WHERE email=? AND password=?", (e, p))
        user = cursor.fetchone()

        if user:
            win.destroy()
            open_customer_dashboard(user)
        else:
            messagebox.showerror("Error", "Invalid details")

    tk.Button(win, text="Login", bg="#4CAF50", fg="white",
              command=login).pack(pady=10)

# ---------------- CUSTOMER DASHBOARD ----------------
def open_customer_dashboard(user):
    dash = tk.Toplevel(root)
    dash.title("Customer Dashboard")
    dash.geometry("900x600")

    user_email = user[2]
    cart = []

    # MUST keep references to images or Tkinter deletes them
    images_ref = []

    def view_cart():
        win = tk.Toplevel(dash)
        total = sum([i[1] for i in cart])

        for i in cart:
            tk.Label(win, text=f"£{i[1]}").pack()

        tk.Label(win, text=f"Total: £{total}").pack()

        def checkout():
            if not cart:
                messagebox.showerror("Error", "Cart empty")
                return

            items = ", ".join([i[0] for i in cart])

            cursor.execute(
                "INSERT INTO orders (user_email, items, total) VALUES (?, ?, ?)",
                (user_email, items, total)
            )
            conn.commit()

            cart.clear()
            messagebox.showinfo("Success", "Order placed")

        tk.Button(win, text="Checkout", command=checkout).pack()

    # ---------------- PRODUCT DISPLAY (IMAGES ONLY) ----------------
    main = tk.Frame(dash, bg="white")
    main.pack(fill="both", expand=True)

    cursor.execute("SELECT name, price FROM products")
    products = cursor.fetchall()

    def add_to_cart(name, price):
        cart.append((name, price))
        messagebox.showinfo("Cart", f"Added item")

    r = c = 0

    for p in products:
        box = tk.Frame(main, bd=1, relief="solid", bg="white")
        box.grid(row=r, column=c, padx=15, pady=15)

        # load image
        img_path = product_images.get(p[0])

        try:
            img = tk.PhotoImage(file=img_path)
            images_ref.append(img)

            tk.Label(box, image=img, bg="white").pack()

        except:
            tk.Label(box, text="No Image").pack()

        tk.Button(box, text="Add",
                  command=lambda n=p[0], pr=p[1]: add_to_cart(n, pr)).pack()

        c += 1
        if c == 3:
            c = 0
            r += 1

    tk.Button(dash, text="View Cart", command=view_cart).pack()

# ---------------- BUTTONS ----------------
tk.Button(card, text="Customer", bg="#4CAF50", fg="white",
          width=20, command=open_login).pack(pady=10)

root.mainloop()
