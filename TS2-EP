import psycopg2
import psycopg2.extras
import datetime

conn = psycopg2.connect(host = "localhost", dbname= "postgres",  user =  "postgres" ,
                        password = "1234", port="5432")
cur =   conn.cursor()

cur.execute("""CREATE TABLE IF NOT EXISTS autos(
    id SERIAL PRIMARY KEY,
    tipo VARCHAR(255),        
    marca VARCHAR(255),
    placa VARCHAR(255) UNIQUE,        
    color  VARCHAR(255));""")

conn.commit()

cur.execute("""CREATE TABLE IF NOT EXISTS ingreso(
    id SERIAL PRIMARY KEY,
    id_auto INT REFERENCES autos(id),
    hora_ingreso TIMESTAMP,
    hora_salida TIMESTAMP
);""")
conn.commit()

cur.execute("""CREATE TABLE IF NOT EXISTS historial(
    id_auto INT REFERENCES autos(id),
    tipo VARCHAR(255),        
    marca VARCHAR(255),
    placa VARCHAR(255),        
    color  VARCHAR(255),
    hora_ingreso TIMESTAMP,
    hora_salida TIMESTAMP
);""")
conn.commit()

def ingreso_de_autos():
    tipo = input("Ingrese el tipo de vehiculo que ingreso: ") #Motocicletas,Autos,Camiones.
    marca = input("Ingrese marca del vehiculo: ")
    placa = input("Ingrese número de placa del vehiculo: ")
    color = input("Ingrese color del vehiculo: ")
    

    try:
        cur.execute("INSERT INTO autos (tipo, marca, placa, color) VALUES (%s, %s, %s, %s) RETURNING id", (tipo, marca, placa, color))
        id_auto = cur.fetchone()[0]
        conn.commit()

        print("Vehiculo ingresado con ID:", id_auto)
        hora_ingreso = datetime.datetime.now()
        cur.execute("INSERT INTO ingreso (id_auto, hora_ingreso) VALUES (%s, %s)", (id_auto, hora_ingreso))
        conn.commit()
        cur.execute("INSERT INTO historial (id_auto,tipo, marca, placa, color, hora_ingreso) VALUES (%s, %s, %s, %s, %s,%s)", (id_auto,tipo, marca, placa, color, hora_ingreso))
        conn.commit()

    except psycopg2.errors.UniqueViolation as e:
        print("Error: La placa ingresada ya está registrada.")
        conn.rollback() # Deshacer cualquier cambio parcial


def salida_de_autos():
    id_auto = int(input("Ingrese id del auto: "))
    hora_salida = datetime.datetime.now()
    cur.execute("UPDATE ingreso SET hora_salida = %s WHERE id_auto = %s", (hora_salida, id_auto))
    conn.commit()

    cur.execute("SELECT hora_ingreso, hora_salida FROM ingreso WHERE id_auto = %s", (id_auto,))
    timestamps = cur.fetchone()
    
    if timestamps is None:
        print("Error: El auto con ID proporcionado no está registrado.")
        return

    if timestamps[0] and timestamps[1]:
        tiempo = timestamps[1] - timestamps[0]
        horas = tiempo.total_seconds() / 3600
        
        # Aplicar cobro mínimo
        if horas < 1:
            cobro = 1000  # cobro mínimo que es el precio de una hora
        else:
            cobro = round(horas)*1000

        print("Hora de ingreso:", timestamps[0].strftime('%Y-%m-%d %H:%M:%S'))
        print("Hora de salida:", timestamps[1].strftime('%Y-%m-%d %H:%M:%S'))
        print("Tiempo estacionado:", tiempo)
        print("Monto a cobrar: ", cobro)
        print("Gracias por su visita, le esperamos pronto")
        print("")

    # En lugar de borrar el auto de la tabla autos, solo se borra el registro de la tabla ingreso.
    cur.execute("DELETE FROM ingreso WHERE id_auto = %s", (id_auto,))
    cur.execute("UPDATE historial SET hora_salida = %s WHERE id_auto = %s", (hora_salida, id_auto))
    conn.commit()

def mostrar_vehiculos_ingresados():
    cur.execute("SELECT id, tipo, marca, placa, color FROM autos")
    vehiculos = cur.fetchall()
    if not vehiculos:
        print("No hay vehículos ingresados.")
    else:
        print("ID\tTipo\tMarca\tPlaca\tColor")
        for vehiculo in vehiculos:
            print(f"{vehiculo[0]}\t{vehiculo[1]}\t{vehiculo[2]}\t{vehiculo[3]}\t{vehiculo[4]}")

def borrar_historial():
    confirmacion = input("¿Está seguro que desea borrar todo el historial? (sí/no): ").strip().lower()
    if confirmacion == 'sí':
        cur.execute("DELETE FROM historial")
        conn.commit()
        print("Historial borrado con éxito.")
    else:
        print("Operación cancelada.")

def informes():
    def autos_estacionados():
        cur.execute("SELECT a.id, a.tipo, a.marca, a.placa, a.color, i.hora_ingreso FROM autos a INNER JOIN ingreso i ON a.id = i.id_auto")
        vehiculos = cur.fetchall()
        for vehiculo in vehiculos:
            id_auto, tipo, marca, placa, color, hora_ingreso = vehiculo
            print("ID:", id_auto, "\tTipo:", tipo, "\tMarca:", marca, "\tPlaca:", placa, "\tColor:", color, "\tHora de ingreso:", hora_ingreso.strftime('%Y-%m-%d %H:%M:%S'))

    def autos_que_salieron():
        cur.execute("SELECT h.id_auto, h.tipo, h.marca, h.placa, h.color, h.hora_ingreso, h.hora_salida FROM historial h WHERE h.hora_salida IS NOT NULL")
        vehiculos = cur.fetchall()
        for vehiculo in vehiculos:
            id_auto, tipo, marca, placa, color, hora_ingreso, hora_salida = vehiculo
            print("ID:", id_auto, "\tTipo:", tipo, "\tMarca:", marca, "\tPlaca:", placa, "\tColor:", color, "\tHora de ingreso:", hora_ingreso.strftime('%Y-%m-%d %H:%M:%S'), "\tHora de salida:", hora_salida.strftime('%Y-%m-%d %H:%M:%S'))

      
    while True:
        print("1. Vehiculos estacionados.")
        print("2. Vehiculos que salieron.")
        print("3. Salir.")
        opcion = int(input("Ingrese una opcion: "))
        if opcion == 1:
            autos_estacionados()
        elif opcion == 2:
            autos_que_salieron()
        elif opcion == 3:
            break
        else:
            print("Opcion incorrecta")
              
    conn.commit()

def generar_reporte_ganancias():
    # conteo de registros que tienen entrada y salida registrada
    cur.execute("SELECT COUNT(*) FROM historial WHERE hora_ingreso IS NOT NULL AND hora_salida IS NOT NULL")
    total_registros = cur.fetchone()[0]
    
    ganancias = total_registros * 5.00  # Calcula las ganancias multiplicando por $5.00
    print(f"Ganancias totales hasta la fecha: ${ganancias:.2f}\n")

def generar_reporte_vehiculos():
    # Selecciona la marca de cada vehículo y cuenta cuántas veces aparece cada una
    cur.execute("SELECT marca, COUNT(marca) FROM autos GROUP BY marca")
    marcas = cur.fetchall()
    
    print("Conteo de Vehículos por Marca:")
    for marca, conteo in marcas:
        print(f"{marca}: {conteo}")

def reportes():
    while True:
        print("----- Reportes -----")
        print("1. Generar Reporte Ganancias.")
        print("2. Generar Reporte de Vehículos.")
        print("3. Salir.")
        opcion = int(input("Seleccione una opción: "))
        
        if opcion == 1:
            generar_reporte_ganancias()
        elif opcion == 2:
            generar_reporte_vehiculos()
        elif opcion == 3:
            break
        else:
            print("Opción incorrecta")
    
def menu():
    print("-----Bienvenido al sistema de ingreso y salida de Vehiculos-----")
    print("****************************************************************")
    while True:
        print("1. Ingreso de Vehiculo.")
        print("2. Salida de Vehiculo.")
        print("3. Mostrar Vehiculos Ingresados.") 
        print("4. Informes.")
        print("5. Borrar historial.")
        print("6. Reportes.")
        print("7. Salir.")
        
        opcion = int(input("Ingrese una opción: "))
        
        if opcion == 1:
            ingreso_de_autos()
        elif opcion == 2:
            salida_de_autos()
        elif opcion == 3:
            mostrar_vehiculos_ingresados()
        elif opcion == 4:
            informes()
        elif opcion == 5:
            borrar_historial()
        elif opcion == 6:
            reportes()
        elif opcion == 7:
            break
        else:
            print("Opción incorrecta")

menu()

cur.close()
conn.close()
