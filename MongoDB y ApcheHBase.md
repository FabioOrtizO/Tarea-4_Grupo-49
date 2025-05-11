🚀 Proyecto: Carga y Consulta del Parque Automotor en Apache HBase usando Happybase
📄 Descripción General
Este proyecto documenta el procedimiento para cargar un conjunto de datos del parque automotor de Colombia, proveniente de Datos.gov.co, hacia una tabla en Apache HBase.
Utiliza Python y la librería Happybase para conectarse a HBase, cargar datos masivos y realizar consultas básicas.

📥 1. Descarga del Dataset
Desde la máquina virtual donde está instalado HBase, descarga el archivo CSV y renómbralo para uso posterior:

bash

wget https://www.datos.gov.co/api/views/u3vn-bdcy/rows.csv?accessType=DOWNLOAD -O parque_automotor.csv --no-check-certificate
🔵 Nota: Asegúrate que el archivo se llame exactamente parque_automotor.csv para que el script funcione sin errores.

🏗️ 2. Creación de la Tabla en HBase
Nombre de la tabla: parque_automotor

Familias de columnas:

ubicacion:

NOMBRE_DEPARTAMENTO

NOMBRE_MUNICIPIO

vehiculo:

NOMBRE_SERVICIO

ESTADO_DEL_VEHICULO

NOMBRE_DE_LA_CLASE

registro:

FECHA DE REGISTRO

CANTIDAD

MES DE PUBLICACION

AÑO DE PUBLICACIÓN

Antes de proceder:

Inicia el servicio de HBase: start-hbase.sh

Inicia el servicio Thrift: hbase-daemon.sh start thrift

🖥️ 3. Creación del Script Python consultas.py
En la máquina virtual, crea el archivo:

bash
Copiar
Editar
nano consultas.py
Pega el siguiente contenido actualizado:

python
Copiar
Editar
import happybase
import pandas as pd

try:
    connection = happybase.Connection('localhost')
    print("Conexión establecida con HBase")

    table_name = 'parque_automotor'
    families = {
        'ubicacion': dict(),
        'vehiculo': dict(),
        'registro': dict()
    }

    if table_name.encode() in connection.tables():
        print(f"Eliminando tabla existente - {table_name}")
        connection.delete_table(table_name, disable=True)

    connection.create_table(table_name, families)
    table = connection.table(table_name)
    print("Tabla 'parque_automotor' creada exitosamente")

    data = pd.read_csv('parque_automotor.csv').head(100)

    for index, row in data.iterrows():
        row_key = f'vehiculo_{index}'.encode()
        record = {
            b'ubicacion:departamento': str(row['NOMBRE_DEPARTAMENTO']).encode(),
            b'ubicacion:municipio': str(row['NOMBRE_MUNICIPIO']).encode(),
            b'vehiculo:servicio': str(row['NOMBRE_SERVICIO']).encode(),
            b'vehiculo:estado': str(row['ESTADO_DEL_VEHICULO']).encode(),
            b'vehiculo:clase': str(row['NOMBRE_DE_LA_CLASE']).encode(),
            b'registro:fecha_registro': str(row['FECHA DE REGISTRO']).encode(),
            b'registro:cantidad': str(row['CANTIDAD']).encode(),
            b'registro:mes_publicacion': str(row['MES DE PUBLICACION']).encode(),
            b'registro:ano_publicacion': str(row['AÑO DE PUBLICACIÓN']).encode()
        }
        table.put(row_key, record)

    print("Datos cargados exitosamente en HBase")

    # Consultas básicas
    print("\n=== Primeros 3 registros ===")
    count = 0
    for key, data in table.scan():
        if count < 3:
            print(f"ID: {key.decode()}")
            print(f"Municipio: {data.get(b'ubicacion:municipio', b'').decode()}")
            print(f"Servicio: {data.get(b'vehiculo:servicio', b'').decode()}")
            print(f"Cantidad: {data.get(b'registro:cantidad', b'').decode()}")
            count += 1

    print("\n=== Vehículos registrados en Bogotá ===")
    for key, data in table.scan():
        if data.get(b'ubicacion:municipio', b'').decode() == 'BOGOTA':
            print(f"ID: {key.decode()} - Clase: {data.get(b'vehiculo:clase', b'').decode()}")

    print("\n=== Conteo de vehículos por tipo de servicio ===")
    servicio_stats = {}
    for key, data in table.scan():
        servicio = data.get(b'vehiculo:servicio', b'').decode()
        servicio_stats[servicio] = servicio_stats.get(servicio, 0) + 1

    for servicio, count in servicio_stats.items():
        print(f"{servicio}: {count} vehículos")

    print("\n=== Actualización de estado de un vehículo ===")
    vehiculo_actualizar = 'vehiculo_0'
    nuevo_estado = 'INACTIVO'
    table.put(vehiculo_actualizar.encode(), {b'vehiculo:estado': nuevo_estado.encode()})
    print(f"Estado actualizado para {vehiculo_actualizar}")

    print("\n=== Eliminación de un vehículo ===")
    vehiculo_eliminar = 'vehiculo_1'
    table.delete(vehiculo_eliminar.encode())
    print(f"Vehículo {vehiculo_eliminar} eliminado.")

except Exception as e:
    print(f"Error: {str(e)}")

finally:
    connection.close()
    print("Conexión cerrada.")
Guarda los cambios (Ctrl + O, Enter, Ctrl + X) y luego ejecuta:

bash
Copiar
Editar
python3 consultas.py
🔧 4. Instalación de Dependencias
Antes de ejecutar el script, asegúrate de instalar las librerías necesarias:

bash
Copiar
Editar
pip install happybase pandas
📊 5. Verificación de la Tabla en HBase
Puedes consultar el estado de la base de datos accediendo vía navegador:

bash
Copiar
Editar
http://<IP_MAQUINA_VIRTUAL>:16010
Ejemplo:
http://192.168.60.28:16010

Desde aquí podrás:

Verificar si la tabla parque_automotor fue creada.

Ver el número de filas cargadas.

Monitorear el estado de HBase.
