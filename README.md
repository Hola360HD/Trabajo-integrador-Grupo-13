# Trabajo-integrador-Grupo-3


import tkinter as tk
from tkinter import messagebox

# Archivo: ventana_main.py
# Propósito: interfaz simple para crear usuarios, listar usuarios y abrir ventanas de usuario
# cada ventana de usuario permite enviar mensajes y ver la bandeja con scroll y botón de refrescar.

# Clase que representa un usuario (nodo principal)
class persona:
    def _init_(self, nombre, mail):
        # nombre: nombre visible del usuario
        # mail: identificador único (clave en el diccionario 'usuarios')
        self.nombre = nombre
        self.mail = mail
        # bandeja_entrada: gestor de mensajes del usuario (objeto BandejaEntrada)
        self.bandeja_entrada = BandejaEntrada()  # Ahora es un objeto, no una lista

# Clase que representa un mensaje simple
class mensaje:
    def _init_(self, remitente, destinatario, asunto, cuerpo):
        # remitente/destinatario: emails (strings)
        # asunto: texto breve
        # cuerpo: contenido del mensaje
        self.remitente = remitente
        self.destinatario = destinatario
        self.asunto = asunto
        self.cuerpo = cuerpo

# Clase que gestiona la lista de mensajes de un usuario
class BandejaEntrada:
    def _init_(self):
        # lista interna con los mensajes en orden de llegada
        self.mensajes = []

    def agregar_mensaje(self, mensaje):
        # Añade un mensaje al final de la bandeja
        self.mensajes.append(mensaje)

    def ver_todos(self):
        # Devuelve la lista de mensajes (copia no necesaria aquí)
        return self.mensajes

    def _len_(self):
        # Permite usar len(bandeja)
        return len(self.mensajes)

    def _getitem_(self, idx):
        # Acceso por índice (bandeja[i])
        return self.mensajes[idx]

# Clase para enviar mensajes (encapsula el remitente)
class Enviar_mensaje:
    def _init_(self, remitente_mail):
        self.remitente_mail = remitente_mail

    def enviar(self, destinatario_mail, asunto, cuerpo):
        # Verifica que el destinatario exista en el sistema global 'usuarios'
        if destinatario_mail not in usuarios:
            return "El destinatario no existe."
        remitente = usuarios[self.remitente_mail]
        destinatario = usuarios[destinatario_mail]
        # Crea el objeto mensaje y lo agrega a la bandeja del destinatario
        msg = mensaje(remitente.mail, destinatario.mail, asunto, cuerpo)
        destinatario.bandeja_entrada.agregar_mensaje(msg)
        return f"Mensaje enviado a {destinatario.mail}"


# Diccionario global que almacena los usuarios: mail -> persona
usuarios = {}

# Función para refrescar la lista visual de usuarios en la ventana principal
def refrescar_lista_visual():
    # Borra y vuelve a insertar todos los usuarios en el Listbox
    listbox_usuarios.delete(0, tk.END)
    for m, u in usuarios.items():
        listbox_usuarios.insert(tk.END, f"{u.nombre}  <{m}>")

# Crea un nuevo usuario con los valores de los campos en la ventana principal
def crear_usuario():
    Nombre = nombre.get().strip()
    Mail = mail.get().strip()
    # Validaciones básicas: campos no vacíos, mail único y nombre único
    if Nombre == "" or Mail == "":
        resultado_label.config(text="Nombre y Mail no pueden estar vacíos")
        return
    if Mail in usuarios:
        resultado_label.config(text="Ese mail ya está registrado.")
        return
    if any(u.nombre == Nombre for u in usuarios.values()):
        resultado_label.config(text="Ese Nombre ya está registrado.")
        return
    # Crear y almacenar usuario
    user = persona(Nombre, Mail)
    usuarios[Mail] = user
    resultado_label.config(text=f"Usuario creado: {user.nombre}, {user.mail}")
    # Actualizar la lista visual para que aparezca inmediatamente
    refrescar_lista_visual()

# Iniciar sesión usando los campos de texto (nombre + mail)
def ingresar_usuario():
    Nombre = nombre.get().strip()
    Mail = mail.get().strip()
    # Comprueba que el mail exista y que el nombre coincida
    if Mail in usuarios and usuarios[Mail].nombre == Nombre:
        resultado_label.config(text=f"Bienvenido {Nombre}")
        # Abre una ventana nueva para el usuario sin cerrar la principal
        abrir_ventana_usuario(usuarios[Mail])
    else:
        resultado_label.config(text="Usuario o mail incorrecto, o no registrado.")

# Abrir sesión a partir de la selección en el Listbox de usuarios
def abrir_sesion_seleccionada():
    sel = listbox_usuarios.curselection()
    if not sel:
        messagebox.showwarning("Seleccione usuario", "Seleccione un usuario para iniciar sesión.")
        return
    texto = listbox_usuarios.get(sel[0])
    # Extrae el mail que está entre '<' y '>' en la línea del Listbox
    if "<" in texto and ">" in texto:
        mail_sel = texto.split("<",1)[1].split(">",1)[0]
    else:
        mail_sel = texto.split()[-1].strip()
    usuario_actual = usuarios.get(mail_sel)
    if usuario_actual is None:
        messagebox.showerror("Error", "Usuario seleccionado no encontrado.")
        return
    abrir_ventana_usuario(usuario_actual)

# Abre una ventana Toplevel para el usuario activo (ver/mandar mensajes)
def abrir_ventana_usuario(usuario_actual):
    ventana_user = tk.Toplevel(Ventana)
    ventana_user.title(f"Correo - {usuario_actual.nombre}")
    ventana_user.geometry("400x700+800+250")
    ventana_user.config(bg="Black")

    # Cabecera con el nombre del usuario
    tk.Label(ventana_user, text=f"Bienvenido {usuario_actual.nombre}", bg="black", fg="white", font=("Arial", 14)).pack(pady=10)

    # Area de mensajes con scrollbar (Text widget)
    frame_mensajes = tk.Frame(ventana_user, bg="black")
    frame_mensajes.pack(pady=10, fill="both", expand=True)

    scrollbar = tk.Scrollbar(frame_mensajes)
    scrollbar.pack(side="right", fill="y")

    mensajes_text_widget = tk.Text(
        frame_mensajes,
        bg="black",
        fg="lightgreen",
        wrap="word",
        yscrollcommand=scrollbar.set,
        height=10
    )
    mensajes_text_widget.pack(side="left", fill="both", expand=True)
    scrollbar.config(command=mensajes_text_widget.yview)

    # Función interna que carga/actualiza la bandeja en el Text widget
    def cargar_bandeja():
        mensajes_text_widget.config(state="normal")   # permitir escritura temporalmente
        mensajes_text_widget.delete("1.0", "end")     # limpiar contenido previo
        bandeja = usuario_actual.bandeja_entrada
        try:
            if len(bandeja) > 0:
                # Inserta cada mensaje con formato simple
                for msg in bandeja.ver_todos():
                    mensajes_text_widget.insert(
                        "end",
                        f"De: {msg.remitente}\nAsunto: {msg.asunto}\n{msg.cuerpo}\n---\n"
                    )
            else:
                mensajes_text_widget.insert("end", "No tienes mensajes nuevos.")
        except Exception as e:
            # Mostrar error en el widget si ocurre algo inesperado
            mensajes_text_widget.insert("end", f"Error al cargar mensajes: {e}\n")
        mensajes_text_widget.config(state="disabled")  # dejar solo lectura

    # Carga inicial de la bandeja
    cargar_bandeja()

    # Botón para refrescar los mensajes sin cerrar la ventana
    refresh_btn = tk.Button(ventana_user, text="Refrescar mensajes", bg="lightgrey", command=cargar_bandeja)
    refresh_btn.pack(pady=6)

    # Campos para componer y enviar un mensaje
    tk.Label(ventana_user, text="Destinatario (mail):", bg="grey", fg="white").pack(pady=5)
    destinatario_entry = tk.Entry(ventana_user)
    destinatario_entry.pack(pady=5, ipadx=10, ipady=5)

    tk.Label(ventana_user, text="Asunto:", bg="grey", fg="white").pack(pady=5)
    asunto_entry = tk.Entry(ventana_user)
    asunto_entry.pack(pady=5, ipadx=10, ipady=5)

    tk.Label(ventana_user, text="Cuerpo:", bg="grey", fg="white").pack(pady=5)
    cuerpo_entry = tk.Entry(ventana_user)
    cuerpo_entry.pack(pady=5, ipadx=10, ipady=5)

    resultado_envio = tk.Label(ventana_user, text="", bg="black", fg="yellow")
    resultado_envio.pack(pady=5)

    # Envía el mensaje usando la clase Enviar_mensaje y actualiza vista
    def enviar_mensaje_usuario():
        destinatario_mail = destinatario_entry.get().strip()
        asunto = asunto_entry.get().strip()
        cuerpo = cuerpo_entry.get().strip()
        em = Enviar_mensaje(usuario_actual.mail)
        resultado = em.enviar(destinatario_mail, asunto, cuerpo)
        resultado_envio.config(text=resultado)
        # Refrescar la vista para mostrar cambios (si aplica)
        cargar_bandeja()
        refrescar_lista_visual()

    enviar_btn = tk.Button(ventana_user, text="Enviar Mensaje", bg="lightblue", font=("Arial", 12), command=enviar_mensaje_usuario)
    enviar_btn.pack(pady=10)

    # Botón para cerrar solo esta ventana de usuario
    cerrar_sesion_btn = tk.Button(ventana_user, text="Cerrar sesión", bg="red", fg="white", font=("Arial", 12), command=ventana_user.destroy)
    cerrar_sesion_btn.pack(pady=10)

# Función principal que crea la ventana de gestión de usuarios
def main():
    global Ventana, nombre, mail, resultado_label, listbox_usuarios
    Ventana = tk.Tk()
    Ventana.title("Correo")
    Ventana.geometry("400x600+600+150")
    Ventana.config(bg="Black")

    # Botones y campos para crear usuario o iniciar sesión por campos
    Crear_user = tk.Button(Ventana, text="Crear usuario", bg="white", font=("Arial", 12), command=crear_usuario)
    Crear_user.pack(pady=10)

    iniciar_sesion = tk.Button(Ventana, text="Iniciar sesion", bg="white", font=("Arial", 12), command=ingresar_usuario)
    iniciar_sesion.pack(pady=5)

    label_usuario = tk.Label(Ventana, text="Usuario", bg="grey", fg="white")
    label_usuario.pack(pady=5)

    # Variables vinculadas a los Entry de nombre y mail
    nombre = tk.StringVar(Ventana)
    nombre_entry = tk.Entry(Ventana, textvariable=nombre)
    nombre_entry.pack(pady=5, ipadx=10, ipady=5)

    lable_mail = tk.Label(Ventana, text="Mail", bg="grey", fg="white")
    lable_mail.pack(pady=5)

    mail = tk.StringVar(Ventana)
    mail_entry = tk.Entry(Ventana, textvariable=mail)
    mail_entry.pack(pady=5, ipadx=10, ipady=5)

    resultado_label = tk.Label(Ventana, text="", bg="Black", fg="yellow")
    resultado_label.pack(pady=10)

    # Lista visual de usuarios para seleccionar e iniciar sesión (lista actualizada con refrescar_lista_visual)
    tk.Label(Ventana, text="Usuarios creados:", bg="Black", fg="white").pack(anchor="w", padx=12)
    listbox_usuarios = tk.Listbox(Ventana, width=48, height=10)
    listbox_usuarios.pack(padx=12, pady=(4,8))

    btn_abrir_seleccionado = tk.Button(Ventana, text="Iniciar sesión (seleccionado)", command=abrir_sesion_seleccionada)
    btn_abrir_seleccionado.pack(pady=4)

    # Mostrar usuarios almacenados al iniciar la app
    refrescar_lista_visual()
    Ventana.mainloop()

# Ejecuta main cuando se corre el script directamente
if _name_ == "_main_":
    main()
