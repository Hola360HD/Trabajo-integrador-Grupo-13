# Trabajo-integrador-Grupo-13


class Persona:
    def _init_(self, Nombre, Mail):
        self.Nombre = Nombre
        self.Mail = Mail

    def crear_usuario(self):
        print("Ingrese su nombre: ", end="")
        nombre_nuevo = input()
        print("Ingrese su mail: ", end="")
        mail_nuevo = input()
        print("")
        return Persona(nombre_nuevo, mail_nuevo)
    

class Mensaje:
    def _init_(self, persona, mensajes_guardados):
        self.persona = persona
        self.mensajes_guardados = mensajes_guardados

    def mandar_mensaje(self, Amigo):
        print(f"Enviando mensaje a {Amigo.Nombre} ({Amigo.Mail})")

        print("escriba su mensaje: ", end="")
        mensaje = input()
        # Guardar el mensaje en el historial del amigo
        if Amigo.Mail not in self.mensajes_guardados:
            self.mensajes_guardados[Amigo.Mail] = []
        self.mensajes_guardados[Amigo.Mail].append((self.persona.Nombre, mensaje))
        print("Mensaje enviado y guardado.")
        print("")
        return mensaje
        
    def recibir_mensajes(self):
        # Mostrar todos los mensajes recibidos por el usuario actual
        mail = self.persona.Mail
        if mail in self.mensajes_guardados and self.mensajes_guardados[mail]:
            print(f"Mensajes para {self.persona.Nombre}:")
            for remitente, mensaje in self.mensajes_guardados[mail]:
                print(f"De {remitente}: {mensaje}")
            self.mensajes_guardados[mail] = []
        else:
            print("No hay mensajes para este usuario.")

def iniciar_sesion():
    mensajes_guardados = {}  # Diccionario para guardar los mensajes por mail
    usuarios = {}  # Diccionario para guardar usuarios por mail
    usuario = None

    while True:
        print("eliga una opcion: ")
        print("1- ingresar usuario")
        print("2- Crear usuario")
        print("3- cambiar usuario")
        print("4- salir")

        print("")

        opcion = int(input())

        print("")

        if opcion == 1:
            print("Ingrese su nombre: ", end="")
            nombre = input()
            print("Ingrese su mail: ", end="")
            mail = input()
            if mail in usuarios and usuarios[mail].Nombre == nombre:
                usuario = usuarios[mail]
                print(f"Bienvenido {usuario.Nombre}")
            elif nombre == "juan" and mail == "aaa":
                usuario = Persona(nombre, mail)
                usuarios[mail] = usuario
                print(f"Bienvenido {usuario.Nombre}")
            else:
                print("Usuario no encontrado.")
                usuario = None

        elif opcion == 2:
            usuario = Persona("", "")
            usuario = usuario.crear_usuario()
            usuarios[usuario.Mail] = usuario
            print(f"Bienvenido {usuario.Nombre}")

        elif opcion == 3:
            print("Ingrese su nombre: ", end="")
            nombre = input()
            print("Ingrese su mail: ", end="")
            mail = input()
            if mail in usuarios and usuarios[mail].Nombre == nombre:
                usuario = usuarios[mail]
                print(f"Cambio a usuario {usuario.Nombre}")
            else:
                print("Usuario no encontrado.")
                usuario = None

        elif opcion == 4:
            print("saliendo...")
            break

        else:
            print("opcion incorrecta")
            continue

        if usuario is not None:
            while True:
                print("")
                print("1- mandar mensaje")
                print("2- ver mensajes recibidos")
                print("3- cambiar de usuario")
                print("4- salir")
                print("")
                subopcion = int(input())
                print("")
                mensaje_obj = Mensaje(usuario, mensajes_guardados)

                if subopcion == 1:
                    print("¿Desea crear un usuario amigo para enviarle mensajes? (s/n): ", end="")
                    crear_amigo = input().lower()
                    if crear_amigo == "s":
                        amigo = Persona("", "")
                        amigo = amigo.crear_usuario()
                        usuarios[amigo.Mail] = amigo
                    else:
                        print("Ingrese el mail del amigo: ", end="")
                        mail_amigo = input()
                        if mail_amigo in usuarios:
                            amigo = usuarios[mail_amigo]
                        else:
                            print("Usuario amigo no encontrado, se usará usuario por defecto.")
                            amigo = Persona("pepe", "bbb")
                            usuarios[amigo.Mail] = amigo
                    mensaje_obj.mandar_mensaje(amigo)

                elif subopcion == 2:
                    mensaje_obj.recibir_mensajes()

                elif subopcion == 3:
                    print("Ingrese su nombre: ", end="")
                    nombre = input()
                    print("Ingrese su mail: ", end="")
                    mail = input()
                    if mail in usuarios and usuarios[mail].Nombre == nombre:
                        usuario = usuarios[mail]
                        print(f"Cambio a usuario {usuario.Nombre}")
                    else:
                        print("Usuario no encontrado.")
                        usuario = None
                        break

                elif subopcion == 4:
                    print("saliendo...")
                    return

                else:
                    print("opcion incorrecta")
        else:
            print("No se inició sesión correctamente.")

iniciar_sesion()
