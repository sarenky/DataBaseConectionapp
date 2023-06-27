# DataBaseConectionapp
First script for a secure database conection and interactions.

# Al inincio del programa se pregunta por el tipo de conexión con la BBDD. La opción predeterminada es con
# la configuración que se dispone en el PDF del reto, y la segunda opción permite introducir los datos manualmente para
# para una conexión personalizada. ( Pedirá HOST, USER, PASSWD y DB.(Teletubbies)).


import mysql.connector
from mysql.connector import errorcode
import time
import random

# ----------------------- DEFINICIÓN DE FUNCIONES EMPLEADAS EN LA EJECUCION DEL BLOQUE PRINCIPAL -----------------

# FUNCION PARA ENCRIPTAR - DESENCRIPTAR
# Cada caracter de la cadena de entrada, lo cambia por su simétrico (respecto a la posición) en las cadenas de
# sustitución: 'cadena_sust' y 'numero_sust'. Si el caracter no se encuentra en ninguna de las dos, lo deja intacto.
# <cadena> es el mensaje que se desea tratar. Debe ir entrecomillado.
# <opcion> define la acción a realizar. Será [0] para encriptar y [1] para desencriptar.

def encriptar(cadena):
    cadena_sust = 'aáAÁbBcCdDeéEÉfFgGhHiíIÍjJkKlLmMnNñÑoóOÓpPqQrRsStTuüúUÜÚvVwWxXyYzZ'
    numero_sust = '0123456789'
    cadena_salida = ''
    for caracter in cadena:  # recorre los caracteres de cadena
        if (caracter not in cadena_sust) and (caracter not in numero_sust):
            caracter = caracter
        elif caracter in cadena_sust:
            pos = cadena_sust.find(caracter)
            caracter = cadena_sust[len(cadena_sust)-pos - 1]
        elif caracter in numero_sust:
            pos = numero_sust.find(caracter)
            caracter = numero_sust[len(numero_sust) - pos - 1]
        cadena_salida += caracter  # construye la cadena de salida con los caracteres sustituidos
    return cadena_salida  # devuelve la cadena tratada

# FUNCIÓN PARA GENERAR CONTRASEÑAS
# <long> es la longitud de la contraseña deseada. Si no se incluye, se toma 9 por defecto.
# <caracteres> es la cadena con carateres que han de emplearse en la generación de la contraseña.
#  Si no se incluye, se usa una cadena por defecto.

def new_password2(long=9, caracteres='abcdefghijklmnopqrstuvwxyz0123456789.-@#!*%'):
    password = ''
    for i in range(0, long-1):
        caracter = random.choice(caracteres)
        tipo = random.choice('lu')
        if tipo == 'l':
            password += caracter.lower()
        elif tipo == 'u':
            password += caracter.upper()
    return password

# FUNCIÓN DE COMPROBAR CONTRASEÑA
# Esta función comprueba las características de la contraseña (cadena de entrada) y devuelve una variable booleana que
# será True si la cadena cumple los requisitos pertinentes y false si no.
#<cadena> es la cadena de caracteres cuyas características se comprueban
def comprueba_nueva_pass(cadena):
    contrasenia_buena = False
    numeros = 0
    mayusculas = 0
    caracteres = 0
    for x in cadena:
        if x in '0123456789':
            numeros += 1
        elif x in '!@*%':
            caracteres += 1
        elif x == x.upper():
            mayusculas += 1
    if numeros >= 2 and caracteres >= 1 and mayusculas >= 2:
        contrasenia_buena = True
    return contrasenia_buena  # devuelve booleano que es True si la contraseña es buena. False si no.


# FUNCIÓN DE NUEVO USUARIO
# Esta función, una vez llamada, inicia un proceso para añadir a un nuevo usuario a la base de datos.
def alta_usuario():
    # Pide nombre de usuario y comprueba que no existe. Si existe, vuelve a pedir
    nombre_usuario = input('Introduzca el nombre del usuario (PERO TU QUIEN ERES PEAZO PUTA(Veneno pa tu piel)): ')
    while True:
        # Comprueba que el usuario no exista
        consulta_existencia = ('SELECT COUNT(usuario) coincidencia FROM acceso WHERE usuario = %s')
        cur.execute(consulta_existencia, (nombre_usuario,))
        existencia = True
        for coincidencia in cur:
            if coincidencia[0] == 0:
                existencia = False
        # Si no existe, sale del bucle y continua
        if existencia is False:
            break
        # Si la condición anterior no salta, el nombre existe. Se vuelve a preguntar
        print('¿Pero qué haces? Tu ya estás registrado chavalín...')
        nombre_usuario = input('Invéntate otro nombre, anda: ')
    # Pide pregunta
    pregunta_seguridad = input('Introduzca la pregunta de seguridad: ')
    # Pide respuesta
    respuesta_seguridad = input('Introduzca la respuesta de la pregunta: ')
    # Pide contraseña y comprueba que es adecuada. Si no lo es, vuelve a preguntar
    clave_acceso = input('Introduzca la clave de acceso del usuario: ')
    while True:
        if comprueba_nueva_pass(clave_acceso) is False:
            print('La contraseña debe tener las siguientes carácterísticas: ')
            print(' - Longitud de, al menos, 9 caracteres')
            print(' - Como mínimo dos números')
            print(' - Como mínimo 2 mayusculas')
            print(' - Al menos un caracter especial (!@*%)')
            clave_acceso = input('Introduzca una clave de acceso con las propiedades descriptas: ')
            continue
        # Si la condición anterior no salta, la contraseña es buena
        break
    # Se añaden los nuevos datos a la tabla correspondiente
    estado_bloqueo = 0  # por defecto, desbloqueado
    insertar = ('INSERT INTO acceso (usuario, acceso_pw, bloqueo, pregunta, respuesta) '
                'VALUES (' + '%s, %s, %s, %s, %s)')
    cur.execute(insertar, (nombre_usuario, clave_acceso, estado_bloqueo, pregunta_seguridad, respuesta_seguridad,))
    miConexion.commit()
    print('\x1b[1;34m' + 'Usuario', nombre_usuario, 'creado con éxito.' + '\033[;m')
    return

# FUNCION PARA DESBLOQUEAR USUARIO                                                         # CAMBIAR POR LA NUEVA
# Función que actualiza la variable 'bloquo' en la bbdd del usuario de entrada
# <usuario> es el usuario que se quiere desbloquear
def desbloqueo_usuario(usuario):
    print('Para desbloquear este usuario debe responder correctamente a la siguiente pregunta,\n'
          'mira que como no lo aciertes, te banneo pa siempre!: ')
    consulta_pregunta = ('SELECT pregunta FROM acceso WHERE usuario = %s')
    cur.execute(consulta_pregunta, (usuario,))
    for pregunta in cur:
        print('\x1b[1;34m' + pregunta[0] + '\033[;m')
    respuesta_entrada = input('Respuesta: ')
    consulta_respuesta = ('SELECT respuesta FROM acceso WHERE usuario = %s')
    cur.execute(consulta_respuesta, (usuario,))
    for respuesta in cur:
        respuesta_correcta = respuesta[0]
    while respuesta_entrada != respuesta_correcta:
        respuesta_entrada = input('ESA NO ES, LISTO. Invéntate otra, anda: ')
    # Si sale del bucle anterior, respuesta correcta. Se actualiza el estado de bloqueo
    actualizacion = ('UPDATE acceso SET bloqueo = 0 WHERE bloqueo = 1 AND usuario = %s')
    cur.execute(actualizacion, (usuario,))
    miConexion.commit()
    print('\x1b[1;32m' + 'Usuario', usuario, 'desbloqueado.' + '\033[;m')
    return


# FUNCION DE ACCESO A LA APLICACIÓN
# Esta función no inicia un proceso de acceso a la aplicación, sino que comprueba si los datos de entrada
# pedidos en el proceso de acceso son correctos o no. Devuelve una lista con 3 variables booleanas correspondientes
# a cada uno de los tres parámetros relevantes en el acceso a la aplicación : nombre de usuario, password y bloqueo
# <usuario> es el nombre del usuario que se quiere comprobar
# <password> es la contraseña que se quiere comporbar
def comprueba_usuario(usuario='', password=''):
    # Variables de salida por defecto
    usuario_correcto = False
    contraseña_correcta = False
    bloqueado = True
    # Comprueba si usuario tiene coincidencia con los datos de la tabla
    consulta_usuario = ('SELECT COUNT(usuario) coincidencia FROM acceso WHERE usuario = %s')
    cur.execute(consulta_usuario, (usuario,))
    for coincidencia in cur:
        if coincidencia[0] == 1:  # Si hay coincidencia
            usuario_correcto = True  # Actualiza la variable de salida
            # Comprueba si el usuario está bloqueado
            # Comprueba si el usuario está bloqueado y si la contraseña es correcta
            consulta_acceso = ('SELECT acceso_pw, bloqueo FROM acceso WHERE usuario = %s')
            cur.execute(consulta_acceso, (usuario,))
            for acceso_pw, bloqueo in cur:
                # Si password correcta, actualiza variable de salida
                if acceso_pw == password:
                    contraseña_correcta = True
                # Si no bloqueado, actualiza variable de salida
                if bloqueo == 0:
                    bloqueado = False
    return [usuario_correcto, contraseña_correcta, bloqueado]  # devuelve lista con los 3 booleanos


# FUNCIÓN QUE SOLICITA UN NOMBRE DE USUARIO Y UNA CONTRASEÑA Y VERIFICA QUE SEAN CORRECTOS.
# DEVUELVE EL ACCESO AL PROGRAMA O EL BLOQUEO DEL USUARIO SI COLOCA MAL LA CONTRASEÑA 3 VECES.
def acceso_aplicacion():
    while True:
        atras = False  # variable auxiliar para volver al menu durante un proceso

        print('\n-------------- MENU DE ACCESO -----------------', end=time.sleep(0.3))
        print('-     [1] -> Inicio de sesión                 -', end=time.sleep(0.3))
        print('-     [2] -> Desbloquear usuario              -', end=time.sleep(0.3))
        print('-     [3] -> Alta de nuevo usuario            -', end=time.sleep(0.3))
        print('-     [4] -> Salir de la aplicación           -', end=time.sleep(0.3))
        print('-----------------------------------------------\n', end=time.sleep(0.3))
        accion = input('A por qué vienes, figura?: ')

        # Iniciar sesión
        if accion == '1':
            # Pide usuario y contraseña
            while True:
                usuario = input('\nQUIÉN ES? (Nombre de usuario) ([0] para volver al menu): ')
                if usuario == '0':
                    atras = True
                    break
                # Si el usuario es incorrecto, vuelve al inicio del bucle
                #    print(acceso_aplicacion(usuario)[0])  # TEST
                if comprueba_usuario(usuario)[0] == False:
                    print('No te conozco, ¿pero tu quién eres peazo puta?.')
                    continue
                # Si el usuario no es incorrecto pero esta bloqueado, vuelve al inicio del bucle
                elif comprueba_usuario(usuario)[2] == True:
                    print('Estás banneao por tonto.')
                    while True:
                        desbloquear = input('¿Quieres que te desbannee? [Si] o [No]: ').lower()
                        if desbloquear == 'si':
                            print('JAJA, pues no, pringao.')
                            for i in range(10):
                                time.sleep(1)
                                print(':)')
                            print('Que nooooo, que es broma, ya te desbanneo.')
                            desbloqueo_usuario(usuario)
                            break
                        elif desbloquear == 'no':
                            break
                        else:
                            print('Aprende a hablar, hijo.')
                    continue
                # Si el usuario no es incorrecto y no está bloqueado, pasa a pedir la contraseña
                else:
                    intento = 0
                    while True:
                        password = input('Pon tu clave de acceso: ')
                        # Si la contraseña es correcta, finaliza el bucle
                        if comprueba_usuario(usuario, password)[1] == True:
                            print('\x1b[1;32m' + 'SOY YO.(Clave correcta).' + '\033[;m')
                            break
                        # Si la contraseña es incorrecta, suma intento y vuelve a preguntar
                        else:
                            print('\x1b[1;31m' + 'POS VA A SER QUE NO. (Clave incorrecta).' + '\033[;m')
                            intento += 1
                        # Si llega a 3 intentos, bloquea al usuario y finaliza el programa
                        if intento == 3:
                            print('\n')
                            print('\x1b[1;31m' + '-----------------------------------------------', end=time.sleep(0.5))
                            print('------------- TE BANNEO POR TONTO -------------', end=time.sleep(0.5))
                            print('-----------------------------------------------' + '\033[;m', end=time.sleep(0.5))
                            time.sleep(5)
                            print('QUE ERES MUUU TONTO.')
                            time.sleep(5)
                            print('PERO TONTO DEL TÓ, VAMO...')
                            # Actualiza estado bloqueo en base de datos
                            actualizacion = ('UPDATE acceso SET bloqueo = 1 WHERE bloqueo = 0 AND usuario = %s')
                            cur.execute(actualizacion, (usuario,))
                            miConexion.commit()
                            quit()
                        else:
                            print('Te quedan', 3 - intento, 'intentos antes de que te bannee por tonto.')
                            continue
                        break
                break
            if atras is True:
                continue
            break

        # Desbloquear usuario
        elif accion == '2':
            while True:
                usuario_entrada = input('¿A qué tonto quieres desbannear hoy?¿Cuál va a ser tu obra de caridad del día?\n'
                                        '([0] para volver al menú): ')
                if usuario_entrada == '0':
                    atras = True
                    break
                # Se comprueba si el usuario existe. Si no existe, se vuelve al inicio
                consulta_existencia = ('SELECT COUNT(usuario) coincidencia FROM acceso WHERE usuario = %s')
                cur.execute(consulta_existencia, (usuario_entrada,))
                no_existe = False
                for coincidencia in cur:
                    if coincidencia[0] == 0:
                        no_existe = True
                if no_existe:
                    print('A este tonto no le conozco, dime otro.')
                    continue
                # Se comprueba si esta bloqueado. Si no lo está, se vuelve al inicio
                consulta_estado = ('SELECT bloqueo FROM acceso WHERE usuario = %s')
                cur.execute(consulta_estado, (usuario_entrada,))
                bloqueado = True
                for bloqueo in cur:
                    if bloqueo[0] == 0:
                        bloqueado = False
                if not bloqueado:
                    print('Este angelito no está en mi lista negra.')
                    break
                # Si no saltaron las condiciones anteriores, existe y está bloqueado. Se desbloquea
                desbloqueo_usuario(usuario_entrada)
                break
            if atras is True:
                continue

        # Dar de alta a nuevo usuario
        elif accion == '3':
            alta_usuario()

        # Salir
        elif accion == '4':
            seguro = input('¿Seguro? Mira que si te vas no vuelves... [Si] o [No]: ').lower()
            while seguro != 'si' and seguro != 'no':
                seguro = input('¿Pero qué me estás contando?. Pon [Si] o [No], cazurro: ')
            time.sleep(1)
            if seguro == 'si':
                miConexion.close()
                quit()
            elif seguro == 'no':
                continue

        # Entrada no válida
        else:
            print('¿Pero tu ves algún', accion, 'en la lista que te he dao? Caralpargata este...')

    return usuario


# FUNCION DE ADMIN - BLOQUEAR USUARIO
def admin_bloqueo():
    # Diccionario de Usuarios y sus bloqueos
    usuarios = {}
    # Guardamos los valores en el diccionario
    cur.execute("""SELECT usuario,bloqueo FROM acceso;""")
    for usuario, bloqueo in cur.fetchall():
        usuarios[usuario] = bloqueo

    # Preguntamos el nombre del usuario
    user = input('Hombre, señor Administrador, a quién quiere bloquear hoy?:\n')

    # Variables con las querys de MySQL
    bloquear = """
                UPDATE acceso 
                    SET bloqueo = True 
                    WHERE usuario = %s
                    """
    desbloquear = """
                    UPDATE acceso 
                        SET bloqueo = False 
                        WHERE usuario = %s
                        """
    # Verificamos que el usuario esté en el llavero
    if user not in usuarios.keys():
        print(f'Exmo Administrador, no conozco al tonto este llamado {user} .\n')
        return
    else:
        pregunta = input(f'Querido y adorado Líder Supremo, quieres Bloquear o Desbloquear al carapapa de {user}? [B] o [D]:\n').upper()
        if pregunta == 'B':
            if usuarios[user] == 1:
                print(f'Este carapapa {user} ya está banneao por tonto.\n')
                return
            else:
                pregunta2 = input('Está seguro que desea bloquear al usuario?: [S/N]\n').upper()
                if pregunta2.startswith('S') or pregunta2 == 1:
                    cur.execute(bloquear, (user,))  # Modificamos la columna de bloqueo de la base de datos.
                    miConexion.commit()
                    print(f'El usuario {user} ha sido bloqueado.')
                    return
                else:
                    quit()

        elif pregunta == 'D':
            if usuarios[user] == 0:
                print(f'El usuario {user} ya se encuentra desbloqueado\n')
                return
            else:
                pregunta2 = input('Está seguro que desea desbloquear al usuario?: [S/N]\n').upper()
                if pregunta2.startswith('S') or pregunta2 == 1:
                    cur.execute(desbloquear, (user,))  # Modificamos la columna de bloqueo de la base de datos.
                    miConexion.commit()
                    print(f'El usuario {user} ha sido desbloqueado.')
                    return
                else:
                    return
        else:
            print('La opción no es correcta.')
            return

# FUNCION DE ADMIN - LISTAR CONTRASEÑAS POR USUARIO
def admin_view_user():
    # Solicitamos el nombre del usuario
    user = input('Ingrese el nombre del Usuario:\n')

    # Variables con las querys de MySQL.
    check_user = """
                SELECT usuario, bloqueo
                    FROM acceso 
            """
    select = """
                SELECT aplicacion, contraseña
                    FROM user_pass 
                    WHERE usuario = %s
                    ORDER BY aplicacion ASC
            """

    # Guardamos los usuarios en un diccionario para acceder a ellos en verificaciones posteriores
    cur.execute(check_user)
    usuarios = {}
    for usuario, bloqueo in cur.fetchall():
        usuarios[usuario] = bloqueo

    # Verificamos que el usuario esté en el diccionario
    if user not in usuarios.keys():
        print('El usuario no se encuentra en la base de datos.\n')  # Si no está, vuelve al menú
        return
    else:  # Si está, imprime sus credenciales guardadas.
        cur.execute(select, (user,))
        print(f'Las aplicaciones y contraseñas del usuario {user} son:\n')
        for aplicacion, contraseña in cur.fetchall():
            pass_unlocked = encriptar(contraseña)
            print(f'--> \x1b[1;42m\x1b[1;30m{aplicacion}:\033[;m ---- \x1b[1;42m\x1b[1;30m'
                  f'{pass_unlocked}\033[;m', end=time.sleep(0.3))
        return

# FUNCION DE ADMIN - LISTAR ALL
def admin_view_all():
    # Variable con las querys de MySQL.
    order_by_select = """
                SELECT usuario, aplicacion, contraseña 
                    FROM user_pass 
                    GROUP BY usuario, aplicacion, contraseña 
                    ORDER BY usuario, aplicacion ASC
                    """

    cur.execute(order_by_select)  # Ordenamos las contraseñas por usuario y las imprimimos
    print(f'Las contraseñas guardadas en el llavero son:\n', end=time.sleep(0.3))
    for usuario, aplicacion, contraseña in cur.fetchall():
        pass_unlocked = encriptar(contraseña)  # Acá hay que desencriptar las contraseñas, cambiar el nombre de la fórmula
        print(f'{usuario} --> \x1b[1;42m\x1b[1;30m{aplicacion}:\033[;m ---- \x1b[1;42m\x1b[1;30m'
              f'{pass_unlocked}\033[;m', end=time.sleep(0.3))
    return

# FUNCION DE ADMIN - BORRAR USUARIO
def admin_delete():
    cur.execute("SELECT usuario, bloqueo FROM acceso")
    lista_usuarios = []
    for row, index in cur:
        lista_usuarios.append(row)
    print("Mr Admin, el listado de usuarios es el siguiente: ", lista_usuarios)

    # Pregunta por usuario que se desea eliminar y eliminación si existe
    eliminacion = str(input("¿Qué usuario de dicho listado desea borrar?: "))
    if eliminacion in lista_usuarios:
        cur.execute("DELETE FROM acceso WHERE usuario = %s", (eliminacion,))
        cur.execute("DELETE FROM user_pass WHERE usuario = %s", (eliminacion,))
        miConexion.commit()
        print(f'El usuario {eliminacion} y sus aplicaciones han sido eliminadas con éxito')
        return
    else:
        print("El usuario introducido no se encuentra en la base de datos")
        return

# FUNCION MENU ADMIN
def menu_admin():
    while True:
        # IMPRESIÓN DEL MENÚ
        print('\n\n|---------------' + '\x1b[1;32m' + 'MENÚ ADMINISTRADOR' + '\033[;m' + '--------------|', end=time.sleep(0.2))
        print('|    1- Bloquear o desbloquear un usuario.      |', end=time.sleep(0.2))
        print('|    2- Imprimir todas las contraseñas.         |', end=time.sleep(0.2))
        print('|    3- Imprimir contraseñas de un usuario.     |', end=time.sleep(0.2))
        print('|    4- Eliminar un usuario.                    |', end=time.sleep(0.2))
        print('|    ' + '\x1b[1;31m' + '0- SALIR' + '\033[;m' + '                                   |', end=time.sleep(0.2))
        print('|-----------------------------------------------|', end=time.sleep(0.2))

        # INPUT CON VERIFICACIÓN DE ERRORES PARA ELEGIR LA OPCIÓN DEL MENÚ QUE QUEREMOS REALIZAR
        try:
            time.sleep(0.3)
            option = int(input('\nQué operación desea realizar?:\n'))
        except ValueError:
            time.sleep(0.3)
            print('Por favor, escriba el número de la acción que desea realizar.\n')
        else:  # SI LA OPCIÓN ES CORRECTA, ENTRAMOS EN LOS CONDICIONALES DE CADA OPCIÓN DEL MENÚ

            if option == 1:
                admin_bloqueo()

            elif option == 2:
                admin_view_all()
                input('Presione [Enter] para continuar.')

            elif option == 3:
                admin_view_user()
                input('Presione [Enter] para continuar.')

            elif option == 4:
                admin_delete()
                input('Presione [Enter] para continuar.')

            elif option == 0:
                miConexion.close()
                quit()

            else:
                time.sleep(1)
                print('Lo sentimos, la opción elegida no está en el menú, vuelva a intentarlo.\n')

# ------------------------------------------- CONEXIÓN CON LA BASE DE DATOS ------------------------------------------

# Para conectarse a la bbd, se presentan dos opciones:
# La primera, que toma como configuración los datos del enunciado del pdf.
print('---------------------------- CONEXIÓN CON DATABASE ----------------------------\n')
connect_predeterminada = input('Si desea conectarse a la base de datos configurada por defecto, introduzca [S].\n'
                               'En caso contrario, introduzca [N] y se le pedirán los datos necesarios: ').lower()
while connect_predeterminada not in 'sn' and len(connect_predeterminada) != 1:
    connect_predeterminada = input('Entrada incorrecta.\n'
                                   'Introduzca [N] o [S]:')

# Conexion por defecto.

if connect_predeterminada == 's':
    config = {
        'host': 'localhost',
        'user': 'root',
        'passwd': '12345678',                   ################### MODIFICAR PASSWORD SI SE REQUIERE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
        'db': 'llavero',
        'auth_plugin': 'mysql_native_password'
    }  # configuración de la conexión predeterminada. es la que figura en el pdf del curso
    try:
        miConexion = mysql.connector.connect(**config)
    # Si no logra conectarse, indica el motivo
    except mysql.connector.Error as err:
        if err.errno == errorcode.ER_ACCESS_DENIED_ERROR:
            print('\x1b[1;31m' + 'ERROR: ' + '\033[;m' + 'El nombre de usuario o la contraseña son incorrectos.')
            quit()
        elif err.errno == errorcode.ER_BAD_DB_ERROR:
            print('\x1b[1;31m' + 'ERROR: ' + '\033[;m' + 'La base de datos no existe.')
            quit()
        else:
            print(err)
            quit()

# La segunda, se conecta a partir de los datos introducidos manualemente.

elif connect_predeterminada == 'n':
    # Se piden los datos necesarios para conectarse a la base de datos
    print('A continuación, introduzca los datos de la base de datos a la que se conectará la aplicación:\n')
    host = input(' * host: ')
    usuario_bbdd = input(' * user: ')
    password_bbdd = input(' * password: ')
    nombre_bbdd = input(' * db: ')
    # Se crea la conexión
    config = {
        'host': host,
        'user': usuario_bbdd,
        'passwd': password_bbdd,
        'db': nombre_bbdd,
        'auth_plugin': 'mysql_native_password'
    }  # configuración de acceso a la base de datos
    # Se conecta a la base de datos
    while True:
        # Se crea la conexión
        config = {
            'host': host,
            'user': usuario_bbdd,
            'passwd': password_bbdd,
            'db': nombre_bbdd,
            'auth_plugin': 'mysql_native_password'}
        try:
            miConexion = mysql.connector.connect(**config)
            break
        # Si no logra conectarse, indica el motivo y, si en algunos casos, vuelve a pedir los datos que
        # producian el error.
        except mysql.connector.Error as err:
            if err.errno == errorcode.ER_ACCESS_DENIED_ERROR:
                print('\x1b[1;31m' + 'ERROR: ' + '\033[;m' + 'El nombre de usuario o la contraseña son incorrectos. '
                                                             'Pruebe a introducirlos de nuevo.')
                usuario_bbdd = input(' user: ')
                password_bbdd = input(' * password: ')
            elif err.errno == errorcode.ER_BAD_DB_ERROR:
                print('\x1b[1;31m' + 'ERROR: ' + '\033[;m' + 'La base de datos no existe. Pruebe con otro nombre:')
                nombre_bbdd = input(' * db: ')
            else:
                print(err)
                quit()

# Si sale del bucle anterior, la conexión se ha realizado correctamente
print('')
print('\x1b[1;32m' + 'La conexión se ha efectuado correctamente.' + '\033[;m')

# Se activa la conexión
cur = miConexion.cursor()

# ---------------------------------------------------- MENU ACCESO ---------------------------------------------------

usuario = acceso_aplicacion()

# ---------------------------------------------------- MENU ADMIN ----------------------------------------------------

if usuario == 'administrator':
    print('Bienvenido Excelentisimo,')
    time.sleep(0.4)
    print('Grandioso,')
    time.sleep(0.4)
    print('Crack,')
    time.sleep(0.4)
    print('Figura,')
    time.sleep(0.4)
    print('Monstruo,')
    time.sleep(0.4)
    print('Grande,')
    time.sleep(0.4)
    print('Que te como la cara, coñooo...')
    time.sleep(0.4)
    print('Alabado sea el Gran Líder Supremo, ¡viva el señor Administrador!\n\n')
    time.sleep(0.6)
    menu_admin()

# ---------------------------------------------------- MENU INICIAL --------------------------------------------------

while True:
    # Variable auxiliar para volver al menú en caso de haberse equivocado.
    atras = False
    # Menu de la aplicación
    print('\n'+'\x1b[1;34m'+'----------------------- QUE VIENES A BUSCAR -----------------------',
          '\033[;m', end=time.sleep(0.3))
    print('\x1b[1;36m'+'-           [1] -> Guardar una nueva contraseña                   -', end=time.sleep(0.3))
    print('-           [2] -> Modificar una contraseña existente             -', end=time.sleep(0.3))
    print('-           [3] -> Eliminar una contraseña existente              -', end=time.sleep(0.3))
    print('-           [4] -> Buscar una contraseña almacenada               -', end=time.sleep(0.3))
    print('-           [5] -> Listar contraseñas existentes                  -', end=time.sleep(0.3))
    print('-           [6] -> Salir de la aplicación                         -'+'\033[;m', end=time.sleep(0.3))
    print('\x1b[1;34m'+'-------------------------------------------------------------------'+
          '\033[;m'+'\n', end=time.sleep(0.3))

    # Pedir acción y repetir pregunta si el formato no es correcto
    while True:
        accion = input('¿Que quieren hacer hoy esos ojitos lindos?: ')
        if accion in '123456' and len(accion) == 1:
            break
        else:
            print('Esto... hoy va a ser que no, cari. Prueba otra cosa.')
    print('\n')

    # Si la opción es 1
    if accion == '1':
        # Pide el nombre de la nueva contraseña. Si ya existe, vuelve a preguntar
        existe_contrasenia = True  # booleana para comprobar la existencia de la nueva contraseña en la bbdd
        while existe_contrasenia is True:
            nombre = input('¿Cómo se llama lo que quieres meterme? (Nombre de la contraseña, pervertido.)\n'
                           '([0] para volver al menú): ')
            if nombre == '0':
                atras = True
                break
            # Comprueba que el nombre no esta en la BBDD
            consulta = ('SELECT COUNT(aplicacion) coincidencias FROM user_pass WHERE aplicacion = %s AND usuario = %s')
            cur.execute(consulta, (nombre, usuario,))
            for coincidencias in cur:
                if coincidencias[0] == 0:
                    existe_contrasenia = False
                else:
                    print('Eso ya me lo has metido. (Ese nombre ya existe.')
        if atras is True:
            continue

        # Pide o genera nueva contraseña
        while True:
            generar = input('¿Puedo meter lo que yo quiera? (Una contraseña aleatoria, que mente tenemos hoy...\n '
                            '[Si] o [No]: ').lower()
            if generar == 'si':
                while True:
                    nueva_contrasenia = new_password2()
                    if comprueba_nueva_pass(nueva_contrasenia) is True:
                        break
                print('\nContraseña: ', '\x1b[1;34m' + nueva_contrasenia + '\033[;m')
                break
            elif generar == 'no':
                nueva_contrasenia = input('Introduce lo que tu quieras (una contraseña): ')
                if comprueba_nueva_pass(nueva_contrasenia) is False:
                    print('Vaya mierda la que has metido... No es una contraseña segura, \n '
                          'para serlo debería cumplir los siguientes requisitos:')
                    print(' - Longitud de, al menos, 9 caracteres')
                    print(' - Como mínimo dos números')
                    print(' - Como mínimo 2 mayusculas')
                    print(' - Al menos un caracter especial (!@*%)')
                break
            else:
                print('Que no te enteras, responde [Si] o [No].')

        # Encripta la nueva contraseña
        nueva_contrasenia_encriptada = encriptar(nueva_contrasenia)
        # Inserta la nueva contraseña en la base de datos
        insertar = ('INSERT INTO user_pass (aplicacion, contraseña, usuario) VALUES (' + '%s, %s, %s)')
        nuevos_datos = (nombre, nueva_contrasenia_encriptada, usuario,)
        cur.execute(insertar, nuevos_datos)
        miConexion.commit()
        time.sleep(0.5)
        print('\x1b[1;35m'+'Guardando', end='')
        time.sleep(1)
        print('.', end='')
        time.sleep(1)
        print('.', end='')
        time.sleep(1)
        print('.'+'\033[;m', end='')
        time.sleep(1)
        print('\nMe has metido', nombre, 'con exito. Muy bien campeón.\n')
        time.sleep(2)

    # Si la opción es 2
        # Modificar contraseña
    elif accion == '2':
        # Pide el nombre de la contraseña. Si no existe, vuelve a preguntar
        existe_contrasenia = False  # booleana para comprobar la existencia de la nueva contraseña en la bbdd
        while existe_contrasenia is False:
            nombre = input('Introduzca el nombre de la contraseña que quiere modificar \n([0] para volver al menú): ')
            if nombre == '0':
                atras = True
                break
            # Comprueba que el nombre no esta en la BBDD
            consulta = ('SELECT COUNT(aplicacion) coincidencias FROM user_pass WHERE aplicacion = %s AND usuario = %s')
            cur.execute(consulta, (nombre, usuario,))
            for coincidencias in cur:
                if coincidencias[0] != 0:
                    existe_contrasenia = True
                else:
                    print('Bobo, que deci bobo. Esta contraseña no existe.')
        if atras is True:
            continue

        # Pide o genera nueva contraseña
        while True:
            generar = input('Quieres generar una contraseña aleatoria? [Si] o [No]: ').lower()
            if generar == 'si':
                while True:
                    nueva_contrasenia = new_password2()
                    if comprueba_nueva_pass(nueva_contrasenia) is True:
                        break
                print('\nContraseña: ', '\x1b[1;34m' + nueva_contrasenia + '\033[;m')
                break
            elif generar == 'no':
                nueva_contrasenia = input('Introduce la nueva contraseña: ')
                if comprueba_nueva_pass(nueva_contrasenia) is False:
                    print('Vaya mierda la que has metido... No es una contraseña segura, \n '
                          'para serlo debería cumplir los siguientes requisitos:')
                    print(' - Longitud de, al menos, 9 caracteres')
                    print(' - Como mínimo dos números')
                    print(' - Como mínimo 2 mayusculas')
                    print(' - Al menos un caracter especial (!@*%)')
                break
            else:
                print('Que no te enteras, responde [Si] o [No].')
        # Encripta la nueva contraseña
        nueva_contrasenia_encriptada = encriptar(nueva_contrasenia)
        # Modifica el registro
        consulta_antigua_contrasenia = (
            'SELECT contraseña FROM user_pass WHERE aplicacion = %s AND usuario = %s')
        cur.execute(consulta_antigua_contrasenia, (nombre, usuario,))
        for contrasenia in cur:
            antigua_contrasenia = contrasenia[0]
        modificacion = ('UPDATE user_pass SET contraseña = %s WHERE contraseña = %s AND usuario = %s')
        cur.execute(modificacion, (nueva_contrasenia_encriptada, antigua_contrasenia, usuario,))
        miConexion.commit()
        time.sleep(0.5)
        print('\x1b[1;35m'+'Modificando', end='')
        time.sleep(1)
        print('.', end='')
        time.sleep(1)
        print('.', end='')
        time.sleep(1)
        print('.'+'\033[;m', end='')
        time.sleep(1)
        print('\x1b[1;33m'+'\nLa contraseña', nombre, 'se ha modificado con exito.\n'+'\033[;m')
        time.sleep(2)

    # Si la opción es 3

        # Eliminar contraseña
    elif accion == '3':
        time.sleep(1)
        # Pide el nombre de la contraseña. Si no existe, vuelve a preguntar
        existe_contrasenia = False  # booleana para comprobar la existencia de la nueva contraseña en la bbdd
        while existe_contrasenia is False:
            nombre = input('¿Que hueco quieres dejar libre? (Sácame lo que quieras)\n([0] para volver al menú): ')
            if nombre == '0':
                atras = True
                break
            # Comprueba que el nombre no esta en la BBDD
            consulta = ('SELECT COUNT(aplicacion) coincidencias FROM user_pass WHERE aplicacion = %s AND usuario = %s')
            cur.execute(consulta, (nombre, usuario,))
            for coincidencias in cur:
                if coincidencias[0] != 0:
                    existe_contrasenia = True
                else:
                    print('No me has metido nada con ese nombre. No hay ninguna contraseña con ese nombre.')
        if atras is True:
            continue

        # Preguntar si esta seguro de su acción.
        segura = input('¿Estás seguro de que quieres sacármela? [S/N]: ').lower()
        while segura != 'n' and segura != 's':
            print('Que no te enteras, responde [S] o [N].')
            segura = input('¿Estás seguro de que quieres sacármela? [S/N]: ').lower()
            time.sleep(1)
        if segura == 'n':
            atras = True
            break
        elif segura == 's':
            # Borra el registro
            borrado = ('DELETE FROM user_pass WHERE aplicacion = %s AND usuario = %s')
            cur.execute(borrado, (nombre, usuario,))
            miConexion.commit()
            time.sleep(0.5)
            print('\x1b[1;35m'+'Eliminando', end='')
            time.sleep(1)
            print('.', end='')
            time.sleep(1)
            print('.', end='')
            time.sleep(1)
            print('.'+'\033[;m', end='')
            time.sleep(1)
            print('\x1b[1;33m'+'\nLa contraseña', nombre, 'ha salido del chat (F).\n'+'\033[;m')
            time.sleep(2)
        if atras is True:
            continue
    # Si la opción es 4
    elif accion == '4':
        time.sleep(1)
        # Pide el nombre de la contraseña. Si no existe, vuelve a preguntar
        existe_contrasenia = False  # booleana para comprobar la existencia de la nueva contraseña en la bbdd
        while existe_contrasenia is False:
            nombre = input('¿Qué quieres que te enseñe? (Qué contraseña, malpensado...) \n([0] para volver al menú: ')
            if nombre == '0':
                atras = True
                break
            # Comprueba que el nombre no esta en la BBDD
            consulta = ('SELECT COUNT(aplicacion) coincidencias FROM user_pass WHERE aplicacion = %s AND usuario = %s')
            cur.execute(consulta, (nombre, usuario,))
            for coincidencias in cur:
                if coincidencias[0] != 0:
                    existe_contrasenia = True
                else:
                    print('Bobo, que deci bobo. Esta contraseña no existe.')
        if atras is True:
            continue

        # Hace la consulta con el nombre especificado
        consulta = ('SELECT contraseña FROM user_pass WHERE aplicacion = %s AND usuario = %s')
        cur.execute(consulta, (nombre, usuario,))
        time.sleep(0.5)
        print('\x1b[1;35m'+'Quédate quieto', end='')
        time.sleep(1)
        print('.', end='')
        time.sleep(1)
        print('.', end='')
        time.sleep(1)
        print('.', end='')
        time.sleep(1)
        print('\nYASTA'+'\033[;m', end='')
        time.sleep(0.3)
        for contrasenia in cur:
            print('\nLa contraseña', '\x1b[1;34m' + nombre +
                  '\033[;m', 'es: ', '\x1b[1;34m' + encriptar(contrasenia[0]) + '\033[;m')
        print('')
        # time.sleep(2)
        input('Pulse [Enter] para continuar.\n')

    # Si la opción es 5

    elif accion == '5':
        time.sleep(1)
        print('Que es lo que tengo, que tengo de to... tengo gambas, tengo chopitos, tengo croquetas y tengo jamón: ')
        # Consulta de la tabla
        cur.execute('SELECT aplicacion FROM user_pass WHERE usuario = %s', (usuario,))
        # Imprime la primera columna, que es la buscada
        for aplicacion in cur:
            print('*', aplicacion[0])
        print('')
        # time.sleep(2)
        input('Pulse [Enter] para continuar.\n')

    # Si la opción es 6
    elif accion == '6':
        seguro = input('¿Ah..., que ya te vas?[S/N]: ').lower()
        while seguro != 's' and 'n':
            print('Tio, venga ya, que no es tan difícil... Pon [S/N], coño: ')
            seguro = input('¿Estás seguro?, mira que si te vas no sabes volver tontolava...[S/N]: ').lower()
        time.sleep(1)
        if seguro == 's':
            print('Vale vale, ya vendrás arrastrándote...')
            miConexion.close()
            quit()
        elif seguro == 'n':
            break
