# Fetch exactly what you need in the right order:
cursor.execute("SELECT name, price, img_file FROM products WHERE name LIKE ?", (term,))
