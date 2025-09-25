# estadodecuenta
from datetime import date
import tkinter as tk

# ----- Clases base -----
class Cliente:
    def __init__(self, nombre, apellido1, apellido2, fecha_nac, domicilio):
        self.nombre = nombre
        self.apellido1 = apellido1
        self.apellido2 = apellido2
        self.fecha_nac = fecha_nac
        self.domicilio = domicilio

class Cuenta:
    def __init__(self, numero):
        self.numero = numero
        self.saldo = 0

    def abonar(self, cant):
        self.saldo += cant

    def cargar(self, cant):
        if self.saldo >= cant:
            self.saldo -= cant
            return True
        else:
            return False

class Movimiento:
    def __init__(self, fecha_mov, descripcion, cargo, abono, saldo):
        self.fecha_mov = fecha_mov
        self.descripcion = descripcion
        self.cargo = cargo
        self.abono = abono
        self.saldo = saldo

class EstadoCuenta:
    def __init__(self, cliente: Cliente, cuenta: Cuenta, fecha_ingreso: date, movimientos=None):
        self.cliente = cliente
        self.cuenta = cuenta
        self.fecha_ingreso = fecha_ingreso
        self.movimientos = movimientos if movimientos is not None else []

ventana = tk.Tk()
ventana.title("Estado de cuenta")
ventana.geometry("600x550")

tk.Label(ventana, text="Nombre:").pack()
entry_nombre = tk.Entry(ventana)
entry_nombre.pack()

tk.Label(ventana, text="Apellido paterno:").pack()
entry_apellido1 = tk.Entry(ventana)
entry_apellido1.pack()

tk.Label(ventana, text="Apellido materno:").pack()
entry_apellido2 = tk.Entry(ventana)
entry_apellido2.pack()

tk.Label(ventana, text="Fecha de nacimiento (ej. 2000-02-06):").pack()
entry_fecha = tk.Entry(ventana)
entry_fecha.pack()

tk.Label(ventana, text="Domicilio:").pack()
entry_domicilio = tk.Entry(ventana)
entry_domicilio.pack()

datos_cliente_label = tk.Label(ventana, text="", font=("Arial", 10))
datos_cliente_label.pack(pady=10)

saldo_label = tk.Label(ventana, text="", font=("Arial", 12))
saldo_label.pack()

tk.Label(ventana, text="Cantidad a abonar o cargar:").pack()
entry_cantidad = tk.Entry(ventana)
entry_cantidad.pack()

movimientos_text = tk.Text(ventana, height=10, width=70)
movimientos_text.pack(pady=10)

cliente = None
cuenta = None
estado = None

def registrar_cliente():
    global cliente, cuenta, estado

    nombre = entry_nombre.get()
    apellido1 = entry_apellido1.get()
    apellido2 = entry_apellido2.get()
    fecha_nac = entry_fecha.get()
    domicilio = entry_domicilio.get()

    if not all([nombre, apellido1, apellido2, fecha_nac, domicilio]):
        print("Faltan datos: Por favor llena todos los campos del cliente.")
        return

    cliente = Cliente(nombre, apellido1, apellido2, fecha_nac, domicilio)
    cuenta = Cuenta("123456")
    estado = EstadoCuenta(cliente, cuenta, date.today())

    datos_cliente_label.config(
        text=f"Cliente: {cliente.nombre} {cliente.apellido1} {cliente.apellido2}\n"
             f"Nacimiento: {cliente.fecha_nac} | Domicilio: {cliente.domicilio}"
    )
    saldo_label.config(text=f"Saldo actual: ${cuenta.saldo:.2f}")
    actualizar_movimientos()

def hacer_abono():
    if estado is None:
        print("Cliente no registrado: Primero registra los datos del cliente.")
        return

    try:
        cantidad = float(entry_cantidad.get())
        cuenta.abonar(cantidad)
        estado.movimientos.append(Movimiento(date.today(), "Abono", 0, cantidad, cuenta.saldo))
        saldo_label.config(text=f"Saldo actual: ${cuenta.saldo:.2f}")
        actualizar_movimientos()
    except ValueError:
        print("Error: Ingresa una cantidad válida.")

def hacer_cargo():
    if estado is None:
        print("Cliente no registrado: Primero registra los datos del cliente.")
        return

    try:
        cantidad = float(entry_cantidad.get())
        if cuenta.cargar(cantidad):
            estado.movimientos.append(Movimiento(date.today(), "Cargo", cantidad, 0, cuenta.saldo))
            saldo_label.config(text=f"Saldo actual: ${cuenta.saldo:.2f}")
        else:
            print("Saldo insuficiente: No hay suficiente saldo.")
        actualizar_movimientos()
    except ValueError:
        print("Error: Ingresa una cantidad válida.")

def actualizar_movimientos():
    movimientos_text.delete("1.0", tk.END)
    if estado is None:
        return
    for mov in estado.movimientos:
        linea = f"{mov.fecha_mov} - {mov.descripcion}: Cargo ${mov.cargo:.2f}, Abono ${mov.abono:.2f}, Saldo: ${mov.saldo:.2f}\n"
        movimientos_text.insert(tk.END, linea)

tk.Button(ventana, text="Registrar cliente", command=registrar_cliente).pack()
tk.Button(ventana, text="Abonar", command=hacer_abono).pack()
tk.Button(ventana, text="Cargar", command=hacer_cargo).pack()

ventana.mainloop()
