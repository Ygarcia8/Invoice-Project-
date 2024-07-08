import tkinter as tk
from tkinter import ttk, messagebox
from fpdf import FPDF
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas


# Define the services dictionary with prices
services = {
    'Sewer Cleaning': [('Storm Flushing', 100), ('Sanitary Flushing', 50), ('FDC Flushing', 100), ('CB Cleaning', 150),
                       ('CB Lead Flushing', 90), ('RLBC Lead Flushing', 150), ('MH Cleaning', 375),
                       ('Service Line Flush', 450), ('Vc Cleaning', 500)],
    'CCTV Inspection': [('Storm', 120), ('Sanitary', 50), ('FDC', 50), ('CB Lead', 120), ('Lateral', 40),
                        ('RLCB Lead', 170), ('Service Line', 420), ('Subdrain', 470), ('Dye Test', 520),
                        ('Pole Camera', 570)],
    'Trenchless Repairs': [('Sg Repair', 130), ('Pipe Re-Rounding', 180), ('Linear Liner', 230), ('Tee Liner', 280),
                           ('Spot Repair', 330), ('Main Lateral Grout', 280), ('MH Grouting', 130),
                           ('Mandrile Test', 180), ('Air Test', 130), ('Smoke Test', 80)],
    'Extra Services': [('Soft Excavation', 340), ('Investigation', 190), ('Traffic Control', 140),
                       ('Water Box Repair', 290), ('Hidrovac Excavation', 440), ('Utility Locates', 290),
                       ('MH Parging Repair', 440), ('Catch Basin Parging', 490), ('Robotic Cutting', 540),
                       ('General Repair', 590)]
}



class InvoiceApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Invoice Generator")

        self.items = []
        # Canvas for header
        self.canvas = tk.Canvas(root, width=400, height=100)
        self.canvas.grid(row=0, column=0, columnspan=2)
        self.canvas.create_text(200, 20, text="Global Sewer Services",font=('Arial', 24, 'bold'))

        # Service selection
        self.service_var = tk.StringVar()
        self.service_combo = ttk.Combobox(root, textvariable=self.service_var)
        self.service_combo['values'] = list(services.keys())
        self.service_combo.grid(row=3, column=1, padx=10, pady=5)
        self.service_combo.bind("<<ComboboxSelected>>", self.update_items)

        # Item selection
        self.item_var = tk.StringVar()
        self.item_combo = ttk.Combobox(root, textvariable=self.item_var)
        self.item_combo.grid(row=4, column=1, padx=10, pady=5)

        # Hours input
        self.hours_var = tk.IntVar()
        self.hours_entry = tk.Entry(root, textvariable=self.hours_var)
        self.hours_entry.grid(row=5, column=1, padx=10, pady=5)

        # Company address input
        self.address_var = tk.StringVar()
        self.address_entry = tk.Entry(root, textvariable=self.address_var)
        self.address_entry.grid(row=0, column=1, padx=10, pady=5)

        # Invoice number input
        self.invoice_var = tk.StringVar()
        self.invoice_entry = tk.Entry(root, textvariable=self.invoice_var)
        self.invoice_entry.grid(row=1, column=1, padx=10, pady=5)

        # Date input
        self.date_var = tk.StringVar()
        self.date_entry = tk.Entry(root, textvariable=self.date_var)
        self.date_entry.grid(row=2, column=1, padx=10, pady=5)

        # Buttons
        tk.Button(root, text="Add Item", command=self.add_item).grid(row=6, column=1, padx=10, pady=5)
        tk.Button(root, text="Generate Invoice", command=self.generate_invoice).grid(row=7, column=1, padx=10, pady=5)

        # Listbox for items
        self.items_listbox = tk.Listbox(root)
        self.items_listbox.grid(row=8, column=1, padx=10, pady=5)

        # Labels
        tk.Label(root, text="Company Address:").grid(row=0, column=0, padx=10, pady=5)
        tk.Label(root, text="Invoice Number:").grid(row=1, column=0, padx=10, pady=5)
        tk.Label(root, text="Date:").grid(row=2, column=0, padx=10, pady=5)
        tk.Label(root, text="Select Service:").grid(row=3, column=0, padx=10, pady=5)
        tk.Label(root, text="Select Item:").grid(row=4, column=0, padx=10, pady=5)
        tk.Label(root, text="Enter Hours:").grid(row=5, column=0, padx=10, pady=5)


    def update_items(self, event):
        service = self.service_var.get()
        if service in services:
            items = [item[0] for item in services[service]]
            self.item_combo['values'] = items

    def add_item(self):
        service = self.service_var.get()
        item = self.item_var.get()
        hours = self.hours_var.get()

        if service and item and hours:
            price = next(price for name, price in services[service] if name == item)
            total = price * hours
            self.items.append((service, item, hours, total))
            self.items_listbox.insert(tk.END, f"{service} - {item} - {hours} hours - ${total:.2f}")
        else:
            messagebox.showwarning("Input Error", "Please fill all fields.")

    def generate_invoice(self):
        address = self.address_var.get()
        invoice_number = self.invoice_var.get()
        date = self.date_var.get()

        if not address or not invoice_number or not date:
            messagebox.showwarning("Input Error", "Please fill all fields.")
            return

        pdf = FPDF()
        pdf.add_page()
        pdf.set_font("Arial", size=16)

        pdf.cell(200, 10, txt="Global Sewer Services", ln=True, align='C')
        pdf.cell(200, 10, txt="69 Maplecrete Rd, Concord", ln=True, align='C')
        pdf.cell(200, 10, txt="Email: contact@globalsewerservices.com", ln=True, align='C')
        pdf.cell(200, 10, txt="Phone: (905-738-6704)", ln=True, align='C')
        pdf.cell(200, 10, txt="", ln=True, align='C')


        pdf.cell(200, 10, txt=f"Date: {date}", ln=True, align='L')
        pdf.cell(200, 10, txt=f"Invoice #: {invoice_number}", ln=True, align='L')
        pdf.cell(200, 10, txt=f"Address: {address}", ln=True, align='L')
        pdf.cell(200, 10, txt="", ln=True, align='L')

        pdf.cell(200, 10, txt="Services Rendered:", ln=True, align='L')
        pdf.cell(200, 10, txt="", ln=True, align='L')

        total_amount = 0
        for service, item, hours, total in self.items:
            pdf.cell(200, 10, txt=f"{service} - {item} - {hours} hours - ${total:.2f}", ln=True, align='L')
            total_amount += total

        pdf.cell(200, 10, txt="", ln=True, align='L')
        pdf.cell(200, 10, txt=f"Total Amount Due: ${total_amount:.2f}", ln=True, align='L')

        pdf.output("invoice.pdf")
        messagebox.showinfo("Invoice Generated", "Invoice has been generated and saved as 'invoice.pdf'.")

if __name__ == "__main__":
    root = tk.Tk()
    app = InvoiceApp(root)
    root.mainloop()
