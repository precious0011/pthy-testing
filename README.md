import tkinter as tk
from tkinter import messagebox
import sqlite3
import random

# ---------------- DATABASE ----------------
conn = sqlite3.connect("glh_v5.db")
cursor = conn.cursor()

# USERS
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

# PRODUCTS
cursor.execute("""
CREATE TABLE IF NOT EXISTS products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT,
    price REAL,
    stock INTEGER
)
""")

# ORDERS (WITH TRACKING)
cursor.execute("""
CREATE TABLE IF NOT EXISTS orders (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_email TEXT,
    items TEXT,
    total REAL,
    status TEXT,
    tracking_id TEXT
)
""")

# OFFERS
cursor.execute("""
CREATE TABLE IF NOT EXISTS offers (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    product_name TEXT,
    discount REAL
)
""")

# INSERT PRODUCTS
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

# INSERT OFFERS
cursor.execute("SELECT COUNT(*) FROM offers")
if cursor.fetchone()[0] == 0:
    cursor.executemany("""
    INSERT INTO offers (product_name, discount)
    VALUES (?, ?)
    """, [
        ("Milk", 0.5),
        ("Bread", 0.3),
        ("Apple", 0.2)
    ])
    conn.commit()

# ---------------- MAIN WINDOW ----------------
root = tk.Tk()
root.title("Greenfield Local Hub V5")
root.geometry("750x550")

# ---------------- LOGIN ----------------
def open_login():
    win = tk.Toplevel(root)
    win.title("Login")
    win.geometry("300x300")

    tk.Label(win, text="Email").pack()
    email = tk.Entry(win)
    email.pack()

    tk.Label(win, text="Password").pack()
    password = tk.Entry(win, show="*")
    password.pack()

    def login():
        cursor.execute("SELECT * FROM users WHERE email=? AND password=?",
                       (email.get().lower(), password.get()))
        user = cursor.fetchone()

        if user:
            win.destroy()
            open_customer_dashboard(user)
        else:
            messagebox.showerror("Error", "Invalid login")

    tk.Button(win, text="Login", command=login).pack(pady=10)
    tk.Button(win, text="Register",
              command=lambda: open_register(win)).pack()

# ---------------- REGISTER ----------------
def open_register(prev):
    prev.destroy()
    win = tk.Toplevel(root)

    fields = {}
    for label in ["Username", "Email", "Password", "Address", "County"]:
        tk.Label(win, text=label).pack()
        entry = tk.Entry(win, show="*" if label=="Password" else None)
        entry.pack()
        fields[label] = entry

    def register():
        data = {k: v.get() for k, v in fields.items()}
        cursor.execute("""
        INSERT INTO users (username, email, password, address, county)
        VALUES (?, ?, ?, ?, ?)
        """, (data["Username"], data["Email"], data["Password"],
              data["Address"], data["County"]))
        conn.commit()
        win.destroy()

    tk.Button(win, text="Register", command=register).pack()

# ---------------- CUSTOMER DASHBOARD ----------------
def open_customer_dashboard(user):
    dash = tk.Toplevel(root)
    dash.geometry("1000x700")

    user_email = user[2]
    cart = []

    header = tk.Frame(dash, bg="#2e7d32")
    header.pack(fill="x")

    tk.Label(header, text="GLH Store", fg="white",
             bg="#2e7d32").pack(side="left")

    # SEARCH
    search = tk.Entry(header)
    search.pack(side="left")

    def get_discount(name):
        cursor.execute("SELECT discount FROM offers WHERE product_name=?", (name,))
        r = cursor.fetchone()
        return r[0] if r else 0

    main = tk.Frame(dash)
    main.pack()

    def display(products):
        for w in main.winfo_children():
            w.destroy()

        for p in products:
            box = tk.Frame(main, bd=1)
            box.pack(pady=5)

            discount = get_discount(p[0])
            price = p[1] - discount

            tk.Label(box, text=p[0]).pack()

            if discount > 0:
                tk.Label(box, text=f"£{p[1]}", fg="grey").pack()
                tk.Label(box, text=f"£{price} OFFER!", fg="red").pack()
            else:
                tk.Label(box, text=f"£{p[1]}").pack()

            tk.Button(box, text="Add",
                      command=lambda n=p[0], pr=price: cart.append((n, pr))).pack()

    def search_products():
        cursor.execute("SELECT name, price FROM products WHERE name LIKE ?",
                       ('%' + search.get() + '%',))
        display(cursor.fetchall())

    tk.Button(header, text="Search", command=search_products).pack(side="left")

    # CART
    def checkout():
        if not cart:
            return

        items = ", ".join([i[0] for i in cart])
        total = sum([i[1] for i in cart])
        tracking = "TRK" + str(random.randint(10000, 99999))

        cursor.execute("""
        INSERT INTO orders (user_email, items, total, status, tracking_id)
        VALUES (?, ?, ?, ?, ?)
        """, (user_email, items, total, "Processing", tracking))

        conn.commit()
        cart.clear()
        messagebox.showinfo("Order", f"Tracking ID: {tracking}")

    tk.Button(header, text="Checkout", command=checkout).pack(side="right")

    # VIEW ORDERS
    def view_orders():
        win = tk.Toplevel(dash)
        cursor.execute("""
        SELECT items, total, status, tracking_id
        FROM orders WHERE user_email=?
        """, (user_email,))
        for o in cursor.fetchall():
            tk.Label(win, text=f"{o[0]} | £{o[1]}").pack()
            tk.Label(win, text=f"{o[2]} | {o[3]}").pack()

    tk.Button(header, text="Orders", command=view_orders).pack(side="right")

    cursor.execute("SELECT name, price FROM products")
    display(cursor.fetchall())

# ---------------- PRODUCER DASHBOARD ----------------
def open_producer_dashboard():
    dash = tk.Toplevel(root)
    dash.geometry("1000x600")

    sidebar = tk.Frame(dash, bg="#1e1e2f", width=200)
    sidebar.pack(side="left", fill="y")

    content = tk.Frame(dash)
    content.pack(side="right", expand=True, fill="both")

    def clear():
        for w in content.winfo_children():
            w.destroy()

    def products():
        clear()
        cursor.execute("SELECT name, price, stock FROM products")
        for p in cursor.fetchall():
            tk.Label(content, text=f"{p[0]} £{p[1]} Stock:{p[2]}").pack()

    def update_stock():
        clear()
        name = tk.Entry(content)
        name.pack()
        stock = tk.Entry(content)
        stock.pack()

        def update():
            cursor.execute("UPDATE products SET stock=? WHERE name=?",
                           (stock.get(), name.get()))
            conn.commit()

        tk.Button(content, text="Update", command=update).pack()

    def orders():
        clear()
        cursor.execute("SELECT user_email, items, total, status, tracking_id FROM orders")
        for o in cursor.fetchall():
            tk.Label(content, text=f"{o[0]} | {o[1]} | £{o[2]}").pack()
            tk.Label(content, text=f"{o[3]} | {o[4]}").pack()

    def update_status():
        win = tk.Toplevel(dash)
        t = tk.Entry(win)
        t.pack()
        s = tk.Entry(win)
        s.pack()

        def upd():
            cursor.execute("UPDATE orders SET status=? WHERE tracking_id=?",
                           (s.get(), t.get()))
            conn.commit()

        tk.Button(win, text="Update", command=upd).pack()

    tk.Button(sidebar, text="Products", command=products).pack(pady=10)
    tk.Button(sidebar, text="Stock", command=update_stock).pack(pady=10)
    tk.Button(sidebar, text="Orders", command=orders).pack(pady=10)
    tk.Button(sidebar, text="Update Status", command=update_status).pack(pady=10)

# ---------------- BUTTONS ----------------
tk.Button(root, text="Customer", command=open_login).pack(pady=20)
tk.Button(root, text="Producer", command=open_producer_dashboard).pack()

root.mainloop()
