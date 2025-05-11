# Proyecto: Carga y Consulta del Parque Automotor en Apache HBase usando Happybase

## 📄 Descripción General

Este proyecto documenta el procedimiento para cargar un conjunto de datos del parque automotor de Colombia, proveniente de [Datos.gov.co](https://www.datos.gov.co/d/u3vn-bdcy), hacia una tabla en **Apache HBase**. Utiliza **Python** y la librería **Happybase** para conectarse a HBase, cargar datos masivos y realizar consultas básicas.

---

## 📅 1. Descarga del Dataset

Desde la máquina virtual donde está instalado HBase, descarga el archivo CSV y renómbralo para uso posterior:

```bash
wget https://www.datos.gov.co/api/views/u3vn-bdcy/rows.csv?accessType=DOWNLOAD -O parque_automotor.csv --no-check-certificate
```

✨ **Nota:** Asegúrate que el archivo se llame exactamente **parque\_automotor.csv** para que el script funcione sin errores.

---

## 🏠 2. Creación de la Tabla en HBase

**Nombre de la tabla:** `parque_automotor`

**Familias de columnas:**

* `ubicacion`: NOMBRE\_DEPARTAMENTO, NOMBRE\_MUNICIPIO
* `vehiculo`: NOMBRE\_SERVICIO, ESTADO\_DEL\_VEHICULO, NOMBRE\_DE\_LA\_CLASE
* `registro`: FECHA DE REGISTRO, CANTIDAD, MES DE PUBLICACION, AÑO DE PUBLICACIÓN

Antes de proceder:

```bash
start-hbase.sh
hbase-daemon.sh start thrift
```

---

## 💻 3. Creación del Script Python `consultas.py`

Crea el archivo:

```bash
nano consultas.py
```

Pega el contenido disponible en la carpeta del proyecto y ejecútalo:

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


```bash
python3 consultas.py
```

Este script realiza:

* Conexión con HBase
* Creación de tabla con familias
* Carga de los primeros 100 registros
* Consultas:

  * Visualización de los primeros 3
  * Filtro por municipio (ej. BOGOTA)
  * Conteo por tipo de servicio
  * Actualización de estado
  * Eliminación de un registro

---

## 🔧 4. Instalación de Dependencias

```bash
pip install happybase pandas
```

Estas librerías permiten conectar Python con HBase y manipular el archivo CSV.

---

## 📊 5. Verificación de la Tabla en HBase

Accede a la interfaz web de HBase:

```bash
http://<IP_MAQUINA_VIRTUAL>:16010
```

Ejemplo:

```
http://192.168.60.28:16010
```

Allí puedes:

* Verificar la creación de la tabla
* Consultar filas y columnas activas
* Monitorear nodos y servicios

---
