
# 🧠 GUÍA COMPLETA DE LIQUIBASE

## Liquibase: Control de versiones para bases de datos

###  1. ¿Qué es Liquibase?

Liquibase es una herramienta open source que permite gestionar y versionar cambios en bases de datos de forma automatizada, controlada y repetible.

Así como Git controla las versiones del código fuente, Liquibase controla las versiones del esquema y los datos de la base de datos.

Es ampliamente utilizada en entornos de desarrollo DevOps, CI/CD y microservicios, donde múltiples desarrolladores y entornos necesitan mantener sincronizados los cambios estructurales en la base de datos.

###  2. ¿Por qué se necesita Liquibase?
 Problemas comunes sin control de versiones en BD:

- Scripts desordenados o duplicados: cada desarrollador modifica la base por su cuenta.
- Desincronización entre entornos: dev, test y prod tienen estructuras distintas.
- Errores humanos al ejecutar scripts manualmente.
- Falta de trazabilidad: no se sabe quién cambió qué ni cuándo.
- Difícil rollback: revertir cambios es inseguro o manual.

Liquibase soluciona todo esto mediante un enfoque declarativo y versionado.

###  3. Objetivos de Liquibase

-  Controlar el historial completo de cambios en la base de datos.
-  Aplicar migraciones de manera incremental y segura.
-  Automatizar despliegues y rollbacks.
-  Facilitar la colaboración entre desarrolladores.
-  Integrar la base de datos en el flujo de CI/CD.
-  Mantener coherencia entre múltiples entornos.

###  4. ¿Cómo funciona Liquibase?

Liquibase no almacena la base de datos en sí, sino una historia de los cambios que se deben aplicar.

El flujo general es:

1. El desarrollador crea un archivo de cambio (ChangeLog).
2. Ejecuta el comando `liquibase update`.
3. Liquibase:
   - Lee los cambios pendientes.
   - Los aplica a la base de datos.
   - Registra qué se aplicó en una tabla interna (DATABASECHANGELOG).
   - Si se vuelve a ejecutar, solo aplica lo nuevo.
   - Puede hacer rollback, status, diffs, etc.

###  5. Componentes principales

| Componente          | Descripción                                                     |
|---------------------|----------------------------------------------------------------|
| ChangeSet           | Unidad mínima de cambio (crear tabla, insertar dato, etc.).   |
| ChangeLog           | Archivo que agrupa uno o más ChangeSets. Puede estar en XML, YAML, JSON o SQL. |
| DatabaseChangeLog   | Tabla interna que guarda qué cambios ya se aplicaron.          |
| Liquibase.properties | Archivo con los parámetros de conexión y configuración.         |
| Rollback            | Definición de cómo revertir un ChangeSet.                      |

###  6. Requisitos

Liquibase no es una herramienta “mágica” que corre sola: requiere ciertos elementos que le permiten conectarse y ejecutar cambios sobre bases de datos. Veamos uno a uno:

| Requisito                     | Descripción y explicación técnica                                                                                                                                                                                                                                                                                                                                                                                  |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Java 11 o superior**        | Liquibase está desarrollado en **Java**, por lo tanto necesita una **JVM (Java Virtual Machine)** instalada. Esto le permite ejecutarse en cualquier sistema operativo (Windows, macOS o Linux). <br> Si no tienes Java, Liquibase no podrá ejecutarse ni interpretar sus comandos CLI. <br>Comando para verificar: `java -version`.                                                                             |
| **Motor de base de datos**    | Liquibase no tiene su propia base, sino que **se conecta a una existente** (PostgreSQL, MySQL, SQL Server, Oracle, etc.) mediante JDBC. <br> Requiere que la base esté accesible (local o remota) y que acepte conexiones TCP.                                                                                                                                                                                   |
| **Driver JDBC**               | JDBC (Java Database Connectivity) es una **interfaz de conexión entre Java y las bases de datos**. <br>El driver JDBC actúa como “traductor” entre Liquibase (Java) y el motor de base. <br> Ejemplo: PostgreSQL usa `org.postgresql.Driver` y su archivo JAR (`postgresql-42.2.xx.jar`). <br>Este driver debe estar disponible en el **classpath**, es decir, en la carpeta donde Liquibase lo puede encontrar. |
| **Acceso a la base de datos** | El usuario con el que Liquibase se conecta debe tener permisos suficientes para ejecutar cambios estructurales: **CREATE, ALTER, DROP, INSERT, UPDATE, DELETE**. <br> Sin estos permisos, Liquibase fallará al intentar aplicar los ChangeSets.                                                                                                                                                                  |


### 🔹 7. Instalación

Liquibase puede instalarse de varias maneras dependiendo del sistema operativo.
A continuación te explico cada método, qué hace y cómo verificar la instalación.

#### 🧩 7.1. Windows
```bash
choco install liquibase
```
🔹 Chocolatey es un gestor de paquetes para Windows (similar a apt o brew en Linux/macOS).
El comando descarga la versión más reciente de Liquibase, la descomprime y la añade a la variable de entorno PATH, para que puedas usar el comando liquibase desde cualquier carpeta.

#### 🧩 7.2. macOS
```bash
brew install liquibase
```

🔹 Homebrew es el gestor de paquetes más usado en macOS.
Este comando instala Liquibase y todas sus dependencias, dejándolo listo en el sistema.

#### 🧩 7.3. Linux
```bash
sudo snap install liquibase
```

🔹 Snap es un sistema de distribución universal para Linux.
Con este comando, el sistema instala Liquibase como un “paquete autocontenido”, incluyendo binarios, scripts y plantillas.

#### 🧩 7.4. Verificar instalación
```bash
liquibase --version
```

📘 Qué hace:
El comando ejecuta el binario de Liquibase y solicita que imprima su versión.
Si todo está correcto, devuelve algo como:
**Resultado esperado:**
> Liquibase Community 4.x.x

💡 Si no aparece la versión, puede que:

Java no esté instalado.
La variable de entorno PATH no incluya Liquibase.
O se requiera reiniciar la terminal.

### 🔹 8. Estructura típica de un proyecto Liquibase

Liquibase organiza sus archivos de forma modular.
El siguiente árbol muestra una estructura ideal:

```plaintext
📁 project/
 ├── db/
 │   ├── changelog-master.xml
 │   ├── changes/
 │   │   ├── 001-create-tables.xml
 │   │   ├── 002-insert-data.xml
 │   └── liquibase.properties
 ├── src/
 └── ...
```

Explicación de cada parte:

| Carpeta / Archivo        | Propósito                                                                                   |
| ------------------------ | ------------------------------------------------------------------------------------------- |
| **db/**                  | Carpeta raíz que contiene toda la configuración y los archivos de cambios de Liquibase.     |
| **liquibase.properties** | Archivo de configuración principal (conexión, credenciales, archivo maestro, driver, etc.). |
| **changelog-master.xml** | Archivo maestro que agrupa e incluye todos los cambios que Liquibase debe aplicar.          |
| **changes/**             | Carpeta que contiene los archivos individuales de cambios (ChangeSets).                     |
| **src/**                 | Código fuente del proyecto (no obligatorio, puede estar separado).                          |

💡 Importante:
Liquibase solo necesita el contenido de la carpeta db/ para funcionar. Todo lo demás pertenece al proyecto de aplicación.

### 🔹 9. Archivo de configuración liquibase.properties
Este archivo actúa como puente de conexión entre Liquibase y la base de datos.
Define conexión, archivo maestro y otros parámetros.

```properties
changeLogFile=db/changelog-master.xml
url=jdbc:postgresql://localhost:5432/demo_liquibase
username=postgres
password=admin
driver=org.postgresql.Driver
classpath=postgresql-42.2.20.jar
```
Detalle de cada propiedad:

| Propiedad               | Qué hace                                                                                                                                     |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| **changeLogFile**       | Indica la ubicación del changelog principal (archivo maestro). Liquibase inicia desde aquí para saber qué cambios ejecutar.                  |
| **url**                 | Es la cadena de conexión JDBC. Tiene el formato: `jdbc:<motor>://<host>:<puerto>/<base>`.<br>Ejemplo: `jdbc:mysql://127.0.0.1:3306/mi_base`. |
| **username / password** | Credenciales con las que Liquibase se conectará a la base de datos.                                                                          |
| **driver**              | Clase Java del controlador JDBC que permite la conexión (debe coincidir con el motor de base).                                               |
| **classpath**           | Indica la ruta donde está el archivo `.jar` del driver JDBC, en caso de que Liquibase no lo tenga integrado.                                 |

💡 Si no especificas classpath, Liquibase intentará usar los drivers que ya vienen embebidos.
PostgreSQL, por ejemplo, normalmente ya está incluido en la instalación estándar.

### 🔹 10. El archivo maestro changelog-master.xml

Este es el punto de entrada de todos los cambios versionados.
Aquí se “incluyen” los demás archivos de cambio.

```xml
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd">

    <include file="changes/001-create-user.xml"/>
    <include file="changes/002-insert-user.xml"/>

</databaseChangeLog>
```

📘 Explicación técnica:

<databaseChangeLog> → elemento raíz obligatorio; define el espacio de nombres XML (namespace).
xmlns y xsi → aseguran que el XML use la estructura reconocida por Liquibase.
<include file="..."/> → instrucción para incluir archivos de cambio externos en orden.
Liquibase procesará los archivos en el orden en que aparecen, aplicando cada uno si aún no fue ejecutado.

### 🔹 11. Ejemplo de un ChangeSet

```xml
<changeSet id="001" author="karol">
  <createTable tableName="users">
    <column name="id" type="serial" />
    <column name="username" type="varchar(100)" />
    <column name="email" type="varchar(150)" />
  </createTable>
</changeSet>
```
### Qué es un ChangeSet:

Un ChangeSet es la unidad mínima de cambio en Liquibase.
Cada ChangeSet tiene un id único (por autor) y representa una acción atómica: crear, modificar, insertar, borrar, etc.

Liquibase guarda internamente:

- id del ChangeSet

- author

- archivo origen

- fecha y checksum (hash)

📘 Así, si intentas ejecutar el mismo ChangeSet dos veces, Liquibase detectará que ya fue aplicado y lo omitirá.

### 🔹 12. Comandos fundamentales

| Comando                           | Propósito y funcionamiento                                                                                                                                                                                                                         | Ejemplo                       |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------- |
| **`liquibase update`**            | Aplica todos los ChangeSets **pendientes** (es decir, que no están registrados en la tabla `DATABASECHANGELOG`). <br> Liquibase compara la lista de cambios definidos en tu changelog con los que ya aplicó y ejecuta solo los nuevos.             | `liquibase update`            |
| **`liquibase rollbackCount 1`**   | Revierte los últimos *N* ChangeSets aplicados. <br>El número indica cuántos revertir (1 = el último). <br>Solo funciona si esos ChangeSets tienen definido un bloque `<rollback>`.                                                                 | `liquibase rollbackCount 1`   |
| **`liquibase status`**            | Muestra un resumen del estado actual: qué cambios faltan por aplicar, cuántos se han ejecutado y si hay inconsistencias.                                                                                                                           | `liquibase status`            |
| **`liquibase history`**           | Lista todos los ChangeSets ejecutados, mostrando su id, autor, fecha y archivo de origen. <br>Es útil para auditoría y ver qué cambios se aplicaron en qué orden.                                                                                  | `liquibase history`           |
| **`liquibase changelogSync`**     | Marca todos los ChangeSets actuales como “aplicados” sin ejecutarlos. <br>Útil cuando ya aplicaste los cambios manualmente y solo quieres sincronizar el historial.                                                                                | `liquibase changelogSync`     |
| **`liquibase clearCheckSums`**    | Liquibase usa *checksums* (hashes) para detectar si un ChangeSet fue modificado. <br>Si cambias algo en un archivo ya aplicado, Liquibase lanza error. <br>Con este comando limpias los checksums para que se recalculen. ⚠️ Úsalo con precaución. | `liquibase clearCheckSums`    |
| **`liquibase diff`**              | Compara dos bases de datos (por ejemplo, “dev” y “prod”) y genera un reporte de diferencias estructurales. <br>Sirve para validar si ambas están sincronizadas.                                                                                    | `liquibase diff`              |
| **`liquibase generateChangeLog`** | Crea un archivo changelog inicial a partir de una base existente (escanea las tablas, columnas y datos). <br>Ideal para proyectos que ya tienen una base y quieren empezar a versionarla.                                                          | `liquibase generateChangeLog` |

💬 En resumen:

update aplica cambios.

rollbackCount revierte.
status y history informan.
changelogSync sincroniza sin aplicar.
diff compara.
generateChangeLog crea el punto de partida.

### 🔹 13. Ejemplo práctico paso a paso (DEMO GUIADA)
Aquí hacemos un recorrido realista del uso completo de Liquibase.


#### 🧩 PASO 1 — Preparar estructura de proyecto
Liquibase necesita:

Un archivo de propiedades (liquibase.properties).

Un archivo maestro (changelog-master.xml).

Una carpeta con los cambios (changes/).

```plaintext
📁 db/
 ├── liquibase.properties
 ├── changelog-master.xml
 └── changes/
      └── 001-create-products.xml
```

#### 🧩 PASO 2 — Configurar liquibase.properties

Contiene la conexión y parámetros básicos.

```properties
changeLogFile=db/changelog-master.xml
url=jdbc:postgresql://localhost:5432/liquibase_demo
username=postgres
password=Admin123
driver=org.postgresql.Driver
```

Liquibase usará esto cada vez que se ejecute un comando, así no tienes que escribir toda la configuración en la línea de comandos.

#### 🧩 PASO 3 — Crear changelog-master.xml
Define los cambios que se van a aplicar.

```xml
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd">

    <include file="changes/001-create-products.xml"/>
</databaseChangeLog>
```

Liquibase recorrerá este archivo y ejecutará los includes en orden.

#### 🧩 PASO 4 — Crear el primer ChangeSet 001-create-products.xml

```xml
<changeSet id="001" author="yerson">
  <createTable tableName="products">
    <column name="id" type="serial" />
    <column name="name" type="varchar(100)" />
    <column name="price" type="decimal(10,2)" />
  </createTable>
</changeSet>
```
💬 Aquí definimos qué estructura nueva se creará.
Liquibase traduce este XML en SQL nativo de PostgreSQL y lo ejecuta.


#### 🧩 PASO 5 — Ejecutar el comando principal

```bash
liquibase update
```

🔍 **¿Qué ocurre internamente?**

- Liquibase lee `liquibase.properties`.
- Se conecta a la base de datos.
- Lee el `changelog-master.xml`.
- Identifica los ChangeSets pendientes.
- Aplica los cambios.
- Registra en `DATABASECHANGELOG` que el id 001 ya fue ejecutado.

#### 🧩 PASO 6 — Verificar en la base de datos

En PostgreSQL:

```sql
SELECT * FROM products;
SELECT * FROM databasechangelog;
```

Verás:

- La tabla `products` creada.
- El registro del cambio en `databasechangelog`.

#### 🧩 PASO 7 — Agregar nuevo cambio (insertar datos)
Creamos un nuevo ChangeSet 002-insert-products.xml.


```xml
<changeSet id="002" author="karol">
  <insert tableName="products">
    <column name="name" value="Laptop" />
    <column name="price" value="2500.00" />
  </insert>
</changeSet>
```
Y lo añadimos al maestro:

```xml
<include file="changes/002-insert-products.xml"/>
```

Ejecutar:

```bash
liquibase update
```

🔍 **Liquibase solo aplicará lo nuevo (id=002), ya que el 001 ya está en el log.**

#### 🧩 PASO 8 — Rollback (revertir último cambio)

```bash
liquibase rollbackCount 1
```

Liquibase eliminará el último ChangeSet aplicado, si tiene rollback definido.

Podemos definir rollback en el mismo archivo:

```xml
<changeSet id="002" author="karol">
  <insert tableName="products">
    <column name="name" value="Laptop" />
    <column name="price" value="2500.00" />
  </insert>
  <rollback>
    <delete tableName="products">
      <where>name='Laptop'</where>
    </delete>
  </rollback>
</changeSet>
```

#### 🧩 PASO 9 — Ver estado actual

```bash
liquibase status
```

Muestra qué cambios aún no se han ejecutado.

### 🔹 14. Cómo se integra Liquibase con CI/CD

Liquibase puede ejecutarse en pipelines (Jenkins, GitLab, GitHub Actions) con un simple comando:

```bash
liquibase update --defaultsFile=db/liquibase.properties
```

Cada vez que se despliega una nueva versión, Liquibase aplica los cambios pendientes. Si algo falla → rollback.

### 🔹 15. Buenas prácticas

- ✅ Numerar los archivos (001, 002, …).
- ✅ No modificar un ChangeSet ya aplicado.
- ✅ Usar un changelog maestro que los incluya todos.
- ✅ Añadir rollback siempre que sea posible.
- ✅ Versionar el directorio `db/` junto con el código (Git).
- ✅ Probar los cambios primero en entornos de staging.

### 🔹 16. Ventajas clave de Liquibase

| Ventaja         | Explicación                                               |
|------------------|----------------------------------------------------------|
| Seguridad        | No ejecuta el mismo cambio dos veces.                   |
| Control          | Cada cambio queda registrado con autor, fecha y checksum. |
| Automatización    | Compatible con scripts y pipelines.                      |
| Trazabilidad     | Permite auditar el historial de migraciones.            |
| Reversibilidad   | Soporta rollback automático.                             |

### 🔹 17. Conclusión general

Liquibase permite:

- Mantener la consistencia entre entornos.
- Evitar errores humanos en despliegues.
- Tener historial completo y auditable de la base de datos.
- Integrar el ciclo DevOps de manera uniforme: código + base.

💬 Liquibase convierte los scripts SQL en historia controlada.
