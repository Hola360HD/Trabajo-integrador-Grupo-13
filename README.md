# Trabajo-integrador-Grupo-13


import tkinter as tk

#creamos la clase persona que va a ser principalmente, un nodo
class persona:
    def __init__(self, nombre, mail):
        self.nombre = nombre
        self.mail = mail
        self.bandeja_entrada = BandejaEntrada()  # Ahora es un objeto, no una lista

#crearemos otra clase que va a ser otro nodo
class mensaje:
    def __init__(self, remitente, destinatario, asunto, cuerpo):
        self.remitente = remitente
        self.destinatario = destinatario
        self.asunto = asunto
        self.cuerpo = cuerpo

class BandejaEntrada:
    def __init__(self):
        self.mensajes = []

    def agregar_mensaje(self, mensaje):
        self.mensajes.append(mensaje)

    def ver_todos(self):
        return self.mensajes

    def __len__(self):
        return len(self.mensajes)

    def __getitem__(self, idx):
        return self.mensajes[idx]

#y la tercera clase para poder enviar mensajes
class Enviar_mensaje:
    def __init__(self, remitente_mail):
        self.remitente_mail = remitente_mail

    def enviar(self, destinatario_mail, asunto, cuerpo):
        if destinatario_mail not in usuarios:
            return "El destinatario no existe."
        remitente = usuarios[self.remitente_mail]
        destinatario = usuarios[destinatario_mail]
        msg = mensaje(remitente.mail, destinatario.mail, asunto, cuerpo)
        destinatario.bandeja_entrada.agregar_mensaje(msg)
        return f"Mensaje enviado a {destinatario.mail}"


#aqui guardaremos los usuarios creados.
usuarios = {}

#aqui crearemos los usuarios y verificaremos que si ya existen o no.
def crear_usuario():
    Nombre = nombre.get()
    Mail = mail.get()
    if Nombre == "" or Mail == "":
        resultado_label.config(text="Nombre y Mail no pueden estar vacíos")
    elif Mail in usuarios:
        resultado_label.config(text="Ese mail ya está registrado.")
    elif Nombre in usuarios:
        resultado_label.config(text="Ese Nombre ya está registrado.")
    else:
        user = persona(Nombre, Mail)
        usuarios[Mail] = user
        resultado_label.config(text=f"Usuario creado: {user.nombre}, {user.mail}")

#en esta parte ingresaremos el usuario ya existente
def ingresar_usuario():
    Nombre = nombre.get()
    Mail = mail.get()
    if Mail in usuarios and usuarios[Mail].nombre == Nombre:
        resultado_label.config(text=f"Bienvenido {Nombre}")
        Ventana.destroy()
        abrir_ventana_usuario(usuarios[Mail])
    else:
        resultado_label.config(text="Usuario o mail incorrecto, o no registrado.")

#al ingresar el usuario, cerraremos la ventana actual y abriremos la nueva ventana
def abrir_ventana_usuario(usuario_actual):
    #se ejecurara el volver_al_login solamente cuando el usuario ya alla iniciado sesion
    def volver_a_login():
        #cerramos la ventana del usuario
        ventana_user.destroy()
        #Volvemos a mostrar la ventana principal
        main()

    ventana_user = tk.Tk()
    ventana_user.title(f"Correo - {usuario_actual.nombre}")
    ventana_user.geometry("400x700+800+250")
    ventana_user.config(bg="Black")

    tk.Label(ventana_user, text=f"Bienvenido {usuario_actual.nombre}", bg="black", fg="white", font=("Arial", 14)).pack(pady=10)

 # Mostrar mensajes recibidos en un área con scroll
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

    bandeja = usuario_actual.bandeja_entrada
    if len(bandeja) > 0:
        for msg in bandeja.ver_todos():
            mensajes_text_widget.insert(
                "end",
                f"De: {msg.remitente}\nAsunto: {msg.asunto}\n{msg.cuerpo}\n---\n"
            )
    else:
        mensajes_text_widget.insert("end", "No tienes mensajes nuevos.")

    mensajes_text_widget.config(state="disabled")  # Solo lectura

    # Campos para enviar mensaje
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

    def enviar_mensaje_usuario():
        destinatario_mail = destinatario_entry.get()
        asunto = asunto_entry.get()
        cuerpo = cuerpo_entry.get()
        em = Enviar_mensaje(usuario_actual.mail)
        resultado = em.enviar(destinatario_mail, asunto, cuerpo)
        resultado_envio.config(text=resultado)

    enviar_btn = tk.Button(ventana_user, text="Enviar Mensaje", bg="lightblue", font=("Arial", 12), command=enviar_mensaje_usuario)
    enviar_btn.pack(pady=10)

    # Botón para cerrar sesión y volver a la ventana principal
    cerrar_sesion_btn = tk.Button(ventana_user, text="Cerrar sesión", bg="red", fg="white", font=("Arial", 12), command=volver_a_login)
    cerrar_sesion_btn.pack(pady=10)

    ventana_user.mainloop()

#iniciamos la ventana principal para poder iniciar secion o para crear un usuario
def main():
    global Ventana, nombre, mail, resultado_label
    Ventana = tk.Tk()
    Ventana.title("Correo")
    Ventana.geometry("400x500+800+250")
    Ventana.config(bg="Black")

    Crear_user = tk.Button(Ventana, text="Crear usuario", bg="white", font=("Arial", 12), command=crear_usuario)
    Crear_user.pack(pady=10)

    iniciar_sesion = tk.Button(Ventana, text="Iniciar sesion", bg="white", font=("Arial", 12), command=ingresar_usuario)
    iniciar_sesion.pack(pady=10)

    label_usuario = tk.Label(Ventana, text="Usuario", bg="grey", fg="white")
    label_usuario.pack(pady=5)

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

    Ventana.mainloop()

# Llama a main() para iniciar la ventana principal
main()
        else:
            print("No se inició sesión correctamente.")

iniciar_sesion()
