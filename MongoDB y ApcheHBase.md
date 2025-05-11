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
