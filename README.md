import os
import random
import datetime
import shutil
import json
import copy
import tkinter as tk
from tkinter import ttk, messagebox, simpledialog, filedialog
from PIL import Image, ImageTk

# ========================= ESTILO TELEMÁTICO =========================
# Paleta de colores (tonos azules y grises)
COLOR_FONDO       = "#ECF4FB"   # Fondo general: azul grisáceo claro
COLOR_BOTON       = "#1F3A93"   # Azul oscuro para botones
COLOR_BOTON_HOVER = "#2E86C1"   # Azul intermedio para hover
COLOR_TEXTO       = "#FFFFFF"   # Texto blanco en botones
COLOR_SECCION_BG  = "#E1ECF7"   # Fondo de menús internos (azul muy claro)
COLOR_BORDES      = "#B3CDE0"   # Bordes suaves

# Colores heredados
FG_CLARO          = "black"
ENTRY_BG          = "white"
ENTRY_FG          = "black"
BTN_CONTINUAR_COLOR  = "#61E25F"  # Se conserva este color para algunos botones (verde)
BTN_REINICIAR_COLOR  = "#FF6B6B"  # Para reiniciar (rojo)

# Modo Docente / Botones en menús
ADMIN_BG          = "#F7F7F7"
ADMIN_FG          = "#333333"
ADMIN_BTN_COLOR   = COLOR_BOTON
ADMIN_REINICIAR_COLOR = "#D9534F"

# Fuentes y dimensiones
FONT_FAMILY   = "Segoe UI"   # Fuente sans-serif moderna
FONT_TITLE    = (FONT_FAMILY, 18, "bold")
FONT_SUBTITLE = (FONT_FAMILY, 20)
FONT_NORMAL   = (FONT_FAMILY, 12)

WINDOW_WIDTH  = 600
WINDOW_HEIGHT = 450
TIEMPO_INICIAL= 3600  # 1 hora en segundos

# ========================= BASE DE PREGUNTAS =========================
# Se conserva la estructura y textos originales
modulos = {
    "Variables": [
        {
            "pregunta": "¿Cuál de las siguientes opciones describe mejor las variables en Python?",
            "respuestas": ["Pueden cambiar de tipo", "Son siempre del mismo tipo", "Deben ser declaradas antes de usarse"],
            "correcta": 0,
            "explicacion": "En Python, las variables son dinámicamente tipadas, lo que significa que pueden cambiar de tipo."
        },
        {
            "pregunta": "¿Qué se requiere para declarar una variable en Python?",
            "respuestas": ["Declarar su tipo", "Asignarle un valor", "No se requiere nada"],
            "correcta": 1,
            "explicacion": "No es necesario declarar el tipo de una variable en Python, ya que es un lenguaje de tipado dinámico."
        },
        {
            "pregunta": "¿Qué ocurre si intentas usar una variable sin asignarle un valor primero?",
            "respuestas": ["Genera un error", "Devuelve un valor por defecto", "Funciona normalmente"],
            "correcta": 0,
            "explicacion": "No puedes usar una variable antes de asignarle un valor, ya que generará un error."
        },
        {
            "pregunta": "¿Qué signo se utiliza para asignar un valor a una variable?",
            "respuestas": ["=", "==", "=!", "=>"],
            "correcta": 0,
            "explicacion": "El signo igual (=) se utiliza para asignar un valor a una variable."
        },
        {
            "pregunta": "¿Cómo se comportan las variables en relación a las mayúsculas y minúsculas?",
            "respuestas": ["Son sensibles a mayúsculas y minúsculas", "No son sensibles", "Solo en algunos casos"],
            "correcta": 0,
            "explicacion": "En Python, 'Variable' y 'variable' son considerados nombres diferentes."
        },
        {
            "pregunta": "¿Cuál es la forma correcta de declarar múltiples variables en una sola línea?",
            "respuestas": ["Usando comas", "Usando punto y coma", "No es posible"],
            "correcta": 0,
            "explicacion": "Puedes declarar múltiples variables en una sola línea usando comas, por ejemplo: a, b = 1, 2."
        }
    ],
    "Datos Compuestos": [
        {
            "pregunta": "¿Qué tipo de datos pueden contener las listas en Python?",
            "respuestas": ["Solo enteros", "Solo cadenas", "Diferentes tipos de datos"],
            "correcta": 2,
            "explicacion": "Las listas en Python pueden contener elementos de diferentes tipos de datos."
        },
        {
            "pregunta": "¿Cómo se comportan los diccionarios en Python en términos de orden?",
            "respuestas": ["Siempre son ordenados", "Nunca son ordenados", "Depende de la versión de Python"],
            "correcta": 0,
            "explicacion": "Desde Python 3.7, los diccionarios mantienen el orden de inserción de los elementos."
        },
        {
            "pregunta": "¿Cuál es la naturaleza de las tuplas en Python?",
            "respuestas": ["Son mutables", "Son inmutables", "Pueden ser mutadas en ciertas condiciones"],
            "correcta": 1,
            "explicacion": "Las tuplas son inmutables, lo que significa que no puedes cambiar sus elementos una vez creadas."
        },
        {
            "pregunta": "¿Cómo se accede a un elemento de una lista?",
            "respuestas": ["Usando índices", "Usando llaves", "Usando paréntesis"],
            "correcta": 0,
            "explicacion": "Puedes acceder a un elemento de una lista usando su índice, por ejemplo: lista[0]."
        },
        {
            "pregunta": "¿Qué método se utiliza para agregar un elemento a una lista?",
            "respuestas": ["append()", "add()", "insert()"],
            "correcta": 0,
            "explicacion": "El método append() se utiliza para agregar un elemento al final de una lista."
        },
        {
            "pregunta": "¿Cómo se define un diccionario en Python?",
            "respuestas": ["Con corchetes", "Con llaves", "Con paréntesis"],
            "correcta": 1,
            "explicacion": "Un diccionario se define usando llaves ({}), no corchetes."
        }
    ],
    # (Se agregarían los demás módulos de Operadores, Funciones, Estructuras de Control, Módulos y Paquetes, Examen Final, etc.
}

default_modulos = copy.deepcopy(modulos)

# ========================= CUESTIONARIOS PERSONALIZADOS =========================
custom_cuestionarios = {}

def cargar_cuestionarios_custom():
    if os.path.exists("cuestionarios_custom.json"):
        try:
            with open("cuestionarios_custom.json", "r", encoding="utf-8") as f:
                data = json.load(f)
                custom_cuestionarios.update(data)
        except Exception as e:
            messagebox.showerror("Error", f"No se pudo cargar los cuestionarios personalizados: {e}")

def guardar_cuestionarios_custom():
    try:
        with open("cuestionarios_custom.json", "w", encoding="utf-8") as f:
            json.dump(custom_cuestionarios, f, indent=4)
    except Exception as e:
        messagebox.showerror("Error", f"No se pudo guardar los cuestionarios personalizados: {e}")

# ========================= CLASE PRINCIPAL =========================
class AplicacionCuestionario:
    def __init__(self, master):
        self.master = master
        self.master.title("LearnPy © 1.1")
        self.master.config(bg=COLOR_FONDO)
        self.centrar_ventana(WINDOW_WIDTH, WINDOW_HEIGHT)

        # Variables de estado
        self.usuario = {"nombre": "", "año": "", "Mencion": "Telemática", "sección": ""}
        self.timer_mode = False
        self.admin_mode = False
        self.custom_module_enabled = False
        self.custom_questionnaire_enabled = False
        self.tiempo_inicial = TIEMPO_INICIAL  # Duración en segundos
        self.remaining_time = self.tiempo_inicial
        self.skip_enabled = False
        self.menu_open = None
        self.rotating = False

        # Contadores
        self.total_preguntas = sum(len(modulos[m]) for m in modulos)
        self.pregunta_global = 0
        self.respuestas_usuario = []
        self.is_custom = False
        self.cuestionario_personalizado = None

        # Cargar datos
        self.cargar_modulos_persistentes()
        cargar_cuestionarios_custom()

        # Estilo para Combobox
        self.style = ttk.Style()
        self.style.theme_use('clam')
        self.style.configure("TCombobox",
                             fieldbackground=COLOR_FONDO,
                             background=COLOR_FONDO,
                             foreground="black",
                             font=(FONT_FAMILY, 11))

        self.menu_registro()

    # --------------------- FUNCIONES DE POSICIONAMIENTO Y ESTILO ---------------------
    def centrar_ventana(self, ancho, alto):
        self.master.update_idletasks()
        ws = self.master.winfo_screenwidth()
        hs = self.master.winfo_screenheight()
        x = (ws // 2) - (ancho // 2)
        y = (hs // 2) - (alto // 2)
        self.master.geometry(f"{ancho}x{alto}+{x}+{y}")

    def crear_label(self, master, text, font=FONT_NORMAL, **options):
        options.setdefault("bg", COLOR_FONDO)
        options.setdefault("fg", "black")
        return tk.Label(master, text=text, font=font, **options)

    # --------------------- FUNCIONES DE CARGA/ALMACENAMIENTO ---------------------
    def cargar_modulos_persistentes(self):
        if os.path.exists("modulos.json"):
            try:
                with open("modulos.json", "r", encoding="utf-8") as f:
                    data = json.load(f)
                global modulos
                modulos = data
            except Exception as e:
                messagebox.showerror("Error", f"No se pudieron cargar los módulos persistentes: {e}")

    def guardar_modulos_persistentes(self):
        try:
            with open("modulos.json", "w", encoding="utf-8") as f:
                json.dump(modulos, f, indent=4)
        except Exception as e:
            messagebox.showerror("Error", f"No se pudieron guardar los módulos: {e}")

    def animate_progress(self, target_value):
        current_value = self.progress['value']
        if current_value < target_value:
            self.progress['value'] = current_value + 1
            self.master.after(20, lambda: self.animate_progress(target_value))
        elif current_value > target_value:
            self.progress['value'] = current_value - 1
            self.master.after(20, lambda: self.animate_progress(target_value))

    # --------------------- EFECTOS DE HOVER EN BOTONES ---------------------
    def on_enter_boton(self, e):
        e.widget['bg'] = COLOR_BOTON_HOVER

    def on_leave_boton(self, e):
        e.widget['bg'] = COLOR_BOTON

    # --------------------- MENÚ DE REGISTRO ---------------------
    def menu_registro(self, registro_data=None):
        if registro_data is None:
            registro_data = {}
        for widget in self.master.winfo_children():
            widget.destroy()
        self.master.config(bg=COLOR_FONDO)
        frame = tk.Frame(self.master, bg=COLOR_FONDO)
        frame.pack(expand=True, fill="both", padx=20, pady=20)

        lbl_titulo = self.crear_label(frame, "Registro de Usuario", font=(FONT_FAMILY, 20, "bold"))
        lbl_titulo.grid(row=0, column=0, columnspan=2, pady=10)

        self.crear_label(frame, "Nombre:", font=FONT_NORMAL).grid(row=1, column=0, sticky="e", padx=5, pady=5)
        self.entry_nombre = tk.Entry(frame, font=FONT_NORMAL, width=30, bg=ENTRY_BG, fg=ENTRY_FG)
        self.entry_nombre.grid(row=1, column=1, sticky="w", padx=5, pady=5)
        if registro_data.get("nombre"):
            self.entry_nombre.insert(0, registro_data["nombre"])

        self.crear_label(frame, "Año:", font=FONT_NORMAL).grid(row=2, column=0, sticky="e", padx=5, pady=5)
        vcmd = (frame.register(lambda P: P.isdigit() or P == ""), '%P')
        self.entry_año = tk.Entry(frame, font=FONT_NORMAL, width=10, bg=ENTRY_BG, fg=ENTRY_FG,
                                  validate="key", validatecommand=vcmd)
        self.entry_año.grid(row=2, column=1, sticky="w", padx=5, pady=5)
        if registro_data.get("año"):
            self.entry_año.insert(0, registro_data["año"])

        self.crear_label(frame, "Mención:", font=FONT_NORMAL).grid(row=3, column=0, sticky="e", padx=5, pady=5)
        self.entry_especialidad = tk.Entry(frame, font=FONT_NORMAL, width=20, bg=ENTRY_BG, fg=ENTRY_FG)
        self.entry_especialidad.insert(0, "Telemática")
        self.entry_especialidad.config(state="readonly")
        self.entry_especialidad.grid(row=3, column=1, sticky="w", padx=5, pady=5)

        self.crear_label(frame, "Sección:", font=FONT_NORMAL).grid(row=4, column=0, sticky="e", padx=5, pady=5)
        self.combo_seccion = ttk.Combobox(frame, values=["A", "B"], font=FONT_NORMAL, state="readonly", width=8)
        self.combo_seccion.grid(row=4, column=1, sticky="w", padx=5, pady=5)
        if registro_data.get("sección"):
            self.combo_seccion.set(registro_data["sección"])
        else:
            self.combo_seccion.current(0)

        btn_continuar = tk.Button(frame, text="Continuar", command=self.guardar_usuario,
                                  font=(FONT_FAMILY, 14, "bold"), bg=COLOR_BOTON, fg=COLOR_TEXTO)
        btn_continuar.grid(row=5, column=0, columnspan=2, pady=20)
        btn_continuar.bind("<Enter>", self.on_enter_boton)
        btn_continuar.bind("<Leave>", self.on_leave_boton)

        self.agregar_boton_configuracion()

    def obtener_datos_registro(self):
        return {
            "nombre": self.entry_nombre.get().strip(),
            "año": self.entry_año.get().strip(),
            "sección": self.combo_seccion.get()
        }

    def guardar_usuario(self):
        nombre = self.entry_nombre.get().strip()
        año = self.entry_año.get().strip()
        seccion = self.combo_seccion.get()
        if not nombre or not año:
            messagebox.showerror("Error", "Complete todos los campos.")
            return
        self.usuario["nombre"] = nombre
        self.usuario["año"] = año
        self.usuario["Mencion"] = self.entry_especialidad.get()
        self.usuario["sección"] = seccion
        self.menu_inicio()

    # --------------------- MENÚ PRINCIPAL ---------------------
    def menu_inicio(self):
        for widget in self.master.winfo_children():
            widget.destroy()
        self.master.config(bg=COLOR_FONDO)
        container = tk.Frame(self.master, bg=COLOR_FONDO)
        container.pack(expand=True, fill="both")
        for i in range(6):
            container.grid_rowconfigure(i, weight=1)
        for i in range(3):
            container.grid_columnconfigure(i, weight=1)

        # Carga de íconos (si existen)
        try:
            ruta_logo = os.path.join(os.path.dirname(os.path.abspath(__file__)), "Imagenes", "LogoTelematica.png")
            imagen_logo = Image.open(ruta_logo).convert("RGBA")
            imagen_logo = imagen_logo.resize((145, 110), Image.Resampling.LANCZOS)
            self.imagen_logo = ImageTk.PhotoImage(imagen_logo)
            tk.Label(container, image=self.imagen_logo, bg=COLOR_FONDO, bd=0).grid(row=0, column=0, sticky="w", padx=10, pady=10)
        except Exception:
            pass

        lbl_titulo = self.crear_label(container, "Bienvenido a LearnPy", font=(FONT_FAMILY, 18, "bold"), bg=COLOR_FONDO)
        lbl_titulo.grid(row=0, column=1, pady=10)

        try:
            ruta_telematica = os.path.join(os.path.dirname(os.path.abspath(__file__)), "Imagenes", "icon_telematica.png")
            imagen_tele = Image.open(ruta_telematica).convert("RGBA")
            imagen_tele = imagen_tele.resize((140, 110), Image.Resampling.LANCZOS)
            self.imagen_telematica = ImageTk.PhotoImage(imagen_tele)
            tk.Label(container, image=self.imagen_telematica, bg=COLOR_FONDO, bd=0).grid(row=0, column=2, sticky="e", padx=10, pady=10)
        except Exception:
            pass

        info_usuario = f"Usuario: {self.usuario['nombre']} | Año: {self.usuario['año']} | Mención: {self.usuario['Mencion']} | Sección: {self.usuario['sección']}"
        lbl_info = self.crear_label(container, info_usuario, font=(FONT_FAMILY, 12, "bold"), bg=COLOR_FONDO)
        lbl_info.grid(row=1, column=0, columnspan=3, padx=10, pady=(0,10), sticky="w")

        descripcion = ("Desarrollo de aplicación en programación básica dirigida a los estudiantes de 4to Año de la mención Telemática "
                       "en la Escuela Técnica Monseñor Gregorio Adams")
        lbl_desc = self.crear_label(container, descripcion, font=(FONT_FAMILY, 15), wraplength=550, bg=COLOR_FONDO)
        lbl_desc.grid(row=2, column=0, columnspan=3, padx=10, pady=10, sticky="w")

        btn_empezar = tk.Button(container, text="Empezar Prueba",
                                command=self.seleccionar_tipo_cuestionario,
                                font=(FONT_FAMILY, 16, "bold"),
                                bg=COLOR_BOTON, fg=COLOR_TEXTO)
        btn_empezar.grid(row=3, column=0, columnspan=3, pady=20)
        btn_empezar.bind("<Enter>", self.on_enter_boton)
        btn_empezar.bind("<Leave>", self.on_leave_boton)

        autores_text = "Autores: Brahyam Velasquez | Jose Gomes | Jose Pereira | Samuel Meneses"
        lbl_autores = self.crear_label(container, autores_text, font=(FONT_FAMILY, 10, "bold"), bg=COLOR_FONDO)
        lbl_autores.grid(row=4, column=0, columnspan=3, padx=10, pady=10, sticky="w")

        btn_historial = tk.Button(container, text="Ver Historial", command=self.acceder_historial,
                                  font=(FONT_FAMILY, 12, "bold"), bg="#CCCCCC", fg="black")
        btn_historial.grid(row=5, column=0, columnspan=3, pady=10)

        btn_sugerencia = tk.Button(container, text="💡 Enviar Sugerencia", command=self.abrir_ventana_sugerencia,
                                   font=(FONT_FAMILY, 12, "bold"), bg=COLOR_BOTON, fg=COLOR_TEXTO)
        btn_sugerencia.grid(row=5, column=2, pady=10)
        btn_sugerencia.bind("<Enter>", self.on_enter_boton)
        btn_sugerencia.bind("<Leave>", self.on_leave_boton)

        self.agregar_boton_configuracion()

    # --------------------- MENÚ DE CONFIGURACIÓN ---------------------
    def agregar_boton_configuracion(self):
        try:
            ruta_settings = os.path.join(os.path.dirname(os.path.abspath(__file__)), "Imagenes", "icon_config.png")
            imagen = Image.open(ruta_settings).convert("RGBA")
            imagen = imagen.resize((70, 70), Image.Resampling.LANCZOS)
            self.original_settings_image = imagen.copy()
            self.imagen_settings = ImageTk.PhotoImage(imagen)
        except Exception:
            self.imagen_settings = None

        self.btn_config = tk.Button(self.master, image=self.imagen_settings, command=self.config_button_click,
                                    borderwidth=0, relief="flat", bg=COLOR_FONDO, activebackground=COLOR_FONDO)
        self.btn_config.place(x=10, y=10, anchor="nw")

    def config_button_click(self):
        if self.rotating:
            return
        if self.menu_open is not None and self.menu_open.winfo_exists():
            self.menu_open.destroy()
            self.menu_open = None
            return
        self.rotate_image(0)
        self.mostrar_menu_configuracion()

    def rotate_image(self, angle):
        self.rotating = True
        if angle < 360:
            rotated = self.original_settings_image.rotate(angle)
            self.imagen_settings = ImageTk.PhotoImage(rotated)
            self.btn_config.config(image=self.imagen_settings)
            self.master.after(10, lambda: self.rotate_image(angle + 10))
        else:
            self.imagen_settings = ImageTk.PhotoImage(self.original_settings_image)
            self.btn_config.config(image=self.imagen_settings)
            self.rotating = False

    def mostrar_menu_configuracion(self):
        menu_win = tk.Toplevel(self.master)
        menu_win.overrideredirect(True)
        menu_win.config(bg=COLOR_SECCION_BG)
        x = self.btn_config.winfo_rootx()
        y = self.btn_config.winfo_rooty() + self.btn_config.winfo_height()
        menu_win.geometry(f"+{x}+{y}")
        self.menu_open = menu_win

        frame = tk.Frame(menu_win, bg=COLOR_SECCION_BG)
        frame.pack(padx=10, pady=10)

        # 1. Modo Docente
        btn_docente = tk.Button(frame, text="👨‍🏫 Modo Docente",
                                font=(FONT_FAMILY, 12, "bold"),
                                bg=COLOR_BOTON, fg=COLOR_TEXTO,
                                relief="ridge", borderwidth=2,
                                command=lambda: [self.acceder_admin(), menu_win.destroy(), setattr(self, 'menu_open', None)])
        btn_docente.pack(fill="x", pady=5)
        btn_docente.bind("<Enter>", self.on_enter_boton)
        btn_docente.bind("<Leave>", self.on_leave_boton)

        # 2. Tutorial e Información
        btn_tutorial = tk.Button(frame, text="📖 Tutorial e Información",
                                 font=(FONT_FAMILY, 12, "bold"),
                                 bg=COLOR_BOTON, fg=COLOR_TEXTO,
                                 relief="ridge", borderwidth=2,
                                 command=lambda: [self.mostrar_tutorial_info(), menu_win.destroy(), setattr(self, 'menu_open', None)])
        btn_tutorial.pack(fill="x", pady=5)
        btn_tutorial.bind("<Enter>", self.on_enter_boton)
        btn_tutorial.bind("<Leave>", self.on_leave_boton)

        # 3. Activar/Desactivar Temporizador
        timer_text = "⏰ Activar Temporizador" if not self.timer_mode else "⏰ Desactivar Temporizador"
        btn_timer = tk.Button(frame, text=timer_text,
                              font=(FONT_FAMILY, 12, "bold"),
                              bg=COLOR_BOTON, fg=COLOR_TEXTO,
                              relief="ridge", borderwidth=2,
                              command=lambda: [self.toggle_timer_from_menu(), menu_win.destroy(), setattr(self, 'menu_open', None)])
        btn_timer.pack(fill="x", pady=5)
        btn_timer.bind("<Enter>", self.on_enter_boton)
        btn_timer.bind("<Leave>", self.on_leave_boton)

        # 4. Enviar Sugerencias
        btn_sugerencia = tk.Button(frame, text="💡 Enviar Sugerencias",
                                   font=(FONT_FAMILY, 12, "bold"),
                                   bg=COLOR_BOTON, fg=COLOR_TEXTO,
                                   relief="ridge", borderwidth=2,
                                   command=lambda: [self.abrir_ventana_sugerencia(), menu_win.destroy(), setattr(self, 'menu_open', None)])
        btn_sugerencia.pack(fill="x", pady=5)
        btn_sugerencia.bind("<Enter>", self.on_enter_boton)
        btn_sugerencia.bind("<Leave>", self.on_leave_boton)

        menu_win.focus_force()

    def toggle_timer_from_menu(self):
        self.timer_mode = not self.timer_mode
        if self.timer_mode:
            messagebox.showinfo("Temporizador",
                                f"Temporizador activado.\nDuración: {self.tiempo_inicial//60} minutos.\nEl temporizador limita el tiempo total de la prueba.")
        else:
            messagebox.showinfo("Temporizador", "Temporizador desactivado. La prueba ya no tendrá límite de tiempo.")

    # --------------------- MODO DOCENTE ---------------------
    def acceder_admin(self):
        password = simpledialog.askstring("Modo Docente", "Ingrese la clave de acceso (solo docentes):", show='*', parent=self.master)
        if password == "Adminsitogratis123-++*":
            self.admin_mode = True
            self.modo_administrador()
        else:
            messagebox.showerror("Acceso Denegado", "Clave incorrecta.")

    def modo_administrador(self):
        admin_window = tk.Toplevel(self.master)
        admin_window.title("Modo Docente - LearnPy")
        admin_window.config(bg=ADMIN_BG)
        self.centrar_ventana(admin_window, 600, 600)

        notebook = ttk.Notebook(admin_window)
        notebook.pack(expand=True, fill="both", padx=10, pady=10)

        # Pestaña: Cuestionarios
        tab_cuestionarios = tk.Frame(notebook, bg=ADMIN_BG)
        notebook.add(tab_cuestionarios, text="Cuestionarios")
        tk.Button(tab_cuestionarios, text="Seleccionar Cuestionario", command=self.admin_seleccionar_cuestionario,
                  font=(FONT_FAMILY, 12), bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO).pack(pady=5, fill="x", padx=10)
        tk.Button(tab_cuestionarios, text="Crear Cuestionario Personalizado", command=self.admin_crear_cuestionario,
                  font=(FONT_FAMILY, 12), bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO).pack(pady=5, fill="x", padx=10)
        tk.Button(tab_cuestionarios, text="Editar Cuestionario", command=self.admin_editar_cuestionario,
                  font=(FONT_FAMILY, 12), bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO).pack(pady=5, fill="x", padx=10)
        tk.Button(tab_cuestionarios, text="Eliminar Cuestionario Personalizado", command=self.admin_eliminar_cuestionario_personalizado,
                  font=(FONT_FAMILY, 12), bg=ADMIN_REINICIAR_COLOR, fg=COLOR_TEXTO).pack(pady=5, fill="x", padx=10)

        # Pestaña: Otras Funciones
        tab_otras = tk.Frame(admin_window, bg=ADMIN_BG)
        notebook.add(tab_otras, text="Otras Funciones")
        self.btn_toggle_custom = tk.Button(tab_otras, text="Activar Personalizaciones",
                                           command=self.admin_toggle_cuestionario,
                                           font=(FONT_FAMILY, 12), bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO)
        self.btn_toggle_custom.pack(pady=5, fill="x", padx=10)
        tk.Button(tab_otras, text="Editar Historial", command=self.admin_editar_historial,
                  font=(FONT_FAMILY, 12), bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO).pack(pady=5, fill="x", padx=10)
        tk.Button(tab_otras, text="Ver Estadísticas", command=self.admin_ver_estadisticas,
                  font=(FONT_FAMILY, 12), bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO).pack(pady=5, fill="x", padx=10)
        tk.Button(tab_otras, text="Ver Sugerencias", command=self.admin_ver_sugerencias,
                  font=(FONT_FAMILY, 12), bg="#004080", fg=COLOR_TEXTO).pack(pady=5, fill="x", padx=10)
        tk.Button(tab_otras, text="⏱ Cambiar Duración Temporizador", command=self.cambiar_tiempo_temporizador,
                  font=(FONT_FAMILY, 12), bg="#004080", fg=COLOR_TEXTO).pack(pady=5, fill="x", padx=10)
        tk.Button(tab_otras, text="Activar Saltar Respuestas", command=self.toggle_skip,
                  font=(FONT_FAMILY, 12), bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO).pack(pady=5, fill="x", padx=10)

        # Pestaña: Mantenimiento
        tab_mantenimiento = tk.Frame(admin_window, bg=ADMIN_BG)
        notebook.add(tab_mantenimiento, text="Mantenimiento")
        tk.Button(tab_mantenimiento, text="Exportar Datos", command=self.admin_exportar_datos,
                  font=(FONT_FAMILY, 12), bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO).pack(pady=5, fill="x", padx=10)
        tk.Button(tab_mantenimiento, text="Restaurar Datos Predeterminados", command=self.admin_reiniciar_datos,
                  font=(FONT_FAMILY, 12), bg=ADMIN_REINICIAR_COLOR, fg=COLOR_TEXTO).pack(pady=5, fill="x", padx=10)
        tk.Button(admin_window, text="Salir Modo Docente", command=lambda: self.set_admin_mode(False, admin_window),
                  font=(FONT_FAMILY, 12), bg="#CCCCCC", fg="black").pack(pady=10)

    def set_admin_mode(self, value, window=None):
        self.admin_mode = value
        if window:
            window.destroy()

    def toggle_skip(self):
        self.skip_enabled = not self.skip_enabled
        if self.skip_enabled:
            messagebox.showinfo("Saltar Respuestas", "Opción de saltar respuestas ACTIVADA.")
        else:
            messagebox.showinfo("Saltar Respuestas", "Opción de saltar respuestas DESACTIVADA.")

    def cambiar_tiempo_temporizador(self):
        nuevo_tiempo = simpledialog.askinteger("Cambiar Temporizador",
                                               "Ingrese la nueva duración del temporizador (en minutos):",
                                               parent=self.master,
                                               initialvalue=self.tiempo_inicial//60,
                                               minvalue=1)
        if nuevo_tiempo:
            self.tiempo_inicial = nuevo_tiempo * 60
            self.remaining_time = self.tiempo_inicial
            messagebox.showinfo("Temporizador", f"Nuevo tiempo de temporizador establecido: {nuevo_tiempo} minutos.")

    # --------------------- SELECCIÓN DE CUESTIONARIO ---------------------
    def seleccionar_tipo_cuestionario(self):
        # Si hay cuestionarios personalizados activos, preguntar al usuario
        if custom_cuestionarios and self.custom_questionnaire_enabled:
            tipo_window = tk.Toplevel(self.master)
            tipo_window.title("Seleccione el Tipo de Cuestionario")
            tipo_window.config(bg=COLOR_FONDO)
            self.centrar_ventana(tipo_window, 300, 200)
            tipo_window.transient(self.master)
            tipo_window.grab_set()
            tk.Label(tipo_window, text="¿Qué tipo de cuestionario desea completar?", font=FONT_NORMAL, bg=COLOR_FONDO).pack(pady=10)
            def iniciar_normal():
                tipo_window.destroy()
                self.iniciar_cuestionario_normal()
            def iniciar_personalizado():
                tipo_window.destroy()
                self.seleccionar_cuestionario_personalizado()
            tk.Button(tipo_window, text="Normal", font=FONT_NORMAL, bg=COLOR_BOTON, fg=COLOR_TEXTO,
                      command=iniciar_normal).pack(pady=5, fill="x", padx=20)
            tk.Button(tipo_window, text="Personalizado", font=FONT_NORMAL, bg=COLOR_BOTON, fg=COLOR_TEXTO,
                      command=iniciar_personalizado).pack(pady=5, fill="x", padx=20)
            tipo_window.wait_window()
        else:
            self.iniciar_cuestionario_normal()

    def iniciar_cuestionario_normal(self):
        self.is_custom = False
        # Selecciona el primer módulo de preguntas
        self.modulo_actual = list(modulos.keys())[0]
        self.pregunta_actual = 0
        self.pregunta_global = 0
        self.aciertos = 0
        self.incorrectos = 0
        self.respuestas_usuario = []
        self.total_preguntas = sum(len(modulos[m]) for m in modulos)
        self.preguntas_modulo = modulos[self.modulo_actual]
        if self.timer_mode:
            self.remaining_time = self.tiempo_inicial
        self.iniciar_interfaz_cuestionario("Módulo: " + self.modulo_actual)

    def seleccionar_cuestionario_personalizado(self):
        if not custom_cuestionarios:
            messagebox.showinfo("Información", "No hay cuestionarios personalizados.")
            return
        sel_window = tk.Toplevel(self.master)
        sel_window.title("Seleccionar Cuestionario Personalizado")
        sel_window.config(bg=COLOR_FONDO)
        self.centrar_ventana(sel_window, 300, 200)
        tk.Label(sel_window, text="Seleccione el cuestionario:", font=FONT_NORMAL, bg=COLOR_FONDO).pack(pady=10)
        listbox = tk.Listbox(sel_window, font=FONT_NORMAL, width=30, height=6)
        listbox.pack(pady=5)
        for nombre in custom_cuestionarios.keys():
            listbox.insert(tk.END, nombre)
        def seleccionar():
            selection = listbox.curselection()
            if selection:
                self.cuestionario_personalizado = listbox.get(selection[0])
                sel_window.destroy()
                self.iniciar_cuestionario_personalizado()
            else:
                messagebox.showwarning("Selección", "Seleccione un cuestionario.")
        tk.Button(sel_window, text="Seleccionar", font=FONT_NORMAL, bg=COLOR_BOTON, fg=COLOR_TEXTO,
                  command=seleccionar).pack(pady=10)
    def iniciar_cuestionario_personalizado(self):
        self.is_custom = True
        self.pregunta_actual = 0
        self.aciertos = 0
        self.incorrectos = 0
        self.respuestas_usuario = []
        self.preguntas_modulo = custom_cuestionarios[self.cuestionario_personalizado]
        self.total_preguntas = len(self.preguntas_modulo)
        if self.timer_mode:
            self.remaining_time = self.tiempo_inicial
        self.iniciar_interfaz_cuestionario("Cuestionario: " + self.cuestionario_personalizado)

    def iniciar_interfaz_cuestionario(self, subtitulo_text):
        for widget in self.master.winfo_children():
            widget.destroy()
        self.master.config(bg=COLOR_FONDO)
        header = tk.Frame(self.master, bg=COLOR_FONDO)
        header.pack(pady=10, fill="x")
        self.crear_label(header, "LearnPy", font=(FONT_FAMILY, 30, "bold")).pack(anchor="center")
        self.subtitulo = self.crear_label(header, subtitulo_text, font=FONT_SUBTITLE)
        self.subtitulo.pack(anchor="center")
        tk.Frame(self.master, height=20, bg=COLOR_FONDO).pack()
        self.etiqueta_pregunta = self.crear_label(self.master, "", font=(FONT_FAMILY, 17), wraplength=550)
        self.etiqueta_pregunta.pack(anchor="center", padx=20)
        response_frame = tk.Frame(self.master, bg=COLOR_FONDO, width=550, height=100)
        response_frame.pack(pady=10)
        response_frame.pack_propagate(False)
        response_frame.grid_rowconfigure(0, minsize=40)
        response_frame.grid_rowconfigure(1, minsize=40)
        self.combo_respuestas = ttk.Combobox(response_frame, state="readonly", width=30)
        self.combo_respuestas.grid(row=0, column=0, pady=(5,0))
        self.boton_aceptar = tk.Button(response_frame, text="Aceptar", command=self.verificar_respuesta,
                                       font=(FONT_FAMILY, 16, "bold"), bg=COLOR_BOTON, fg=COLOR_TEXTO)
        self.boton_aceptar.grid(row=1, column=0, pady=(15,5))
        if self.admin_mode and self.skip_enabled:
            tk.Button(response_frame, text="Saltar Pregunta", font=(FONT_FAMILY, 12, "bold"),
                      bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO, command=self.saltar_pregunta_admin).grid(row=1, column=1, padx=10)
        if self.timer_mode:
            self.timer_label = self.crear_label(self.master, "", font=(FONT_FAMILY, 14, "bold"))
            self.timer_label.pack(pady=(0,10))
            self.update_timer()
        self.progress = ttk.Progressbar(self.master, orient="horizontal", mode="determinate", maximum=self.total_preguntas)
        self.progress.pack(side="bottom", fill="x", padx=20, pady=10)
        self.mostrar_pregunta()

    def update_timer(self):
        minutes = self.remaining_time // 60
        seconds = self.remaining_time % 60
        self.timer_label.config(text=f"Tiempo restante: {minutes:02d}:{seconds:02d}")
        if self.remaining_time <= 0:
            messagebox.showinfo("Tiempo agotado", "Se agotó el tiempo. La prueba finaliza.")
            self.mostrar_resultados()
        else:
            self.remaining_time -= 1
            self.master.after(1000, self.update_timer)

    def mostrar_pregunta(self):
        if self.is_custom:
            if self.pregunta_actual < self.total_preguntas:
                pregunta = self.preguntas_modulo[self.pregunta_actual]
                self.etiqueta_pregunta.config(text=pregunta["pregunta"])
                respuestas = pregunta["respuestas"][:]
                random.shuffle(respuestas)
                self.combo_respuestas['values'] = ["Seleccionar opción"] + respuestas
                self.combo_respuestas.current(0)
                self.animate_progress(self.pregunta_actual)
            else:
                self.mostrar_resultados()
        else:
            if self.pregunta_global < self.total_preguntas:
                if self.pregunta_actual < len(self.preguntas_modulo):
                    pregunta = self.preguntas_modulo[self.pregunta_actual]
                    self.etiqueta_pregunta.config(text=pregunta["pregunta"])
                    respuestas = pregunta["respuestas"][:]
                    random.shuffle(respuestas)
                    self.combo_respuestas['values'] = ["Seleccionar opción"] + respuestas
                    self.combo_respuestas.current(0)
                    self.animate_progress(self.pregunta_global)
                else:
                    if self.modulo_actual != "Examen Final":
                        next_module_index = list(modulos.keys()).index(self.modulo_actual) + 1
                        if next_module_index < len(modulos):
                            self.modulo_actual = list(modulos.keys())[next_module_index]
                        else:
                            self.modulo_actual = "Examen Final"
                        messagebox.showinfo("Cambio de Módulo", f"Ahora estás en el módulo: {self.modulo_actual}")
                        self.pregunta_actual = 0
                        self.preguntas_modulo = modulos[self.modulo_actual]
                        self.subtitulo.config(text=f"Módulo: {self.modulo_actual}")
                        self.mostrar_pregunta()
                    else:
                        self.mostrar_resultados()
            else:
                self.mostrar_resultados()

    def verificar_respuesta(self):
        if self.combo_respuestas.current() == 0:
            messagebox.showerror("Error", "Seleccione una respuesta válida.")
            return
        pregunta = self.preguntas_modulo[self.pregunta_actual]
        respuesta_sel = self.combo_respuestas.get()
        correcta = pregunta["respuestas"][pregunta["correcta"]]
        if respuesta_sel == correcta:
            exp_text = pregunta.get("explicacion_correcta", "Respuesta correcta")
            self.aciertos += 1
        else:
            exp_text = pregunta.get("explicacion_incorrecta", "Respuesta incorrecta")
            self.incorrectos += 1
        info = {
            "pregunta": pregunta["pregunta"],
            "respuesta_seleccionada": respuesta_sel,
            "respuesta_correcta": correcta,
            "explicacion": exp_text,
            "es_correcta": respuesta_sel == correcta
        }
        self.respuestas_usuario.append(info)
        if self.is_custom:
            self.pregunta_actual += 1
        else:
            self.pregunta_actual += 1
            self.pregunta_global += 1
        self.mostrar_pregunta()

    def mostrar_resultados(self):
        for widget in self.master.winfo_children():
            widget.destroy()
        self.guardar_en_historial()
        total = self.aciertos + self.incorrectos
        porcentaje = (self.aciertos / total) * 100 if total > 0 else 0
        datos = (f"Estudiante: {self.usuario['nombre']} | Año: {self.usuario['año']} | "
                 f"Mención: {self.usuario['Mencion']} | Sección: {self.usuario['sección']}")
        self.crear_label(self.master, datos, font=(FONT_FAMILY, 14, "bold")).pack(pady=(10,5), fill="x", padx=10)
        self.crear_label(self.master, "Calificación Final", font=(FONT_FAMILY, 18, "bold")).pack(pady=(10,5), fill="x", padx=10)
        self.crear_label(self.master, f"Puntaje: {porcentaje:.2f}%\nAciertos: {self.aciertos} | Errores: {self.incorrectos}",
                         font=(FONT_FAMILY, 20, "bold"), fg="blue").pack(pady=5, fill="x", padx=10)
        tk.Button(self.master, text="Nueva Prueba", command=self.nueva_prueba,
                  font=(FONT_FAMILY, 16, "bold"), bg=BTN_REINICIAR_COLOR, fg="white").pack(pady=5)
        tk.Button(self.master, text="Mostrar Detalle", command=self.mostrar_detalle,
                  font=(FONT_FAMILY, 16, "bold"), bg=BTN_CONTINUAR_COLOR, fg="white").pack(pady=5)
        messagebox.showinfo("Gracias", "¡Gracias por usar LearnPy!")

    def guardar_en_historial(self, nueva_prueba=False):
        total = self.aciertos + self.incorrectos
        porcentaje = (self.aciertos / total) * 100 if total > 0 else 0
        now = datetime.datetime.now()
        documents_folder = os.path.join(os.path.expanduser("~"), "Documents")
        historial_folder = os.path.join(documents_folder, "LearnPyHistorial")
        if not os.path.exists(historial_folder):
            os.makedirs(historial_folder)
        tipo = self.cuestionario_personalizado if self.is_custom else "Normal"
        file_name = f"resultados_{self.usuario['nombre']}_{self.usuario['año']}_{self.usuario['Mencion']}_{self.usuario['sección']}_{tipo}_{now.strftime('%Y%m%d_%H%M%S')}.txt"
        full_path = os.path.join(historial_folder, file_name)
        try:
            with open(full_path, "w", encoding="utf-8") as f:
                f.write("Resultados del Examen LearnPy\n")
                f.write("=====================================\n\n")
                f.write(f"Estudiante: {self.usuario['nombre']}\n")
                f.write(f"Año: {self.usuario['año']}\n")
                f.write(f"Mención: {self.usuario['Mencion']}\n")
                f.write(f"Sección: {self.usuario['sección']}\n\n")
                f.write(f"Tipo de examen: {tipo}\n")
                f.write(f"Puntaje: {porcentaje:.2f}%\n")
                f.write(f"Aciertos: {self.aciertos}\n")
                f.write(f"Errores: {self.incorrectos}\n\n")
                f.write("Detalle de respuestas:\n")
                for resp in self.respuestas_usuario:
                    f.write(f"\nPregunta: {resp['pregunta']}\n")
                    f.write(f"Respuesta Seleccionada: {resp['respuesta_seleccionada']}\n")
                    f.write(f"Respuesta Correcta: {resp['respuesta_correcta']}\n")
                    f.write(f"Explicación: {resp['explicacion']}\n")
        except Exception as e:
            messagebox.showerror("Error", f"No se pudo guardar el historial: {e}")

    def nueva_prueba(self):
        self.menu_registro()

    def mostrar_detalle(self):
        detalle_window = tk.Toplevel(self.master)
        detalle_window.title("Detalle de Respuestas")
        detalle_window.config(bg=COLOR_FONDO)
        self.centrar_ventana(detalle_window, 600, 400)
        text_area = tk.Text(detalle_window, font=(FONT_FAMILY, 12))
        text_area.pack(expand=True, fill="both", padx=10, pady=10)
        for resp in self.respuestas_usuario:
            text_area.insert(tk.END, f"Pregunta: {resp['pregunta']}\n")
            text_area.insert(tk.END, f"Respuesta Seleccionada: {resp['respuesta_seleccionada']}\n")
            text_area.insert(tk.END, f"Respuesta Correcta: {resp['respuesta_correcta']}\n")
            text_area.insert(tk.END, f"Explicación: {resp['explicacion']}\n")
            text_area.insert(tk.END, "------------------------------------------\n")
        tk.Button(detalle_window, text="Cerrar", command=detalle_window.destroy,
                  font=(FONT_FAMILY, 12), bg="#CCCCCC", fg="black").pack(pady=10)

    def acceder_historial(self):
        historial_folder = os.path.join(os.path.expanduser("~"), "Documents", "LearnPyHistorial")
        if not os.path.exists(historial_folder):
            os.makedirs(historial_folder)
        hist_window = tk.Toplevel(self.master)
        hist_window.title("Historial de Pruebas")
        hist_window.config(bg=COLOR_FONDO)
        self.centrar_ventana(hist_window, 600, 400)
        listbox = tk.Listbox(hist_window, font=(FONT_FAMILY, 12))
        listbox.pack(side="left", fill="both", expand=True, padx=10, pady=10)
        scrollbar = tk.Scrollbar(hist_window)
        scrollbar.pack(side="right", fill="y")
        listbox.config(yscrollcommand=scrollbar.set)
        scrollbar.config(command=listbox.yview)
        files = [f for f in os.listdir(historial_folder) if f.endswith(".txt")]
        for file in files:
            listbox.insert(tk.END, file)
        def mostrar_archivo(event):
            selection = listbox.curselection()
            if selection:
                file_sel = listbox.get(selection[0])
                file_path = os.path.join(historial_folder, file_sel)
                try:
                    with open(file_path, "r", encoding="utf-8") as f:
                        content = f.read()
                    detalle_hist = tk.Toplevel(self.master)
                    detalle_hist.title(file_sel)
                    detalle_hist.config(bg=COLOR_FONDO)
                    self.centrar_ventana(detalle_hist, 600, 400)
                    text_hist = tk.Text(detalle_hist, font=(FONT_FAMILY, 12))
                    text_hist.pack(expand=True, fill="both", padx=10, pady=10)
                    text_hist.insert(tk.END, content)
                    tk.Button(detalle_hist, text="Cerrar", command=detalle_hist.destroy,
                              font=(FONT_FAMILY, 12), bg="#CCCCCC", fg="black").pack(pady=10)
                except Exception as e:
                    messagebox.showerror("Error", f"No se pudo abrir el archivo: {e}")
        listbox.bind("<<ListboxSelect>>", mostrar_archivo)

    # --------------------- TUTORIAL E INFORMACIÓN ---------------------
    def mostrar_tutorial_info(self):
        tutorial_win = tk.Toplevel(self.master)
        tutorial_win.title("Tutorial e Información")
        tutorial_win.config(bg=COLOR_FONDO)
        root_x = self.master.winfo_rootx()
        root_y = self.master.winfo_rooty()
        tutorial_win.geometry(f"700x500+{root_x}+{root_y}")
        notebook = ttk.Notebook(tutorial_win)
        notebook.pack(expand=True, fill="both")
        frame_tutorial = tk.Frame(notebook, bg=COLOR_FONDO)
        tutorial_text = (
            "Bienvenido a LearnPy!\n\n"
            "Tutorial de Uso:\n"
            "1. Regístrate en el formulario de inicio.\n"
            "2. Selecciona el tipo de cuestionario (Normal o Personalizado).\n"
            "3. Responde las preguntas y al finalizar se mostrará tu calificación y el detalle de respuestas.\n"
            "4. Si eres docente, aprovecha las funciones exclusivas para personalizar cuestionarios.\n\n"
            "¡Disfruta aprendiendo Python de forma interactiva!"
        )
        tk.Label(frame_tutorial, text=tutorial_text, font=FONT_NORMAL, bg=COLOR_FONDO, justify="left", wraplength=680).pack(anchor="nw", padx=5, pady=5)
        notebook.add(frame_tutorial, text="Tutorial")
        frame_info = tk.Frame(notebook, bg=COLOR_FONDO)
        info_text = (
            "Información de LearnPy - Versión 1.1\n\n"
            "Características:\n"
            "• Cuestionarios interactivos para aprender Python.\n"
            "• Exclusivo para docentes.\n"
            "• Historial y estadísticas de desempeño.\n"
            "• Ventanas emergentes personalizadas.\n"
            "• Soporte para cuestionarios finales.\n\n"
            "Autores:\n"
            "Brahyam Velasquez, Jose Gomes, Jose Pereira, Samuel Meneses\n\n"
            "Nota:\n"
            "LearnPy está en constante desarrollo; se planean futuras mejoras en la interfaz y nuevas funciones."
        )
        tk.Label(frame_info, text=info_text, font=FONT_NORMAL, bg=COLOR_FONDO, justify="left", wraplength=680).pack(anchor="nw", padx=5, pady=5)
        notebook.add(frame_info, text="Información")

    # --------------------- ENVIAR SUGERENCIAS ---------------------
    def abrir_ventana_sugerencia(self):
        sug_win = tk.Toplevel(self.master)
        sug_win.title("Enviar Sugerencias")
        sug_win.config(bg=COLOR_FONDO)
        root_x = self.master.winfo_rootx()
        root_y = self.master.winfo_rooty()
        sug_win.geometry(f"600x400+{root_x}+{root_y}")
        tk.Label(sug_win, text="Escribe tu sugerencia a LearnPy:", font=FONT_NORMAL, bg=COLOR_FONDO).pack(padx=10, pady=10)
        text_area = tk.Text(sug_win, font=FONT_NORMAL, height=10, width=70)
        text_area.pack(padx=10, pady=10)
        def enviar():
            sugerencia = text_area.get("1.0", tk.END).strip()
            if sugerencia:
                try:
                    with open("sugerencias.txt", "a", encoding="utf-8") as f:
                        now = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
                        f.write(f"{now} - {sugerencia}\n")
                    messagebox.showinfo("Sugerencia Enviada", "¡Gracias por tu sugerencia!")
                    sug_win.destroy()
                except Exception as e:
                    messagebox.showerror("Error", f"No se pudo enviar la sugerencia: {e}")
            else:
                messagebox.showinfo("Sugerencia", "No se ingresó ninguna sugerencia.")
        tk.Button(sug_win, text="Enviar Sugerencias", font=FONT_NORMAL, bg=COLOR_BOTON, fg=COLOR_TEXTO,
                  command=enviar).pack(pady=10)

    # --------------------- VER SUGERENCIAS (Modo Docente) ---------------------
    def admin_ver_sugerencias(self):
        if not os.path.exists("sugerencias.txt"):
            messagebox.showinfo("Sugerencias", "No hay sugerencias registradas.")
            return
        try:
            with open("sugerencias.txt", "r", encoding="utf-8") as f:
                content = f.read()
            sug_win = tk.Toplevel(self.master)
            sug_win.title("Sugerencias")
            sug_win.config(bg=COLOR_FONDO)
            self.centrar_ventana(sug_win, 600, 400)
            text = tk.Text(sug_win, font=FONT_NORMAL)
            text.pack(expand=True, fill="both", padx=10, pady=10)
            text.insert(tk.END, content)
            text.config(state="disabled")
            tk.Button(sug_win, text="Cerrar", command=sug_win.destroy, font=FONT_NORMAL, bg="#CCCCCC", fg="black").pack(pady=10)
        except Exception as e:
            messagebox.showerror("Error", f"No se pudo abrir el archivo de sugerencias: {e}")

    # --------------------- MÉTODOS ADMINISTRATIVOS (Crear/Editar/Eliminar Cuestionarios) ---------------------
    def admin_seleccionar_cuestionario(self):
        sel_window = tk.Toplevel(self.master)
        sel_window.title("Seleccionar Cuestionario")
        sel_window.config(bg=ADMIN_BG)
        self.centrar_ventana(sel_window, 450, 250)
        tk.Label(sel_window, text="Seleccione el cuestionario:", font=(FONT_FAMILY, 12), bg=ADMIN_BG, fg=ADMIN_FG).pack(pady=10)
        listbox = tk.Listbox(sel_window, font=(FONT_FAMILY, 12), width=30, height=6)
        listbox.pack(pady=5)
        for m in modulos.keys():
            listbox.insert(tk.END, m)
        def seleccionar():
            selection = listbox.curselection()
            if selection:
                self.modulo_actual = listbox.get(selection[0])
                self.custom_module_enabled = True
                messagebox.showinfo("Seleccionado", f"Cuestionario seleccionado: {self.modulo_actual}")
                sel_window.destroy()
            else:
                messagebox.showwarning("Selección", "Seleccione un cuestionario.")
        tk.Button(sel_window, text="Seleccionar", font=(FONT_FAMILY, 12), bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO,
                  command=seleccionar).pack(pady=10)
    def admin_toggle_cuestionario(self):
        self.custom_module_enabled = not self.custom_module_enabled
        self.custom_questionnaire_enabled = not self.custom_questionnaire_enabled
        if self.custom_module_enabled and self.custom_questionnaire_enabled:
            self.btn_toggle_custom.config(text="Desactivar Personalizaciones", bg=ADMIN_BTN_COLOR)
            messagebox.showinfo("Personalizaciones", "Personalizaciones ACTIVADAS.")
        else:
            self.btn_toggle_custom.config(text="Activar Personalizaciones", bg="red")
            messagebox.showinfo("Personalizaciones", "Personalizaciones DESACTIVADAS.")
    def admin_crear_cuestionario(self):
        crear_window = tk.Toplevel(self.master)
        crear_window.title("Crear Cuestionario Personalizado")
        crear_window.config(bg=ADMIN_BG)
        self.centrar_ventana(crear_window, 700, 650)
        title_frame = tk.Frame(crear_window, bg=ADMIN_BG)
        title_frame.pack(fill="x", padx=10, pady=10)
        tk.Label(title_frame, text="Nombre del Cuestionario:", font=(FONT_FAMILY, 14, "bold"), bg=ADMIN_BG, fg=ADMIN_FG).pack(side="left")
        entry_nombre = tk.Entry(title_frame, font=(FONT_FAMILY, 12), width=40)
        entry_nombre.pack(side="left", padx=10)
        cuestionario_temp = []
        form_frame = tk.Frame(crear_window, bg=ADMIN_BG)
        form_frame.pack(fill="both", expand=True, padx=10, pady=10)
        tk.Label(form_frame, text="Pregunta:", font=(FONT_FAMILY, 12, "bold"), bg=ADMIN_BG, fg=ADMIN_FG).grid(row=0, column=0, sticky="w", pady=2)
        entry_pregunta = tk.Entry(form_frame, font=(FONT_FAMILY, 12), width=80)
        entry_pregunta.grid(row=0, column=1, columnspan=3, pady=2, sticky="w")
        tk.Label(form_frame, text="Opción 1:", font=(FONT_FAMILY, 12), bg=ADMIN_BG, fg=ADMIN_FG).grid(row=1, column=0, sticky="w", pady=2)
        entry_resp1 = tk.Entry(form_frame, font=(FONT_FAMILY, 12), width=40)
        entry_resp1.grid(row=1, column=1, pady=2, sticky="w")
        tk.Label(form_frame, text="Opción 2:", font=(FONT_FAMILY, 12), bg=ADMIN_BG, fg=ADMIN_FG).grid(row=2, column=0, sticky="w", pady=2)
        entry_resp2 = tk.Entry(form_frame, font=(FONT_FAMILY, 12), width=40)
        entry_resp2.grid(row=2, column=1, pady=2, sticky="w")
        tk.Label(form_frame, text="Opción 3:", font=(FONT_FAMILY, 12), bg=ADMIN_BG, fg=ADMIN_FG).grid(row=3, column=0, sticky="w", pady=2)
        entry_resp3 = tk.Entry(form_frame, font=(FONT_FAMILY, 12), width=40)
        entry_resp3.grid(row=3, column=1, pady=2, sticky="w")
        tk.Label(form_frame, text="Respuesta Correcta:", font=(FONT_FAMILY, 12), bg=ADMIN_BG, fg=ADMIN_FG).grid(row=4, column=0, sticky="w", pady=2)
        combo_correcta = ttk.Combobox(form_frame, values=["Opción 1", "Opción 2", "Opción 3"],
                                       state="readonly", width=37, font=(FONT_FAMILY, 12))
        combo_correcta.grid(row=4, column=1, pady=2, sticky="w")
        combo_correcta.current(0)
        tk.Label(form_frame, text="Explicación si es Correcta:", font=(FONT_FAMILY, 12), bg=ADMIN_BG, fg=ADMIN_FG).grid(row=5, column=0, sticky="nw", pady=2)
        text_explicacion_correcta = tk.Text(form_frame, font=(FONT_FAMILY, 12), height=3, width=60)
        text_explicacion_correcta.grid(row=5, column=1, columnspan=3, pady=2, sticky="w")
        tk.Label(form_frame, text="Explicación si es Incorrecta:", font=(FONT_FAMILY, 12), bg=ADMIN_BG, fg=ADMIN_FG).grid(row=6, column=0, sticky="nw", pady=2)
        text_explicacion_incorrecta = tk.Text(form_frame, font=(FONT_FAMILY, 12), height=3, width=60)
        text_explicacion_incorrecta.grid(row=6, column=1, columnspan=3, pady=2, sticky="w")
        input_btn_frame = tk.Frame(crear_window, bg=ADMIN_BG)
        input_btn_frame.pack(fill="x", padx=10, pady=5)
        def agregar_pregunta():
            pregunta_text = entry_pregunta.get().strip()
            resp1 = entry_resp1.get().strip()
            resp2 = entry_resp2.get().strip()
            resp3 = entry_resp3.get().strip()
            explicacion_correcta = text_explicacion_correcta.get("1.0", tk.END).strip()
            explicacion_incorrecta = text_explicacion_incorrecta.get("1.0", tk.END).strip()
            if not pregunta_text or not resp1 or not resp2 or not resp3:
                messagebox.showerror("Error", "Debe completar la pregunta y las tres opciones.")
                return
            pregunta_dict = {
                "pregunta": pregunta_text,
                "respuestas": [resp1, resp2, resp3],
                "correcta": combo_correcta.current(),
                "explicacion_correcta": explicacion_correcta,
                "explicacion_incorrecta": explicacion_incorrecta
            }
            cuestionario_temp.append(pregunta_dict)
            entry_pregunta.delete(0, tk.END)
            entry_resp1.delete(0, tk.END)
            entry_resp2.delete(0, tk.END)
            entry_resp3.delete(0, tk.END)
            text_explicacion_correcta.delete("1.0", tk.END)
            text_explicacion_incorrecta.delete("1.0", tk.END)
            listbox_questions.insert(tk.END, pregunta_text)
        btn_agregar = tk.Button(input_btn_frame, text="Agregar Pregunta", font=(FONT_FAMILY, 12, "bold"),
                                bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO, relief="ridge", borderwidth=2, command=agregar_pregunta)
        btn_agregar.pack(side="left", padx=5, expand=True, fill="x")
        def vista_previa():
            pregunta_text = entry_pregunta.get().strip()
            resp1 = entry_resp1.get().strip()
            resp2 = entry_resp2.get().strip()
            resp3 = entry_resp3.get().strip()
            explicacion_correcta = text_explicacion_correcta.get("1.0", tk.END).strip()
            explicacion_incorrecta = text_explicacion_incorrecta.get("1.0", tk.END).strip()
            if not pregunta_text:
                messagebox.showerror("Error", "Ingrese la pregunta para vista previa.")
                return
            preview_window = tk.Toplevel(crear_window)
            preview_window.title("Vista Previa de la Pregunta")
            preview_window.config(bg=ADMIN_BG)
            self.centrar_ventana(preview_window, 500, 400)
            tk.Label(preview_window, text="Pregunta:", font=(FONT_FAMILY, 12, "bold"),
                     bg=ADMIN_BG, fg=ADMIN_FG, relief="ridge", borderwidth=2).pack(anchor="w", padx=10, pady=5)
            tk.Label(preview_window, text=pregunta_text, font=(FONT_FAMILY, 12),
                     bg=ADMIN_BG, fg=ADMIN_FG, wraplength=480, justify="left",
                     relief="ridge", borderwidth=2).pack(anchor="w", padx=10)
            tk.Label(preview_window, text="Opciones:", font=(FONT_FAMILY, 12, "bold"),
                     bg=ADMIN_BG, fg=ADMIN_FG, relief="ridge", borderwidth=2).pack(anchor="w", padx=10, pady=5)
            for i, opt in enumerate([resp1, resp2, resp3]):
                tk.Label(preview_window, text=f"{i+1}. {opt}", font=(FONT_FAMILY, 12),
                         bg=ADMIN_BG, fg=ADMIN_FG, wraplength=480, justify="left",
                         relief="ridge", borderwidth=2).pack(anchor="w", padx=10)
            tk.Label(preview_window, text="Explicación si es Correcta:", font=(FONT_FAMILY, 12, "bold"),
                     bg=ADMIN_BG, fg=ADMIN_FG, relief="ridge", borderwidth=2).pack(anchor="w", padx=10, pady=5)
            tk.Label(preview_window, text=explicacion_correcta, font=(FONT_FAMILY, 12),
                     bg=ADMIN_BG, fg=ADMIN_FG, wraplength=480, justify="left",
                     relief="ridge", borderwidth=2).pack(anchor="w", padx=10)
            tk.Label(preview_window, text="Explicación si es Incorrecta:", font=(FONT_FAMILY, 12, "bold"),
                     bg=ADMIN_BG, fg=ADMIN_FG, relief="ridge", borderwidth=2).pack(anchor="w", padx=10, pady=5)
            tk.Label(preview_window, text=explicacion_incorrecta, font=(FONT_FAMILY, 12),
                     bg=ADMIN_BG, fg=ADMIN_FG, wraplength=480, justify="left",
                     relief="ridge", borderwidth=2).pack(anchor="w", padx=10)
        btn_vista_previa = tk.Button(input_btn_frame, text="Vista Previa", command=vista_previa,
                                     font=(FONT_FAMILY, 12, "bold"), bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO,
                                     relief="ridge", borderwidth=2)
        btn_vista_previa.pack(side="left", padx=5, expand=True, fill="x")
        listbox_frame = tk.Frame(crear_window, bg=ADMIN_BG)
        listbox_frame.pack(fill="both", padx=10, pady=10, expand=True)
        tk.Label(listbox_frame, text="Preguntas Agregadas:", font=(FONT_FAMILY, 12, "bold"),
                 bg=ADMIN_BG, fg=ADMIN_FG, relief="ridge", borderwidth=2).pack(anchor="w")
        listbox_questions = tk.Listbox(listbox_frame, font=(FONT_FAMILY, 12), height=5, relief="ridge", borderwidth=2)
        listbox_questions.pack(fill="both", padx=5, pady=5, expand=True)
        action_btn_frame = tk.Frame(crear_window, bg=ADMIN_BG)
        action_btn_frame.pack(fill="x", padx=10, pady=10)
        def eliminar_pregunta():
            selection = listbox_questions.curselection()
            if not selection:
                messagebox.showwarning("Advertencia", "Seleccione una pregunta para eliminar.")
                return
            index = selection[0]
            listbox_questions.delete(index)
            del cuestionario_temp[index]
        btn_eliminar = tk.Button(action_btn_frame, text="Eliminar Pregunta Seleccionada",
                                 font=(FONT_FAMILY, 12, "bold"), bg=ADMIN_REINICIAR_COLOR, fg=COLOR_TEXTO,
                                 relief="ridge", borderwidth=2, command=eliminar_pregunta)
        btn_eliminar.pack(side="left", padx=5, expand=True, fill="x")
        def guardar_cuestionario_form():
            nombre = entry_nombre.get().strip()
            if not nombre:
                messagebox.showerror("Error", "Ingrese el nombre del cuestionario.")
                return
            if not cuestionario_temp:
                messagebox.showerror("Error", "No se ha agregado ninguna pregunta.")
                return
            custom_cuestionarios[nombre] = cuestionario_temp.copy()
            guardar_cuestionarios_custom()
            messagebox.showinfo("Guardar", f"Cuestionario '{nombre}' guardado exitosamente.")
            crear_window.destroy()
        btn_guardar = tk.Button(action_btn_frame, text="Guardar Cuestionario",
                                font=(FONT_FAMILY, 12, "bold"), bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO,
                                relief="ridge", borderwidth=2, command=guardar_cuestionario_form)
        btn_guardar.pack(side="left", padx=5, expand=True, fill="x")

    def admin_editar_cuestionario(self):
        if not custom_cuestionarios:
            messagebox.showinfo("Editar", "No hay cuestionarios personalizados para editar.")
            return
        edit_window = tk.Toplevel(self.master)
        edit_window.title("Editar Cuestionario Personalizado")
        edit_window.config(bg=ADMIN_BG)
        self.centrar_ventana(edit_window, 600, 500)
        left_frame = tk.Frame(edit_window, bg=ADMIN_BG)
        left_frame.pack(side="left", fill="y", padx=10, pady=10)
        right_frame = tk.Frame(edit_window, bg=ADMIN_BG)
        right_frame.pack(side="right", fill="both", expand=True, padx=10, pady=10)
        tk.Label(left_frame, text="Cuestionarios Disponibles", font=(FONT_FAMILY, 12, "bold"), bg=ADMIN_BG, fg=ADMIN_FG).pack(pady=5)
        listbox = tk.Listbox(left_frame, font=(FONT_FAMILY, 12), width=30, height=15, relief="ridge", borderwidth=2)
        listbox.pack(pady=5)
        for key in custom_cuestionarios.keys():
            listbox.insert(tk.END, key)
        text_widget = tk.Text(right_frame, wrap="word", font=(FONT_FAMILY, 12), relief="ridge", borderwidth=2)
        text_widget.pack(fill="both", expand=True)
        def cargar_cuestionario(event):
            selection = listbox.curselection()
            if selection:
                key = listbox.get(selection[0])
                content = json.dumps(custom_cuestionarios[key], indent=2)
                text_widget.config(state="normal")
                text_widget.delete("1.0", tk.END)
                text_widget.insert(tk.END, content)
                text_widget.config(state="disabled")
        listbox.bind("<<ListboxSelect>>", cargar_cuestionario)
        def guardar_edicion():
            selection = listbox.curselection()
            if not selection:
                messagebox.showwarning("Guardar", "Seleccione un cuestionario para editar.")
                return
            key = listbox.get(selection[0])
            text_widget.config(state="normal")
            new_content = text_widget.get("1.0", tk.END)
            text_widget.config(state="disabled")
            try:
                new_data = json.loads(new_content)
                if not isinstance(new_data, list):
                    raise ValueError("Debe ser una lista de preguntas.")
            except Exception as e:
                messagebox.showerror("Error", f"Formato JSON inválido: {e}")
                return
            custom_cuestionarios[key] = new_data
            guardar_cuestionarios_custom()
            messagebox.showinfo("Editar", f"Cuestionario '{key}' actualizado.")
            edit_window.destroy()
        tk.Button(edit_window, text="Guardar Edición", font=(FONT_FAMILY, 12), bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO,
                  relief="ridge", borderwidth=2, command=guardar_edicion).pack(pady=10)

    def admin_eliminar_cuestionario_personalizado(self):
        if not custom_cuestionarios:
            messagebox.showinfo("Eliminar", "No hay cuestionarios personalizados para eliminar.")
            return
        del_window = tk.Toplevel(self.master)
        del_window.title("Eliminar Cuestionario Personalizado")
        del_window.config(bg=ADMIN_BG)
        self.centrar_ventana(del_window, 400, 300)
        tk.Label(del_window, text="Seleccione el cuestionario a eliminar:", font=(FONT_FAMILY, 12, "bold"),
                 bg=ADMIN_BG, fg=ADMIN_FG).pack(pady=10)
        listbox = tk.Listbox(del_window, font=(FONT_FAMILY, 12), width=40, height=6, relief="ridge", borderwidth=2)
        listbox.pack(pady=5)
        for q in custom_cuestionarios.keys():
            listbox.insert(tk.END, q)
        def eliminar():
            selection = listbox.curselection()
            if selection:
                q_selected = listbox.get(selection[0])
                confirm = messagebox.askyesno("Confirmar", f"¿Eliminar '{q_selected}'?")
                if confirm:
                    try:
                        del custom_cuestionarios[q_selected]
                        guardar_cuestionarios_custom()
                        messagebox.showinfo("Eliminado", f"Cuestionario '{q_selected}' eliminado.")
                        listbox.delete(selection[0])
                    except Exception as e:
                        messagebox.showerror("Error", f"No se pudo eliminar: {e}")
            else:
                messagebox.showwarning("Eliminar", "Seleccione un cuestionario.")
        tk.Button(del_window, text="Eliminar", font=(FONT_FAMILY, 12), bg=ADMIN_REINICIAR_COLOR, fg=COLOR_TEXTO,
                  relief="ridge", borderwidth=2, command=eliminar).pack(pady=10)

    def admin_exportar_datos(self):
        datos = {"modulos": modulos, "custom_cuestionarios": custom_cuestionarios}
        dest = filedialog.asksaveasfilename(title="Exportar Datos", defaultextension=".json",
                                              filetypes=[("JSON files", "*.json")])
        if dest:
            try:
                with open(dest, "w", encoding="utf-8") as f:
                    json.dump(datos, f, indent=4)
                messagebox.showinfo("Exportar", f"Datos exportados a {dest}")
            except Exception as e:
                messagebox.showerror("Error", f"No se pudo exportar: {e}")

    def admin_reiniciar_datos(self):
        confirm = messagebox.askyesno("Restaurar Datos", "Se restaurarán los datos predeterminados. ¿Continuar?")
        if confirm:
            global modulos, custom_cuestionarios
            modulos = copy.deepcopy(default_modulos)
            custom_cuestionarios = {}
            self.guardar_modulos_persistentes()
            guardar_cuestionarios_custom()
            messagebox.showinfo("Restaurar", "Datos restaurados a valores predeterminados.")

    def admin_editar_historial(self):
        historial_folder = os.path.join(os.path.expanduser("~"), "Documents", "LearnPyHistorial")
        if not os.path.exists(historial_folder):
            os.makedirs(historial_folder)
        edit_window = tk.Toplevel(self.master)
        edit_window.title("Editar Historial")
        edit_window.config(bg=ADMIN_BG)
        self.centrar_ventana(edit_window, 800, 600)
        left_frame = tk.Frame(edit_window, bg=ADMIN_BG)
        left_frame.pack(side="left", fill="y", padx=10, pady=10)
        right_frame = tk.Frame(edit_window, bg=ADMIN_BG)
        right_frame.pack(side="right", fill="both", expand=True, padx=10, pady=10)
        tk.Label(left_frame, text="Pruebas Guardadas", font=(FONT_FAMILY, 14, "bold"), bg=ADMIN_BG, fg=ADMIN_FG).pack(pady=(0,10))
        listbox = tk.Listbox(left_frame, width=40, height=25, font=(FONT_FAMILY, 12), relief="ridge", borderwidth=2)
        listbox.pack(fill="y", expand=True)
        scrollbar = tk.Scrollbar(left_frame)
        scrollbar.pack(side="right", fill="y")
        listbox.config(yscrollcommand=scrollbar.set)
        scrollbar.config(command=listbox.yview)
        files = [f for f in os.listdir(historial_folder) if f.endswith(".txt")]
        for file in files:
            listbox.insert(tk.END, file)
        text_widget = tk.Text(right_frame, wrap="word", font=(FONT_FAMILY, 12), relief="ridge", borderwidth=2)
        text_widget.pack(fill="both", expand=True)
        text_widget.config(state="disabled")
        btn_frame = tk.Frame(right_frame, bg=ADMIN_BG)
        btn_frame.pack(fill="x", pady=5)
        tk.Button(btn_frame, text="Descargar", font=(FONT_FAMILY, 12, "bold"), bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO,
                  relief="ridge", borderwidth=2, command=lambda: self.descargar_archivo(listbox, historial_folder)).pack(side="left", padx=5)
        tk.Button(btn_frame, text="Eliminar", font=(FONT_FAMILY, 12, "bold"), bg=ADMIN_REINICIAR_COLOR, fg=COLOR_TEXTO,
                  relief="ridge", borderwidth=2, command=lambda: self.eliminar_archivo(listbox, text_widget, historial_folder)).pack(side="left", padx=5)
        tk.Button(btn_frame, text="Eliminar Todo", font=(FONT_FAMILY, 12, "bold"), bg="red", fg=COLOR_TEXTO,
                  relief="ridge", borderwidth=2, command=lambda: self.eliminar_todo(historial_folder, listbox, text_widget)).pack(side="left", padx=5)
        def cargar_archivo(event):
            selection = listbox.curselection()
            if selection:
                file_sel = listbox.get(selection[0])
                file_path = os.path.join(historial_folder, file_sel)
                try:
                    with open(file_path, "r", encoding="utf-8") as f:
                        content = f.read()
                    detalle_hist = tk.Toplevel(self.master)
                    detalle_hist.title(file_sel)
                    detalle_hist.config(bg=COLOR_FONDO)
                    self.centrar_ventana(detalle_hist, 600, 400)
                    text_hist = tk.Text(detalle_hist, font=(FONT_FAMILY, 12))
                    text_hist.pack(expand=True, fill="both", padx=10, pady=10)
                    text_hist.insert(tk.END, content)
                    tk.Button(detalle_hist, text="Cerrar", command=detalle_hist.destroy,
                              font=(FONT_FAMILY, 12), bg="#CCCCCC", fg="black").pack(pady=10)
                except Exception as e:
                    messagebox.showerror("Error", f"No se pudo abrir el archivo: {e}")
        listbox.bind("<<ListboxSelect>>", cargar_archivo)

    def admin_ver_estadisticas(self):
        historial_folder = os.path.join(os.path.expanduser("~"), "Documents", "LearnPyHistorial")
        if not os.path.exists(historial_folder):
            messagebox.showinfo("Estadísticas", "No hay pruebas guardadas.")
            return
        scores = []
        for filename in os.listdir(historial_folder):
            if filename.endswith(".txt"):
                try:
                    with open(os.path.join(historial_folder, filename), "r", encoding="utf-8") as f:
                        for line in f:
                            if "Puntaje:" in line:
                                parts = line.split(":")
                                if len(parts) > 1:
                                    score_str = parts[1].strip().replace("%", "")
                                    try:
                                        score = float(score_str)
                                        scores.append(score)
                                    except:
                                        pass
                                break
                except:
                    pass
        if not scores:
            messagebox.showinfo("Estadísticas", "No se encontraron puntajes.")
            return
        total_tests = len(scores)
        avg_score = sum(scores) / total_tests
        best_score = max(scores)
        worst_score = min(scores)
        stats_msg = (f"Total de pruebas: {total_tests}\nPuntaje Promedio: {avg_score:.2f}%\n"
                     f"Mejor Puntaje: {best_score:.2f}%\nPeor Puntaje: {worst_score:.2f}%")
        stat_window = tk.Toplevel(self.master)
        stat_window.title("Estadísticas de Pruebas - LearnPy")
        stat_window.config(bg=ADMIN_BG)
        self.centrar_ventana(stat_window, 400, 200)
        tk.Label(stat_window, text=stats_msg, font=(FONT_FAMILY, 12), bg=ADMIN_BG, fg=ADMIN_FG).pack(pady=20)
        tk.Button(stat_window, text="Cerrar", font=(FONT_FAMILY, 12), bg="#CCCCCC", fg="black", command=stat_window.destroy).pack(pady=10)

    def descargar_archivo(self, listbox, folder):
        selection = listbox.curselection()
        if selection:
            file_selected = listbox.get(selection[0])
            file_path = os.path.join(folder, file_selected)
            dest = filedialog.asksaveasfilename(initialfile=file_selected, defaultextension=".txt",
                                                filetypes=[("Text files", "*.txt")])
            if dest:
                try:
                    shutil.copy(file_path, dest)
                    messagebox.showinfo("Descargar", f"Archivo descargado a {dest}")
                except Exception as e:
                    messagebox.showerror("Error", f"No se pudo descargar: {e}")
        else:
            messagebox.showwarning("Selección", "Seleccione un archivo.")

    def eliminar_archivo(self, listbox, text_widget, folder):
        selection = listbox.curselection()
        if selection:
            file_selected = listbox.get(selection[0])
            file_path = os.path.join(folder, file_selected)
            confirm = messagebox.askyesno("Eliminar", f"¿Eliminar la prueba '{file_selected}'?")
            if confirm:
                try:
                    os.remove(file_path)
                    listbox.delete(selection[0])
                    text_widget.config(state="normal")
                    text_widget.delete("1.0", tk.END)
                    text_widget.config(state="disabled")
                    messagebox.showinfo("Eliminar", "Archivo eliminado.")
                except Exception as e:
                    messagebox.showerror("Error", f"No se pudo eliminar: {e}")
        else:
            messagebox.showwarning("Selección", "Seleccione un archivo.")

    def eliminar_todo(self, folder, listbox, text_widget):
        confirm = messagebox.askyesno("Eliminar Todo", "¿Eliminar todas las pruebas?")
        if confirm:
            try:
                for f in os.listdir(folder):
                    if f.endswith(".txt"):
                        os.remove(os.path.join(folder, f))
                listbox.delete(0, tk.END)
                text_widget.config(state="normal")
                text_widget.delete("1.0", tk.END)
                text_widget.config(state="disabled")
                messagebox.showinfo("Eliminar Todo", "Todos los archivos eliminados.")
            except Exception as e:
                messagebox.showerror("Error", f"No se pudieron eliminar todos: {e}")

    # --------------------- TUTORIAL E INFORMACIÓN ---------------------
    def mostrar_tutorial_info(self):
        tutorial_win = tk.Toplevel(self.master)
        tutorial_win.title("Tutorial e Información")
        tutorial_win.config(bg=COLOR_FONDO)
        root_x = self.master.winfo_rootx()
        root_y = self.master.winfo_rooty()
        tutorial_win.geometry(f"700x500+{root_x}+{root_y}")
        notebook = ttk.Notebook(tutorial_win)
        notebook.pack(expand=True, fill="both")
        frame_tutorial = tk.Frame(notebook, bg=COLOR_FONDO)
        tutorial_text = ("Bienvenido a LearnPy!\n\n"
                         "Tutorial de Uso:\n"
                         "1. Regístrate en el formulario de inicio.\n"
                         "2. Selecciona el tipo de cuestionario (Normal o Personalizado).\n"
                         "3. Responde las preguntas y al finalizar se mostrará tu calificación y el detalle de respuestas.\n"
                         "4. Si eres docente, aprovecha las funciones exclusivas para personalizar cuestionarios.\n\n"
                         "¡Disfruta aprendiendo Python de forma interactiva!")
        tk.Label(frame_tutorial, text=tutorial_text, font=FONT_NORMAL, bg=COLOR_FONDO, justify="left", wraplength=680).pack(anchor="nw", padx=5, pady=5)
        notebook.add(frame_tutorial, text="Tutorial")
        frame_info = tk.Frame(notebook, bg=COLOR_FONDO)
        info_text = ("Información de LearnPy - Versión 1.1\n\n"
                     "Características:\n"
                     "• Cuestionarios interactivos para aprender Python.\n"
                     "• Exclusivo para docentes.\n"
                     "• Historial y estadísticas de desempeño.\n"
                     "• Ventanas emergentes personalizadas.\n"
                     "• Soporte para cuestionarios finales.\n\n"
                     "Autores:\n"
                     "Brahyam Velasquez, Jose Gomes, Jose Pereira, Samuel Meneses\n\n"
                     "Nota:\n"
                     "LearnPy está en constante desarrollo; se planean futuras mejoras en la interfaz y nuevas funciones.")
        tk.Label(frame_info, text=info_text, font=FONT_NORMAL, bg=COLOR_FONDO, justify="left", wraplength=680).pack(anchor="nw", padx=5, pady=5)
        notebook.add(frame_info, text="Información")

    # --------------------- ENVIAR SUGERENCIAS ---------------------
    def abrir_ventana_sugerencia(self):
        sug_win = tk.Toplevel(self.master)
        sug_win.title("Enviar Sugerencias")
        sug_win.config(bg=COLOR_FONDO)
        root_x = self.master.winfo_rootx()
        root_y = self.master.winfo_rooty()
        sug_win.geometry(f"600x400+{root_x}+{root_y}")
        tk.Label(sug_win, text="Escribe tu sugerencia a LearnPy:", font=FONT_NORMAL, bg=COLOR_FONDO).pack(padx=10, pady=10)
        text_area = tk.Text(sug_win, font=FONT_NORMAL, height=10, width=70)
        text_area.pack(padx=10, pady=10)
        def enviar():
            sugerencia = text_area.get("1.0", tk.END).strip()
            if sugerencia:
                try:
                    with open("sugerencias.txt", "a", encoding="utf-8") as f:
                        now = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
                        f.write(f"{now} - {sugerencia}\n")
                    messagebox.showinfo("Sugerencia Enviada", "¡Gracias por tu sugerencia!")
                    sug_win.destroy()
                except Exception as e:
                    messagebox.showerror("Error", f"No se pudo enviar la sugerencia: {e}")
            else:
                messagebox.showinfo("Sugerencia", "No se ingresó ninguna sugerencia.")
        tk.Button(sug_win, text="Enviar Sugerencias", font=FONT_NORMAL, bg=COLOR_BOTON, fg=COLOR_TEXTO,
                  command=enviar).pack(pady=10)

    # --------------------- VER SUGERENCIAS (Modo Docente) ---------------------
    def admin_ver_sugerencias(self):
        if not os.path.exists("sugerencias.txt"):
            messagebox.showinfo("Sugerencias", "No hay sugerencias registradas.")
            return
        try:
            with open("sugerencias.txt", "r", encoding="utf-8") as f:
                content = f.read()
            sug_win = tk.Toplevel(self.master)
            sug_win.title("Sugerencias")
            sug_win.config(bg=COLOR_FONDO)
            self.centrar_ventana(sug_win, 600, 400)
            text = tk.Text(sug_win, font=FONT_NORMAL)
            text.pack(expand=True, fill="both", padx=10, pady=10)
            text.insert(tk.END, content)
            text.config(state="disabled")
            tk.Button(sug_win, text="Cerrar", command=sug_win.destroy, font=FONT_NORMAL, bg="#CCCCCC", fg="black").pack(pady=10)
        except Exception as e:
            messagebox.showerror("Error", f"No se pudo abrir el archivo de sugerencias: {e}")

    # --------------------- OTRAS FUNCIONES ADMINISTRATIVAS ---------------------
    # (Aquí se mantienen las funciones de crear, editar, eliminar cuestionarios tal como en la versión original)
    # Para brevedad, se incluye la lógica completa a continuación:

    def admin_seleccionar_cuestionario(self):
        sel_window = tk.Toplevel(self.master)
        sel_window.title("Seleccionar Cuestionario")
        sel_window.config(bg=ADMIN_BG)
        self.centrar_ventana(sel_window, 450, 250)
        tk.Label(sel_window, text="Seleccione el cuestionario:", font=(FONT_FAMILY, 12), bg=ADMIN_BG, fg=ADMIN_FG).pack(pady=10)
        listbox = tk.Listbox(sel_window, font=(FONT_FAMILY, 12), width=30, height=6)
        listbox.pack(pady=5)
        for m in modulos.keys():
            listbox.insert(tk.END, m)
        def seleccionar():
            selection = listbox.curselection()
            if selection:
                self.modulo_actual = listbox.get(selection[0])
                self.custom_module_enabled = True
                messagebox.showinfo("Seleccionado", f"Cuestionario seleccionado: {self.modulo_actual}")
                sel_window.destroy()
            else:
                messagebox.showwarning("Selección", "Seleccione un cuestionario.")
        tk.Button(sel_window, text="Seleccionar", font=(FONT_FAMILY, 12), bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO,
                  command=seleccionar).pack(pady=10)

    def admin_toggle_cuestionario(self):
        self.custom_module_enabled = not self.custom_module_enabled
        self.custom_questionnaire_enabled = not self.custom_questionnaire_enabled
        if self.custom_module_enabled and self.custom_questionnaire_enabled:
            self.btn_toggle_custom.config(text="Desactivar Personalizaciones", bg=ADMIN_BTN_COLOR)
            messagebox.showinfo("Personalizaciones", "Personalizaciones ACTIVADAS.")
        else:
            self.btn_toggle_custom.config(text="Activar Personalizaciones", bg="red")
            messagebox.showinfo("Personalizaciones", "Personalizaciones DESACTIVADAS.")

    def admin_crear_cuestionario(self):
        crear_window = tk.Toplevel(self.master)
        crear_window.title("Crear Cuestionario Personalizado")
        crear_window.config(bg=ADMIN_BG)
        self.centrar_ventana(crear_window, 700, 650)
        title_frame = tk.Frame(crear_window, bg=ADMIN_BG)
        title_frame.pack(fill="x", padx=10, pady=10)
        tk.Label(title_frame, text="Nombre del Cuestionario:", font=(FONT_FAMILY, 14, "bold"), bg=ADMIN_BG, fg=ADMIN_FG).pack(side="left")
        entry_nombre = tk.Entry(title_frame, font=(FONT_FAMILY, 12), width=40)
        entry_nombre.pack(side="left", padx=10)
        cuestionario_temp = []
        form_frame = tk.Frame(crear_window, bg=ADMIN_BG)
        form_frame.pack(fill="both", expand=True, padx=10, pady=10)
        tk.Label(form_frame, text="Pregunta:", font=(FONT_FAMILY, 12, "bold"), bg=ADMIN_BG, fg=ADMIN_FG).grid(row=0, column=0, sticky="w", pady=2)
        entry_pregunta = tk.Entry(form_frame, font=(FONT_FAMILY, 12), width=80)
        entry_pregunta.grid(row=0, column=1, columnspan=3, pady=2, sticky="w")
        tk.Label(form_frame, text="Opción 1:", font=(FONT_FAMILY, 12), bg=ADMIN_BG, fg=ADMIN_FG).grid(row=1, column=0, sticky="w", pady=2)
        entry_resp1 = tk.Entry(form_frame, font=(FONT_FAMILY, 12), width=40)
        entry_resp1.grid(row=1, column=1, pady=2, sticky="w")
        tk.Label(form_frame, text="Opción 2:", font=(FONT_FAMILY, 12), bg=ADMIN_BG, fg=ADMIN_FG).grid(row=2, column=0, sticky="w", pady=2)
        entry_resp2 = tk.Entry(form_frame, font=(FONT_FAMILY, 12), width=40)
        entry_resp2.grid(row=2, column=1, pady=2, sticky="w")
        tk.Label(form_frame, text="Opción 3:", font=(FONT_FAMILY, 12), bg=ADMIN_BG, fg=ADMIN_FG).grid(row=3, column=0, sticky="w", pady=2)
        entry_resp3 = tk.Entry(form_frame, font=(FONT_FAMILY, 12), width=40)
        entry_resp3.grid(row=3, column=1, pady=2, sticky="w")
        tk.Label(form_frame, text="Respuesta Correcta:", font=(FONT_FAMILY, 12), bg=ADMIN_BG, fg=ADMIN_FG).grid(row=4, column=0, sticky="w", pady=2)
        combo_correcta = ttk.Combobox(form_frame, values=["Opción 1", "Opción 2", "Opción 3"],
                                       state="readonly", width=37, font=(FONT_FAMILY, 12))
        combo_correcta.grid(row=4, column=1, pady=2, sticky="w")
        combo_correcta.current(0)
        tk.Label(form_frame, text="Explicación si es Correcta:", font=(FONT_FAMILY, 12), bg=ADMIN_BG, fg=ADMIN_FG).grid(row=5, column=0, sticky="nw", pady=2)
        text_explicacion_correcta = tk.Text(form_frame, font=(FONT_FAMILY, 12), height=3, width=60)
        text_explicacion_correcta.grid(row=5, column=1, columnspan=3, pady=2, sticky="w")
        tk.Label(form_frame, text="Explicación si es Incorrecta:", font=(FONT_FAMILY, 12), bg=ADMIN_BG, fg=ADMIN_FG).grid(row=6, column=0, sticky="nw", pady=2)
        text_explicacion_incorrecta = tk.Text(form_frame, font=(FONT_FAMILY, 12), height=3, width=60)
        text_explicacion_incorrecta.grid(row=6, column=1, columnspan=3, pady=2, sticky="w")
        input_btn_frame = tk.Frame(crear_window, bg=ADMIN_BG)
        input_btn_frame.pack(fill="x", padx=10, pady=5)
        def agregar_pregunta():
            pregunta_text = entry_pregunta.get().strip()
            resp1 = entry_resp1.get().strip()
            resp2 = entry_resp2.get().strip()
            resp3 = entry_resp3.get().strip()
            explicacion_correcta = text_explicacion_correcta.get("1.0", tk.END).strip()
            explicacion_incorrecta = text_explicacion_incorrecta.get("1.0", tk.END).strip()
            if not pregunta_text or not resp1 or not resp2 or not resp3:
                messagebox.showerror("Error", "Debe completar la pregunta y las tres opciones.")
                return
            pregunta_dict = {
                "pregunta": pregunta_text,
                "respuestas": [resp1, resp2, resp3],
                "correcta": combo_correcta.current(),
                "explicacion_correcta": explicacion_correcta,
                "explicacion_incorrecta": explicacion_incorrecta
            }
            cuestionario_temp.append(pregunta_dict)
            entry_pregunta.delete(0, tk.END)
            entry_resp1.delete(0, tk.END)
            entry_resp2.delete(0, tk.END)
            entry_resp3.delete(0, tk.END)
            text_explicacion_correcta.delete("1.0", tk.END)
            text_explicacion_incorrecta.delete("1.0", tk.END)
            listbox_questions.insert(tk.END, pregunta_text)
        btn_agregar = tk.Button(input_btn_frame, text="Agregar Pregunta", font=(FONT_FAMILY, 12, "bold"),
                                bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO, relief="ridge", borderwidth=2, command=agregar_pregunta)
        btn_agregar.pack(side="left", padx=5, expand=True, fill="x")
        def vista_previa():
            pregunta_text = entry_pregunta.get().strip()
            resp1 = entry_resp1.get().strip()
            resp2 = entry_resp2.get().strip()
            resp3 = entry_resp3.get().strip()
            explicacion_correcta = text_explicacion_correcta.get("1.0", tk.END).strip()
            explicacion_incorrecta = text_explicacion_incorrecta.get("1.0", tk.END).strip()
            if not pregunta_text:
                messagebox.showerror("Error", "Ingrese la pregunta para vista previa.")
                return
            preview_window = tk.Toplevel(crear_window)
            preview_window.title("Vista Previa de la Pregunta")
            preview_window.config(bg=ADMIN_BG)
            self.centrar_ventana(preview_window, 500, 400)
            tk.Label(preview_window, text="Pregunta:", font=(FONT_FAMILY, 12, "bold"),
                     bg=ADMIN_BG, fg=ADMIN_FG, relief="ridge", borderwidth=2).pack(anchor="w", padx=10, pady=5)
            tk.Label(preview_window, text=pregunta_text, font=(FONT_FAMILY, 12),
                     bg=ADMIN_BG, fg=ADMIN_FG, wraplength=480, justify="left",
                     relief="ridge", borderwidth=2).pack(anchor="w", padx=10)
            tk.Label(preview_window, text="Opciones:", font=(FONT_FAMILY, 12, "bold"),
                     bg=ADMIN_BG, fg=ADMIN_FG, relief="ridge", borderwidth=2).pack(anchor="w", padx=10, pady=5)
            for i, opt in enumerate([resp1, resp2, resp3]):
                tk.Label(preview_window, text=f"{i+1}. {opt}", font=(FONT_FAMILY, 12),
                         bg=ADMIN_BG, fg=ADMIN_FG, wraplength=480, justify="left",
                         relief="ridge", borderwidth=2).pack(anchor="w", padx=10)
            tk.Label(preview_window, text="Explicación si es Correcta:", font=(FONT_FAMILY, 12, "bold"),
                     bg=ADMIN_BG, fg=ADMIN_FG, relief="ridge", borderwidth=2).pack(anchor="w", padx=10, pady=5)
            tk.Label(preview_window, text=explicacion_correcta, font=(FONT_FAMILY, 12),
                     bg=ADMIN_BG, fg=ADMIN_FG, wraplength=480, justify="left",
                     relief="ridge", borderwidth=2).pack(anchor="w", padx=10)
            tk.Label(preview_window, text="Explicación si es Incorrecta:", font=(FONT_FAMILY, 12, "bold"),
                     bg=ADMIN_BG, fg=ADMIN_FG, relief="ridge", borderwidth=2).pack(anchor="w", padx=10, pady=5)
            tk.Label(preview_window, text=explicacion_incorrecta, font=(FONT_FAMILY, 12),
                     bg=ADMIN_BG, fg=ADMIN_FG, wraplength=480, justify="left",
                     relief="ridge", borderwidth=2).pack(anchor="w", padx=10)
        btn_vista_previa = tk.Button(input_btn_frame, text="Vista Previa", command=vista_previa,
                                     font=(FONT_FAMILY, 12, "bold"), bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO,
                                     relief="ridge", borderwidth=2)
        btn_vista_previa.pack(side="left", padx=5, expand=True, fill="x")
        listbox_frame = tk.Frame(crear_window, bg=ADMIN_BG)
        listbox_frame.pack(fill="both", padx=10, pady=10, expand=True)
        tk.Label(listbox_frame, text="Preguntas Agregadas:", font=(FONT_FAMILY, 12, "bold"),
                 bg=ADMIN_BG, fg=ADMIN_FG, relief="ridge", borderwidth=2).pack(anchor="w")
        listbox_questions = tk.Listbox(listbox_frame, font=(FONT_FAMILY, 12), height=5, relief="ridge", borderwidth=2)
        listbox_questions.pack(fill="both", padx=5, pady=5, expand=True)
        action_btn_frame = tk.Frame(crear_window, bg=ADMIN_BG)
        action_btn_frame.pack(fill="x", padx=10, pady=10)
        def eliminar_pregunta():
            selection = listbox_questions.curselection()
            if not selection:
                messagebox.showwarning("Advertencia", "Seleccione una pregunta para eliminar.")
                return
            index = selection[0]
            listbox_questions.delete(index)
            del cuestionario_temp[index]
        btn_eliminar = tk.Button(action_btn_frame, text="Eliminar Pregunta Seleccionada",
                                 font=(FONT_FAMILY, 12, "bold"), bg=ADMIN_REINICIAR_COLOR, fg=COLOR_TEXTO,
                                 relief="ridge", borderwidth=2, command=eliminar_pregunta)
        btn_eliminar.pack(side="left", padx=5, expand=True, fill="x")
        def guardar_cuestionario_form():
            nombre = entry_nombre.get().strip()
            if not nombre:
                messagebox.showerror("Error", "Ingrese el nombre del cuestionario.")
                return
            if not cuestionario_temp:
                messagebox.showerror("Error", "No se ha agregado ninguna pregunta.")
                return
            custom_cuestionarios[nombre] = cuestionario_temp.copy()
            guardar_cuestionarios_custom()
            messagebox.showinfo("Guardar", f"Cuestionario '{nombre}' guardado exitosamente.")
            crear_window.destroy()
        btn_guardar = tk.Button(action_btn_frame, text="Guardar Cuestionario",
                                font=(FONT_FAMILY, 12, "bold"), bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO,
                                relief="ridge", borderwidth=2, command=guardar_cuestionario_form)
        btn_guardar.pack(side="left", padx=5, expand=True, fill="x")

    def admin_editar_cuestionario(self):
        if not custom_cuestionarios:
            messagebox.showinfo("Editar", "No hay cuestionarios personalizados para editar.")
            return
        edit_window = tk.Toplevel(self.master)
        edit_window.title("Editar Cuestionario Personalizado")
        edit_window.config(bg=ADMIN_BG)
        self.centrar_ventana(edit_window, 600, 500)
        left_frame = tk.Frame(edit_window, bg=ADMIN_BG)
        left_frame.pack(side="left", fill="y", padx=10, pady=10)
        right_frame = tk.Frame(edit_window, bg=ADMIN_BG)
        right_frame.pack(side="right", fill="both", expand=True, padx=10, pady=10)
        tk.Label(left_frame, text="Cuestionarios Disponibles", font=(FONT_FAMILY, 12, "bold"), bg=ADMIN_BG, fg=ADMIN_FG).pack(pady=5)
        listbox = tk.Listbox(left_frame, font=(FONT_FAMILY, 12), width=30, height=15, relief="ridge", borderwidth=2)
        listbox.pack(pady=5)
        for key in custom_cuestionarios.keys():
            listbox.insert(tk.END, key)
        text_widget = tk.Text(right_frame, wrap="word", font=(FONT_FAMILY, 12), relief="ridge", borderwidth=2)
        text_widget.pack(fill="both", expand=True)
        def cargar_cuestionario(event):
            selection = listbox.curselection()
            if selection:
                key = listbox.get(selection[0])
                content = json.dumps(custom_cuestionarios[key], indent=2)
                text_widget.config(state="normal")
                text_widget.delete("1.0", tk.END)
                text_widget.insert(tk.END, content)
                text_widget.config(state="disabled")
        listbox.bind("<<ListboxSelect>>", cargar_cuestionario)
        def guardar_edicion():
            selection = listbox.curselection()
            if not selection:
                messagebox.showwarning("Guardar", "Seleccione un cuestionario para editar.")
                return
            key = listbox.get(selection[0])
            text_widget.config(state="normal")
            new_content = text_widget.get("1.0", tk.END)
            text_widget.config(state="disabled")
            try:
                new_data = json.loads(new_content)
                if not isinstance(new_data, list):
                    raise ValueError("Debe ser una lista de preguntas.")
            except Exception as e:
                messagebox.showerror("Error", f"Formato JSON inválido: {e}")
                return
            custom_cuestionarios[key] = new_data
            guardar_cuestionarios_custom()
            messagebox.showinfo("Editar", f"Cuestionario '{key}' actualizado.")
            edit_window.destroy()
        tk.Button(edit_window, text="Guardar Edición", font=(FONT_FAMILY, 12), bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO,
                  relief="ridge", borderwidth=2, command=guardar_edicion).pack(pady=10)

    def admin_eliminar_cuestionario_personalizado(self):
        if not custom_cuestionarios:
            messagebox.showinfo("Eliminar", "No hay cuestionarios personalizados para eliminar.")
            return
        del_window = tk.Toplevel(self.master)
        del_window.title("Eliminar Cuestionario Personalizado")
        del_window.config(bg=ADMIN_BG)
        self.centrar_ventana(del_window, 400, 300)
        tk.Label(del_window, text="Seleccione el cuestionario a eliminar:", font=(FONT_FAMILY, 12, "bold"),
                 bg=ADMIN_BG, fg=ADMIN_FG).pack(pady=10)
        listbox = tk.Listbox(del_window, font=(FONT_FAMILY, 12), width=40, height=6, relief="ridge", borderwidth=2)
        listbox.pack(pady=5)
        for q in custom_cuestionarios.keys():
            listbox.insert(tk.END, q)
        def eliminar():
            selection = listbox.curselection()
            if selection:
                q_selected = listbox.get(selection[0])
                confirm = messagebox.askyesno("Confirmar", f"¿Eliminar '{q_selected}'?")
                if confirm:
                    try:
                        del custom_cuestionarios[q_selected]
                        guardar_cuestionarios_custom()
                        messagebox.showinfo("Eliminado", f"Cuestionario '{q_selected}' eliminado.")
                        listbox.delete(selection[0])
                    except Exception as e:
                        messagebox.showerror("Error", f"No se pudo eliminar: {e}")
            else:
                messagebox.showwarning("Eliminar", "Seleccione un cuestionario.")
        tk.Button(del_window, text="Eliminar", font=(FONT_FAMILY, 12), bg=ADMIN_REINICIAR_COLOR, fg=COLOR_TEXTO,
                  relief="ridge", borderwidth=2, command=eliminar).pack(pady=10)

    # --------------------- TEMPORIZADOR Y PREGUNTAS ---------------------
    def mostrar_interfaz_cuestionario(self, subtitulo_text):
        self.master.config(bg=COLOR_FONDO)
        for widget in self.master.winfo_children():
            widget.destroy()
        header = tk.Frame(self.master, bg=COLOR_FONDO)
        header.pack(pady=10, fill="x")
        self.crear_label(header, "LearnPy", font=(FONT_FAMILY, 30, "bold")).pack(anchor="center")
        self.subtitulo = self.crear_label(header, subtitulo_text, font=FONT_SUBTITLE)
        self.subtitulo.pack(anchor="center")
        tk.Frame(self.master, height=20, bg=COLOR_FONDO).pack()
        self.etiqueta_pregunta = self.crear_label(self.master, "", font=(FONT_FAMILY, 17), wraplength=550)
        self.etiqueta_pregunta.pack(anchor="center", padx=20)
        response_frame = tk.Frame(self.master, bg=COLOR_FONDO, width=550, height=100)
        response_frame.pack(pady=10)
        response_frame.pack_propagate(False)
        response_frame.grid_rowconfigure(0, minsize=40)
        response_frame.grid_rowconfigure(1, minsize=40)
        self.combo_respuestas = ttk.Combobox(response_frame, state="readonly", width=30)
        self.combo_respuestas.grid(row=0, column=0, pady=(5,0))
        self.boton_aceptar = tk.Button(response_frame, text="Aceptar", command=self.verificar_respuesta,
                                       font=(FONT_FAMILY, 16, "bold"), bg=COLOR_BOTON, fg=COLOR_TEXTO)
        self.boton_aceptar.grid(row=1, column=0, pady=(15,5))
        if self.admin_mode and self.skip_enabled:
            tk.Button(response_frame, text="Saltar Pregunta", font=(FONT_FAMILY, 12, "bold"),
                      bg=ADMIN_BTN_COLOR, fg=COLOR_TEXTO, command=self.saltar_pregunta_admin).grid(row=1, column=1, padx=10)
        if self.timer_mode:
            self.timer_label = self.crear_label(self.master, "", font=(FONT_FAMILY, 14, "bold"))
            self.timer_label.pack(pady=(0,10))
            self.update_timer()
        self.progress = ttk.Progressbar(self.master, orient="horizontal", mode="determinate", maximum=self.total_preguntas)
        self.progress.pack(side="bottom", fill="x", padx=20, pady=10)
        self.mostrar_pregunta()

    def mostrar_pregunta(self):
        if self.is_custom:
            if self.pregunta_actual < self.total_preguntas:
                pregunta = self.preguntas_modulo[self.pregunta_actual]
                self.etiqueta_pregunta.config(text=pregunta["pregunta"])
                respuestas = pregunta["respuestas"][:]
                random.shuffle(respuestas)
                self.combo_respuestas['values'] = ["Seleccionar opción"] + respuestas
                self.combo_respuestas.current(0)
                self.animate_progress(self.pregunta_actual)
            else:
                self.mostrar_resultados()
        else:
            if self.pregunta_global < self.total_preguntas:
                if self.pregunta_actual < len(self.preguntas_modulo):
                    pregunta = self.preguntas_modulo[self.pregunta_actual]
                    self.etiqueta_pregunta.config(text=pregunta["pregunta"])
                    respuestas = pregunta["respuestas"][:]
                    random.shuffle(respuestas)
                    self.combo_respuestas['values'] = ["Seleccionar opción"] + respuestas
                    self.combo_respuestas.current(0)
                    self.animate_progress(self.pregunta_global)
                else:
                    if self.modulo_actual != "Examen Final":
                        next_module_index = list(modulos.keys()).index(self.modulo_actual) + 1
                        if next_module_index < len(modulos):
                            self.modulo_actual = list(modulos.keys())[next_module_index]
                        else:
                            self.modulo_actual = "Examen Final"
                        messagebox.showinfo("Cambio de Módulo", f"Ahora estás en el módulo: {self.modulo_actual}")
                        self.pregunta_actual = 0
                        self.preguntas_modulo = modulos[self.modulo_actual]
                        self.subtitulo.config(text=f"Módulo: {self.modulo_actual}")
                        self.mostrar_pregunta()
                    else:
                        self.mostrar_resultados()
            else:
                self.mostrar_resultados()

    def verificar_respuesta(self):
        if self.combo_respuestas.current() == 0:
            messagebox.showerror("Error", "Seleccione una respuesta válida.")
            return
        pregunta = self.preguntas_modulo[self.pregunta_actual]
        respuesta_sel = self.combo_respuestas.get()
        correcta = pregunta["respuestas"][pregunta["correcta"]]
        if respuesta_sel == correcta:
            exp_text = pregunta.get("explicacion_correcta", "Respuesta correcta")
            self.aciertos += 1
        else:
            exp_text = pregunta.get("explicacion_incorrecta", "Respuesta incorrecta")
            self.incorrectos += 1
        info = {
            "pregunta": pregunta["pregunta"],
            "respuesta_seleccionada": respuesta_sel,
            "respuesta_correcta": correcta,
            "explicacion": exp_text,
            "es_correcta": respuesta_sel == correcta
        }
        self.respuestas_usuario.append(info)
        if self.is_custom:
            self.pregunta_actual += 1
        else:
            self.pregunta_actual += 1
            self.pregunta_global += 1
        self.mostrar_pregunta()

    def mostrar_resultados(self):
        for widget in self.master.winfo_children():
            widget.destroy()
        self.guardar_en_historial()
        total = self.aciertos + self.incorrectos
        porcentaje = (self.aciertos / total) * 100 if total > 0 else 0
        datos = (f"Estudiante: {self.usuario['nombre']} | Año: {self.usuario['año']} | "
                 f"Mención: {self.usuario['Mencion']} | Sección: {self.usuario['sección']}")
        self.crear_label(self.master, datos, font=(FONT_FAMILY, 14, "bold")).pack(pady=(10,5), fill="x", padx=10)
        self.crear_label(self.master, "Calificación Final", font=(FONT_FAMILY, 18, "bold")).pack(pady=(10,5), fill="x", padx=10)
        self.crear_label(self.master, f"Puntaje: {porcentaje:.2f}%\nAciertos: {self.aciertos} | Errores: {self.incorrectos}",
                         font=(FONT_FAMILY, 20, "bold"), fg="blue").pack(pady=5, fill="x", padx=10)
        tk.Button(self.master, text="Nueva Prueba", command=self.nueva_prueba,
                  font=(FONT_FAMILY, 16, "bold"), bg=BTN_REINICIAR_COLOR, fg="white").pack(pady=5)
        tk.Button(self.master, text="Mostrar Detalle", command=self.mostrar_detalle,
                  font=(FONT_FAMILY, 16, "bold"), bg=BTN_CONTINUAR_COLOR, fg="white").pack(pady=5)
        messagebox.showinfo("Gracias", "¡Gracias por usar LearnPy!")

    def guardar_en_historial(self, nueva_prueba=False):
        total = self.aciertos + self.incorrectos
        porcentaje = (self.aciertos / total) * 100 if total > 0 else 0
        now = datetime.datetime.now()
        documents_folder = os.path.join(os.path.expanduser("~"), "Documents")
        historial_folder = os.path.join(documents_folder, "LearnPyHistorial")
        if not os.path.exists(historial_folder):
            os.makedirs(historial_folder)
        tipo = self.cuestionario_personalizado if self.is_custom else "Normal"
        file_name = f"resultados_{self.usuario['nombre']}_{self.usuario['año']}_{self.usuario['Mencion']}_{self.usuario['sección']}_{tipo}_{now.strftime('%Y%m%d_%H%M%S')}.txt"
        full_path = os.path.join(historial_folder, file_name)
        try:
            with open(full_path, "w", encoding="utf-8") as f:
                f.write("Resultados del Examen LearnPy\n")
                f.write("=====================================\n\n")
                f.write(f"Estudiante: {self.usuario['nombre']}\n")
                f.write(f"Año: {self.usuario['año']}\n")
                f.write(f"Mención: {self.usuario['Mencion']}\n")
                f.write(f"Sección: {self.usuario['sección']}\n\n")
                f.write(f"Tipo de examen: {tipo}\n")
                f.write(f"Puntaje: {porcentaje:.2f}%\n")
                f.write(f"Aciertos: {self.aciertos}\n")
                f.write(f"Errores: {self.incorrectos}\n\n")
                f.write("Detalle de respuestas:\n")
                for resp in self.respuestas_usuario:
                    f.write(f"\nPregunta: {resp['pregunta']}\n")
                    f.write(f"Respuesta Seleccionada: {resp['respuesta_seleccionada']}\n")
                    f.write(f"Respuesta Correcta: {resp['respuesta_correcta']}\n")
                    f.write(f"Explicación: {resp['explicacion']}\n")
        except Exception as e:
            messagebox.showerror("Error", f"No se pudo guardar el historial: {e}")

    def nueva_prueba(self):
        self.menu_registro()

    def mostrar_detalle(self):
        detalle_window = tk.Toplevel(self.master)
        detalle_window.title("Detalle de Respuestas")
        detalle_window.config(bg=COLOR_FONDO)
        self.centrar_ventana(detalle_window, 600, 400)
        text_area = tk.Text(detalle_window, font=(FONT_FAMILY, 12))
        text_area.pack(expand=True, fill="both", padx=10, pady=10)
        for resp in self.respuestas_usuario:
            text_area.insert(tk.END, f"Pregunta: {resp['pregunta']}\n")
            text_area.insert(tk.END, f"Respuesta Seleccionada: {resp['respuesta_seleccionada']}\n")
            text_area.insert(tk.END, f"Respuesta Correcta: {resp['respuesta_correcta']}\n")
            text_area.insert(tk.END, f"Explicación: {resp['explicacion']}\n")
            text_area.insert(tk.END, "------------------------------------------\n")
        tk.Button(detalle_window, text="Cerrar", command=detalle_window.destroy,
                  font=(FONT_FAMILY, 12), bg="#CCCCCC", fg="black").pack(pady=10)

    # --------------------- TUTORIAL E INFORMACIÓN (se mantiene el texto original) ---------------------
    def mostrar_tutorial_info(self):
        tutorial_win = tk.Toplevel(self.master)
        tutorial_win.title("Tutorial e Información")
        tutorial_win.config(bg=COLOR_FONDO)
        root_x = self.master.winfo_rootx()
        root_y = self.master.winfo_rooty()
        tutorial_win.geometry(f"700x500+{root_x}+{root_y}")
        notebook = ttk.Notebook(tutorial_win)
        notebook.pack(expand=True, fill="both")
        frame_tutorial = tk.Frame(notebook, bg=COLOR_FONDO)
        tutorial_text = ("Bienvenido a LearnPy!\n\n"
                         "Tutorial de Uso:\n"
                         "1. Regístrate en el formulario de inicio.\n"
                         "2. Selecciona el tipo de cuestionario (Normal o Personalizado).\n"
                         "3. Responde las preguntas y al finalizar se mostrará tu calificación y el detalle de respuestas.\n"
                         "4. Si eres docente, aprovecha las funciones exclusivas para personalizar cuestionarios.\n\n"
                         "¡Disfruta aprendiendo Python de forma interactiva!")
        tk.Label(frame_tutorial, text=tutorial_text, font=FONT_NORMAL, bg=COLOR_FONDO, justify="left", wraplength=680).pack(anchor="nw", padx=5, pady=5)
        notebook.add(frame_tutorial, text="Tutorial")
        frame_info = tk.Frame(notebook, bg=COLOR_FONDO)
        info_text = ("Información de LearnPy - Versión 1.1\n\n"
                     "Características:\n"
                     "• Cuestionarios interactivos para aprender Python.\n"
                     "• Exclusivo para docentes.\n"
                     "• Historial y estadísticas de desempeño.\n"
                     "• Ventanas emergentes personalizadas.\n"
                     "• Soporte para cuestionarios finales.\n\n"
                     "Autores:\n"
                     "Brahyam Velasquez, Jose Gomes, Jose Pereira, Samuel Meneses\n\n"
                     "Nota:\n"
                     "LearnPy está en constante desarrollo; se planean futuras mejoras en la interfaz y nuevas funciones.")
        tk.Label(frame_info, text=info_text, font=FONT_NORMAL, bg=COLOR_FONDO, justify="left", wraplength=680).pack(anchor="nw", padx=5, pady=5)
        notebook.add(frame_info, text="Información")

    # --------------------- ENVIAR SUGERENCIAS ---------------------
    def abrir_ventana_sugerencia(self):
        sug_win = tk.Toplevel(self.master)
        sug_win.title("Enviar Sugerencias")
        sug_win.config(bg=COLOR_FONDO)
        root_x = self.master.winfo_rootx()
        root_y = self.master.winfo_rooty()
        sug_win.geometry(f"600x400+{root_x}+{root_y}")
        tk.Label(sug_win, text="Escribe tu sugerencia a LearnPy:", font=FONT_NORMAL, bg=COLOR_FONDO).pack(padx=10, pady=10)
        text_area = tk.Text(sug_win, font=FONT_NORMAL, height=10, width=70)
        text_area.pack(padx=10, pady=10)
        def enviar():
            sugerencia = text_area.get("1.0", tk.END).strip()
            if sugerencia:
                try:
                    with open("sugerencias.txt", "a", encoding="utf-8") as f:
                        now = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
                        f.write(f"{now} - {sugerencia}\n")
                    messagebox.showinfo("Sugerencia Enviada", "¡Gracias por tu sugerencia!")
                    sug_win.destroy()
                except Exception as e:
                    messagebox.showerror("Error", f"No se pudo enviar la sugerencia: {e}")
            else:
                messagebox.showinfo("Sugerencia", "No se ingresó ninguna sugerencia.")
        tk.Button(sug_win, text="Enviar Sugerencias", font=FONT_NORMAL, bg=COLOR_BOTON, fg=COLOR_TEXTO,
                  command=enviar).pack(pady=10)

    # --------------------- FIN DE MÉTODOS ---------------------
    # (Las funciones relacionadas con la administración de cuestionarios se mantienen completas tal como se muestra)
    # Para resumir, aquí se incluyen: admin_ver_sugerencias, etc.
    def admin_ver_estadisticas(self):
        historial_folder = os.path.join(os.path.expanduser("~"), "Documents", "LearnPyHistorial")
        if not os.path.exists(historial_folder):
            messagebox.showinfo("Estadísticas", "No hay pruebas guardadas.")
            return
        scores = []
        for filename in os.listdir(historial_folder):
            if filename.endswith(".txt"):
                try:
                    with open(os.path.join(historial_folder, filename), "r", encoding="utf-8") as f:
                        for line in f:
                            if "Puntaje:" in line:
                                parts = line.split(":")
                                if len(parts) > 1:
                                    score_str = parts[1].strip().replace("%", "")
                                    try:
                                        score = float(score_str)
                                        scores.append(score)
                                    except:
                                        pass
                                break
                except:
                    pass
        if not scores:
            messagebox.showinfo("Estadísticas", "No se encontraron puntajes.")
            return
        total_tests = len(scores)
        avg_score = sum(scores) / total_tests
        best_score = max(scores)
        worst_score = min(scores)
        stats_msg = (f"Total de pruebas: {total_tests}\nPuntaje Promedio: {avg_score:.2f}%\n"
                     f"Mejor Puntaje: {best_score:.2f}%\nPeor Puntaje: {worst_score:.2f}%")
        stat_window = tk.Toplevel(self.master)
        stat_window.title("Estadísticas de Pruebas - LearnPy")
        stat_window.config(bg=ADMIN_BG)
        self.centrar_ventana(stat_window, 400, 200)
        tk.Label(stat_window, text=stats_msg, font=(FONT_FAMILY, 12), bg=ADMIN_BG, fg=ADMIN_FG).pack(pady=20)
        tk.Button(stat_window, text="Cerrar", font=(FONT_FAMILY, 12), bg="#CCCCCC", fg="black",
                  command=stat_window.destroy).pack(pady=10)

# ========================= EJECUCIÓN PRINCIPAL =========================
if __name__ == "__main__":
    root = tk.Tk()
    app = AplicacionCuestionario(root)
    root.mainloop()
