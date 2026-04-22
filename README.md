Nice — adding tracking details + special offers is exactly what makes your Version 5 feel like a real system. I’ll give you:

1. ✅ Database updates
2. ✅ Tracking system (customer + producer)
3. ✅ Special offers system
4. ✅ Code you can plug straight in
5. ✅ What to write in coursework

⸻

🧾 ✅ 1. DATABASE UPDATES (VERY IMPORTANT)

🔴 Update your orders table to include tracking:

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

⸻

🟢 OPTIONAL: Add offers table

cursor.execute("""
CREATE TABLE IF NOT EXISTS offers (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    product_name TEXT,
    discount REAL
)
""")

⸻

🚚 ✅ 2. TRACKING SYSTEM (CUSTOMER SIDE)

🔧 UPDATE CHECKOUT FUNCTION

Replace your checkout with:

import random
def checkout():
    if not cart:
        messagebox.showerror("Error", "Cart empty")
        return
    items = ", ".join([i[0] for i in cart])
    total = sum([i[1] for i in cart])
    tracking_id = "TRK" + str(random.randint(10000, 99999))
    cursor.execute("""
    INSERT INTO orders (user_email, items, total, status, tracking_id)
    VALUES (?, ?, ?, ?, ?)
    """, (user_email, items, total, "Processing", tracking_id))
    conn.commit()
    cart.clear()
    messagebox.showinfo("Success", f"Order placed\nTracking ID: {tracking_id}")

⸻

👀 VIEW ORDERS WITH TRACKING

Update your view_orders():

cursor.execute("""
SELECT items, total, status, tracking_id 
FROM orders WHERE user_email=?
""", (user_email,))
for o in cursor.fetchall():
    tk.Label(win, text=f"{o[0]} | £{o[1]}").pack()
    tk.Label(win, text=f"Status: {o[2]} | Tracking: {o[3]}").pack()

⸻

🧑‍💼 ✅ 3. PRODUCER: UPDATE ORDER STATUS

Add this inside producer dashboard:

def update_order_status():
    win = tk.Toplevel(dash)
    tk.Label(win, text="Tracking ID").pack()
    tracking = tk.Entry(win)
    tracking.pack()
    tk.Label(win, text="New Status").pack()
    status = tk.Entry(win)
    status.pack()
    def update():
        cursor.execute("""
        UPDATE orders SET status=? WHERE tracking_id=?
        """, (status.get(), tracking.get()))
        conn.commit()
        messagebox.showinfo("Success", "Order updated")
    tk.Button(win, text="Update Status", command=update).pack()

👉 Add button:

tk.Button(dash, text="Update Order Status", command=update_order_status).pack(pady=10)

⸻

💸 ✅ 4. SPECIAL OFFERS SYSTEM

⸻

🔧 ADD SAMPLE OFFERS

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

⸻

🛒 APPLY DISCOUNTS WHEN SHOWING PRODUCTS

Modify product loading:

cursor.execute("SELECT name, price FROM products")
products = cursor.fetchall()
def get_discount(name):
    cursor.execute("SELECT discount FROM offers WHERE product_name=?", (name,))
    result = cursor.fetchone()
    return result[0] if result else 0

⸻

🖥️ DISPLAY WITH OFFERS

for p in products:
    discount = get_discount(p[0])
    final_price = p[1] - discount
    tk.Label(box, text=p[0]).pack()
    if discount > 0:
        tk.Label(box, text=f"£{p[1]}", fg="grey").pack()
        tk.Label(box, text=f"£{final_price} (Offer!)", fg="red").pack()
    else:
        tk.Label(box, text=f"£{p[1]}", fg="green").pack()

⸻




Good — adding references is what separates a pass from a distinction.
I’ll give you proper academic-style references you can use in your coursework.

⸻

📚 ✅ REFERENCES FOR YOUR PROJECT

🧠 Programming & GUI (Tkinter)

* Python Documentation. (2025). Tkinter GUI Programming.
    Available at: https://docs.python.org/3/library/tkinter.html
* W3Schools. (2025). Python GUI Tkinter Tutorial.
    Available at: https://www.w3schools.com/python/python_gui.asp

⸻

🗄️ Database (SQLite)

* SQLite Documentation. (2025). SQLite Home Page.
    Available at: https://www.sqlite.org/index.html
* Python Documentation. (2025). sqlite3 Module.
    Available at: https://docs.python.org/3/library/sqlite3.html

⸻

🧩 UI / UX Design Principles

* Interaction Design Foundation. (2024). User Interface Design Basics.
    Available at: https://www.interaction-design.org
* Nielsen Norman Group. (2024). Usability Heuristics for User Interface Design.
    Available at: https://www.nngroup.com/articles/ten-usability-heuristics/

⸻

🛒 E-commerce Features (Tracking & Offers)

* Amazon. (2025). Order Tracking System.
    Used as real-world reference for tracking IDs and order status.
* Tesco. (2025). Online Grocery Offers System.
    Used as inspiration for discount and offer features.

⸻

🔐 Security & Validation (Optional but Strong)

* OWASP. (2024). Input Validation Guidelines.
    Available at: https://owasp.org

⸻

📝 HOW TO WRITE THEM (FOR YOUR COURSEWORK)

If your teacher wants Harvard style, you can write like this:

Python Software Foundation (2025) Tkinter Documentation. Available at: https://docs.python.org (Accessed: 20 April 2026).

⸻

🎯 💡 WHERE TO PUT REFERENCES

* Task 1 → Design & research
* Task 2 → Development justification
* End of document → Full reference list

⸻

🚀 EXTRA (FOR HIGHER MARKS)

You can also write:

The system design was influenced by real-world platforms such as Amazon and Tesco, particularly in features like order tracking and promotional offers.

⸻

If you want, I can:
👉 ￼ Format all your references into a perfect Harvard reference page
👉 Or ￼ insert them into your PowerPoint automatically

Just tell me 👍
