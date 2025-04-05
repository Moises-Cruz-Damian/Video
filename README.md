import tkinter as tk
from tkinter import messagebox
import sympy as sp
import numpy as np
import matplotlib.pyplot as plt


x = sp.Symbol('x')

def newton_raphson(f_expr, x0, tol=1e-6, max_iter=100):
    f = sp.lambdify(x, f_expr, 'math')
    f_prime = sp.lambdify(x, sp.diff(f_expr, x), 'math')

    x_values = [x0]
    errors = []
    for i in range(max_iter):
        fx = f(x0)
        fpx = f_prime(x0)
        if fpx == 0:
            return None, [], [], i + 1
        x1 = x0 - fx / fpx
        x_values.append(x1)
        errors.append(abs(x1 - x0))
        if abs(x1 - x0) < tol:
            return round(x1, 5), [round(val, 5) for val in x_values], [round(err, 5) for err in errors], i + 1
        x0 = x1
    return None, x_values, errors, max_iter


def secante(f_expr, x0, x1, tol=1e-6, max_iter=100):
    f = sp.lambdify(x, f_expr, 'math')

    x_values = [x0, x1]
    errors = []
    for i in range(max_iter):
        fx0 = f(x0)
        fx1 = f(x1)
        if fx1 - fx0 == 0:
            return None, [], [], i + 1
        x2 = x1 - fx1 * (x1 - x0) / (fx1 - fx0)
        x_values.append(x2)
        errors.append(abs(x2 - x1))
        if abs(x2 - x1) < tol:
            return round(x2, 5), [round(val, 5) for val in x_values], [round(err, 5) for err in errors], i + 1
        x0, x1 = x1, x2
    return None, x_values, errors, max_iter


def plot_convergence(x_values, errors, method_name):
    
    plt.figure(figsize=(10, 6))
    plt.subplot(1, 2, 1)
    plt.plot(range(len(x_values)), x_values, marker='o', color='b')
    plt.title(f"Convergencia del valor de la raíz ({method_name})")
    plt.xlabel("Iteraciones")
    plt.ylabel("Valor de la raíz")

    
    plt.subplot(1, 2, 2)
    plt.plot(range(len(errors)), errors, marker='x', color='r')
    plt.title(f"Convergencia del error ({method_name})")
    plt.xlabel("Iteraciones")
    plt.ylabel("Error")

    plt.tight_layout()
    plt.show()


def calcular():
    fx_str = entry_func.get()
    try:
        
        f_expr = sp.sympify(fx_str)
        x0 = float(entry_x0.get())
        if method_var.get() == "Newton-Raphson":
            result, x_values, errors, iter_count = newton_raphson(f_expr, x0)
            method_name = "Newton-Raphson"
        elif method_var.get() == "Secante":
            x1 = float(entry_x1.get())
            result, x_values, errors, iter_count = secante(f_expr, x0, x1)
            method_name = "Secante"

        if result is None:
            messagebox.showwarning("Error", "No se pudo calcular la raíz con los valores dados.")
        else:
            result_label.config(text=f"Raíz encontrada: {result}")
            iteration_label.config(text=f"Iteraciones: {iter_count}")
            plot_convergence(x_values, errors, method_name)
    except Exception as e:
        messagebox.showerror("Error", f"Error en los datos: {e}")


root = tk.Tk()
root.title("Métodos de Secante y Newton-Raphson")

method_var = tk.StringVar(value="Secante")


tk.Label(root, text="Ingrese la función f(x):").pack()
entry_func = tk.Entry(root, width=50)
entry_func.pack()


tk.Label(root, text="Seleccione el método:").pack()
tk.Radiobutton(root, text="Newton-Raphson", variable=method_var, value="Newton-Raphson").pack()
tk.Radiobutton(root, text="Secante", variable=method_var, value="Secante").pack()


tk.Label(root, text="Ingrese el valor inicial x0:").pack()
entry_x0 = tk.Entry(root)
entry_x0.pack()

tk.Label(root, text="Ingrese el valor x1 (solo si es Secante):").pack()
entry_x1 = tk.Entry(root)
entry_x1.pack()


tk.Button(root, text="Calcular", command=calcular).pack()


result_label = tk.Label(root, text="Raíz encontrada: ")
result_label.pack()


iteration_label = tk.Label(root, text="Iteraciones: ")
iteration_label.pack()

root.mainloop()
