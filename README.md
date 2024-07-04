import tkinter
import tkinter as tk
from tkinter import ttk
from tkinter import messagebox
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas



def create_invoice(invoice_number, invoice_date, due_date, bill_to, items, tax_rate):
    c = canvas.Canvas(f"invoice_{invoice_number}.pdf", pagesize=letter)
    width, height = letter

    # Company Information
    c.setFont("Helvetica-Bold", 20)
    c.drawString(50, 750, "Global Sewer Services")
    c.setFont("Helvetica", 10)
    c.drawString(50, 735, "69 Maplecrete Rd, Concord")
    c.drawString(50, 725, "Vaungh,Ontario, L4K 1E5")
    c.drawString(50, 715, "info@globalsewer.com")
    c.drawString(50, 705, "Phone: 905-738-6704")


    # Invoice Information
    c.setFont("Helvetica-Bold", 16)
    c.drawString(400, 750, "Invoice")
    c.setFont("Helvetica", 10)
    c.drawString(400, 735, f"Invoice Number: {invoice_number}")
    c.drawString(400, 720, f"Invoice Date: {invoice_date}")
    c.drawString(400, 705, f"Due Date: {due_date}")

    # Bill To Information
    c.setFont("Helvetica-Bold", 12)
    c.drawString(50, 680, "Bill To:")
    c.setFont("Helvetica", 10)
    y_position = 665
    for line in bill_to.split('\n'):
        c.drawString(50, y_position, line)
        y_position -= 15

    # Table Header
    c.setFont("Helvetica-Bold", 10)
    c.drawString(50, 630, "Description")
    c.drawString(250, 630, "Quantity")
    c.drawString(350, 630, "Unit Price")
    c.drawString(450, 630, "Total")

    # Table Content
    c.setFont("Helvetica", 10)
    y_position = 615
    subtotal = 0
    for item in items:
        description, quantity, unit_price = item
        total = quantity * unit_price
        subtotal += total
        c.drawString(50, y_position, description)
        c.drawString(250, y_position, str(quantity))
        c.drawString(350, y_position, f"${unit_price:.2f}")
        c.drawString(450, y_position, f"${total:.2f}")
        y_position -= 15

    # Subtotal, Tax, Total
    tax = subtotal * tax_rate / 100
    total_due = subtotal + tax

    c.drawString(350, y_position, "Subtotal")
    c.drawString(450, y_position, f"${subtotal:.2f}")
    y_position -= 15
    c.drawString(350, y_position, f"Tax ({tax_rate}%)")
    c.drawString(450, y_position, f"${tax:.2f}")
    y_position -= 15
    c.setFont("Helvetica-Bold", 10)
    c.drawString(350, y_position, "Total Due")
    c.drawString(450, y_position, f"${total_due:.2f}")

    # Save PDF
    c.save()
def generate_invoice():
    invoice_number = entry_invoice_number.get()
    invoice_date = entry_invoice_date.get()
    due_date = entry_due_date.get()
    bill_to = entry_bill_to.get("1.0", tk.END).strip()

    try:
        tax_rate = float(entry_tax_rate.get())
    except ValueError:
        messagebox.showerror("Invalid Input", "Tax rate must be a number.")
        return

    items = []
    for item in items:
        description = item[0].get()
        try:
            quantity = int(item[1].get())
            unit_price = float(item[2].get())
        except ValueError:
            messagebox.showerror("Invalid Input", "Quantity must be an integer and Unit Price must be a number.")
            return
        items.append((description, quantity, unit_price))

    create_invoice(invoice_number, invoice_date, due_date, bill_to, items, tax_rate)
    messagebox.showinfo("Success", "Invoice generated successfully.")


app = tk.Tk()
app.title("Invoice Global Sewer")

tk.Label(app, text="Invoice Number").grid(row=0, column=0)
entry_invoice_number = tk.Entry(app)
entry_invoice_number.grid(row=0, column=1)

tk.Label(app, text="Invoice Date").grid(row=1, column=0)
entry_invoice_date = tk.Entry(app)
entry_invoice_date.grid(row=1, column=1)

tk.Label(app, text="Due Date").grid(row=2, column=0)
entry_due_date = tk.Entry(app)
entry_due_date.grid(row=2, column=1)

#add sub categories  Adreess pc and city
tk.Label(app, text="Bill To").grid(row=3, column=0)
entry_bill_to = tk.Text(app, height=5, width=30)
entry_bill_to.grid(row=3, column=1)

tk.Label(app, text="Tax Rate (%)").grid(row=4, column=0)
entry_tax_rate = tk.Entry(app)
entry_tax_rate.grid(row=4, column=1)

services={
        'sewer cleaning':['storm flushing','sanitary flushing','FDC Flushing','CB Cleaning','CB lead Flushing','RLBC Lead Flushing','MH Cleaning','Service Line Flush','Vc Cleaning'],
        'CCTV Inspection':['Storm','sanitary','FDC','CB Lead','Lateral','RLCB Lead','Service Line','Subdrain','Dye Test','Pole Camara'],
        'Trenchless Repairs':['Sg Repair','Pipe Re-Ruonding','Linear  Liner','Tee Liner','Spot Repair','Main lateral Grout','MH Grouting','Mandrile Test','Air Test','Smoke test'],
        'Extra Services':['Soft Excavation','Investigation','Traffic Control','Water box Repair','Hidrovac Excavation','Utility Locates','MH Parging Repair','Cach Basin Parging','Robotic Curtting','General Repair'],
    }
tk.Label(app, text="Description").grid(row=5, column=0)

frame = ttk.Frame(app, padding="10")
frame.grid(row=6, column=0, sticky=(tk.W, tk.E, tk.N, tk.S))

# Function to update combobox values based on the selected service category
def update_combobox(serv):
    selected_service = service_category.get()
    serv['values'] = services[selected_service]
    serv.set('')

# Create a combobox for service categories
service_category_label = ttk.Label(frame, text="Select Service Category:")
service_category_label.grid(row=6, column=0)
service_category = ttk.Combobox(frame, values=list(services.keys()))
service_category.grid(row=6, column=1)
service_category.bind("<<ComboboxSelected>>", update_combobox)

frame = ttk.Frame(frame, padding="10")
frame.grid(row=0, column=0, sticky=(tk.W, tk.E, tk.N, tk.S))

# Create a frame for checkboxes
checkbox_frame = ttk.Frame(frame)
checkbox_frame.grid(row=2, column=0, columnspan=2, sticky=(tk.W, tk.E))


# Function to update checkboxes based on the selected service category
def update_checkboxes(event):
    # Clear existing checkboxes
    for widget in checkbox_frame.winfo_children():
        widget.destroy()

    selected_service = service_category.get()
    items = services[selected_service]

    # Create checkboxes for each item
    for item in items:
        var = tk.BooleanVar()
        chk = ttk.Checkbutton(checkbox_frame, text=item, variable=var)
        chk.var = var
        chk.pack(anchor=tk.W)
        checkboxes[item] = var


# Create a combobox for service categories
service_category_label = ttk.Label(frame, text="Select Service Category:")
service_category_label.grid(row=0, column=0, pady=5, sticky=tk.W)
service_category = ttk.Combobox(frame, values=list(services.keys()))
service_category.grid(row=0, column=1, pady=5, sticky=(tk.W, tk.E))
service_category.bind("<<ComboboxSelected>>", update_checkboxes)

# Dictionary to hold the state of each checkbox
checkboxes = {}

# Description label


# Function to update checkboxes based on the selected service category

tk.Label(app, text="Quantity (hrs)").grid(row=5, column=0, columnspan=3)
tk.Label(app, text="Unit Price").grid(row=5, column=2, columnspan=1)


tk.Button(app, text="Generate Invoice", command=generate_invoice).grid(row=9, column=3, columnspan=3)

app.mainloop()
