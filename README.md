# Hotel-Management-System
import tkinter as tk
from tkinter import ttk, messagebox
import mysql.connector

class UserLogin:
    def __init__(self, root):
        self.root = root
        self.root.title("User Login")
        self.root.geometry("300x200")

        self.create_widgets()

    def create_widgets(self):
        # Username Entry
        ttk.Label(self.root, text="Username:").pack()
        self.username_entry = ttk.Entry(self.root)
        self.username_entry.pack()

        # Password Entry
        ttk.Label(self.root, text="Password:").pack()
        self.password_entry = ttk.Entry(self.root, show="*")
        self.password_entry.pack()

        # Login Button
        ttk.Button(self.root, text="Login", command=self.login).pack()

    def login(self):
        username = self.username_entry.get()
        password = self.password_entry.get()

        # Hardcoded username and password for demonstration purposes
        if username == "hotel" and password == "123":
            messagebox.showinfo("Login Successful", "Welcome, " + username + "!")
            self.root.destroy()  # Close the login window
            # Open the main hotel management window
            app = HotelManagement()
            app.root.mainloop()
        else:
            messagebox.showerror("Login Failed", "Invalid username or password.")

class HotelManagement:
    def __init__(self):
        self.root = tk.Tk()
        self.root.title("Hotel Management System")
        self.root.geometry("1000x500")

        # Connect to the database
        self.db_connection = mysql.connector.connect(
            host="localhost",
            user="root",
            password="aaaa",
            database="shrihotel"
        )
        self.db_cursor = self.db_connection.cursor()

        self.create_widgets()

    def create_widgets(self):
        # Customer Details Frame
        customer_frame = ttk.LabelFrame(self.root, text="Customer Details")
        customer_frame.place(x=20, y=20, width=600, height=400)

        labels = ["Customer Name:", "Mobile Number:", "Email:", "Nationality:", "Gender:", "Room Details:", "Address:"]
        for i, label_text in enumerate(labels):
            ttk.Label(customer_frame, text=label_text).grid(row=i, column=0, padx=10, pady=5, sticky="w")

        self.customer_name_entry = ttk.Entry(customer_frame, width=40)
        self.customer_name_entry.grid(row=0, column=1, padx=10, pady=5)
        self.mobile_entry = ttk.Entry(customer_frame, width=40)
        self.mobile_entry.grid(row=1, column=1, padx=10, pady=5)
        self.email_entry = ttk.Entry(customer_frame, width=40)
        self.email_entry.grid(row=2, column=1, padx=10, pady=5)
        self.nationality_entry = ttk.Entry(customer_frame, width=40)
        self.nationality_entry.grid(row=3, column=1, padx=10, pady=5)

        self.gender_var = tk.StringVar()
        ttk.Radiobutton(customer_frame, text="Male", variable=self.gender_var, value="Male").grid(row=4, column=0, padx=10, pady=5, sticky="w")
        ttk.Radiobutton(customer_frame, text="Female", variable=self.gender_var, value="Female").grid(row=4, column=1, padx=10, pady=5, sticky="w")
        ttk.Radiobutton(customer_frame, text="Other", variable=self.gender_var, value="Other").grid(row=4, column=2, padx=10, pady=5, sticky="w")

        self.room_details_entry = ttk.Entry(customer_frame, width=40)
        self.room_details_entry.grid(row=5, column=1, padx=10, pady=5)

        self.address_text = tk.Text(customer_frame, width=40, height=4)
        self.address_text.grid(row=6, column=1, padx=10, pady=5, columnspan=2)

        # Navigation Buttons
        ttk.Button(self.root, text="Submit", command=self.insert_customer_details).place(x=800, y=420)

        # View Button
        ttk.Button(self.root, text="View", command=self.view_customer_details).place(x=700, y=420)

    def validate_customer_details(self):
        # Validation code here
        return True

    def insert_customer_details(self):
        if self.validate_customer_details():
            # Get values from the entry fields
            customer_name = self.customer_name_entry.get()
            mobile_number = self.mobile_entry.get()
            email = self.email_entry.get()
            nationality = self.nationality_entry.get()
            gender = self.gender_var.get()
            room_details = self.room_details_entry.get()
            address = self.address_text.get("1.0", tk.END).strip()

            # Insert data into the database
            sql = "INSERT INTO customer (customer_name, mobile_number, email, nationality, gender, room_details, address) VALUES (%s, %s, %s, %s, %s, %s, %s)"
            values = (customer_name, mobile_number, email, nationality, gender, room_details, address)
            self.db_cursor.execute(sql, values)
            self.db_connection.commit()

            messagebox.showinfo("Success", "Customer details submitted successfully.")

    def view_customer_details(self):
        # Fetch data from the database
        self.db_cursor.execute("SELECT * FROM customer")
        data = self.db_cursor.fetchall()

        # Create a new window to display the fetched data
        view_window = tk.Toplevel(self.root)
        view_window.title("View Customer Details")
        view_window.geometry("600x400")

        # Display the fetched data in the new window
        ttk.Label(view_window, text="Customer Details").pack()
        for row in data:
            ttk.Label(view_window, text=row).pack()

            # Add delete button for each row
            ttk.Button(view_window, text="Delete", command=lambda row=row: self.delete_customer_details(row[0])).pack()

        # Bring the view window to focus
        view_window.focus_set()

    def delete_customer_details(self, customer_id):
        # Delete the selected customer details from the database
        sql = "DELETE FROM customer WHERE id = %s"
        values = (customer_id,)
        self.db_cursor.execute(sql, values)
        self.db_connection.commit()
        messagebox.showinfo("Success", "Customer details deleted successfully.")

if __name__ == "__main__":
    login_root = tk.Tk()
    user_login = UserLogin(login_root)
    login_root.mainloop()
