# Sistema de Gestión Veterinaria - Proyecto Programación IV

## Descripción del Proyecto
Sistema de base de datos relacional implementado en **Oracle AI Database 26ai** para la administración integral de una clínica veterinaria. El modelo contempla el manejo de tablas transaccionales y catálogos del dominio (clientes, mascotas, especies, razas, veterinarios, citas, consultas y tratamientos), garantizando la integridad referencial mediante llaves primarias, foráneas, restricciones (`CHECK`, `UNIQUE`, `NOT NULL`), columnas `IDENTITY` y lógica de negocio encapsulada en PL/SQL con manejo de excepciones.

---

## Requisitos Previos
Para ejecutar y probar este proyecto en un entorno local necesitas contar con:
- **Oracle AI Database 26ai** (instancia local o en contenedor Docker).
- **Oracle SQL Developer** (o cualquier cliente SQL compatible con Oracle).

---

## Estructura del Repositorio
```text
├── database/
│   └── script_veterinaria.sql    # Script maestro (DDL de tablas, restricciones, datos INSERT y paquetes/procedimientos PL/SQL)
└── README.md                     # Documentación e instrucciones de ejecución
```

---

## Instrucciones de Ejecución para el Proyecto

### 1. Preparar la conexión en Oracle SQL Developer
1. Abre **Oracle SQL Developer**.
2. Crea o selecciona la conexión correspondiente a tu esquema de trabajo (por ejemplo: `proyecto-progra4`) con los permisos necesarios para crear tablas, secuencias y objetos PL/SQL.

### 2. Cargar la estructura, los datos y la lógica PL/SQL
1. En el menú superior, ve a **Archivo (File) > Abrir (Open...)** y selecciona el archivo `.sql` ubicado dentro de la carpeta `/database` de este repositorio.
2. Asegúrate de que el selector de conexión en la esquina superior derecha de la hoja de trabajo apunte a tu esquema (`proyecto-progra4`).
3. Ejecuta todo el contenido del archivo presionando **`F5`** (o haciendo clic en el botón **Ejecutar Script**).
4. Verifica en la pestaña *Salida de Script* que no existan errores de compilación y que se hayan creado:
   - Las tablas del modelo con sus llaves y restricciones.
   - Los registros de prueba iniciales (`INSERT`).
   - Los paquetes y procedimientos almacenados del CRUD.

### 3. Verificación rápida de funcionamiento
Una vez ejecutado el script, puedes comprobar que los datos se cargaron correctamente ejecutando una consulta básica en una hoja de trabajo:
```sql
SELECT * FROM TBL_CLIENTE;
SELECT * FROM TBL_MASCOTA;
```

---

## Detalle de Construcción de la Base de Datos

A continuación se documenta el orden lógico de creación de las tablas (respetando primero las tablas padre y luego las tablas con llaves foráneas), la carga de datos de prueba y los procedimientos almacenados.

### Paso 1: Creación de Tablas Catálogo y Entidades Base

#### 1.1 Tabla Especie (`TBL_ESPECIE`)
```sql
CREATE TABLE TBL_ESPECIE (
    ID_ESPECIE NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    NOMBRE     VARCHAR2(100 BYTE) NOT NULL
);
```

#### 1.2 Tabla Raza (`TBL_RAZA`)
```sql
CREATE TABLE TBL_RAZA (
    ID_RAZA    NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    NOMBRE     VARCHAR2(100 BYTE) NOT NULL,
    ID_ESPECIE NUMBER NOT NULL,
    CONSTRAINT TBL_RAZA_ESPECIE_FK FOREIGN KEY (ID_ESPECIE) REFERENCES TBL_ESPECIE (ID_ESPECIE)
);
```

#### 1.3 Tabla Cliente (`TBL_CLIENTE`)
*(Se crea antes de `TBL_MASCOTA` para respetar la llave foránea).*
```sql
CREATE TABLE TBL_CLIENTE (
    ID_CLIENTE NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    CEDULA     VARCHAR2(50 BYTE) NOT NULL,
    NOMBRE     VARCHAR2(100 BYTE) NOT NULL,
    APELLIDOS  VARCHAR2(100 BYTE) NOT NULL,
    TELEFONO   VARCHAR2(20 BYTE),
    CORREO     VARCHAR2(100 BYTE),
    DIRECCION  VARCHAR2(200 BYTE)
);
```

#### 1.4 Tabla Veterinario (`TBL_VETERINARIO`)
*(Se crea antes de `TBL_CITA` para respetar la llave foránea).*
```sql
CREATE TABLE TBL_VETERINARIO (
    ID_VETERINARIO NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    NOMBRE         VARCHAR2(100 BYTE) NOT NULL,
    APELLIDOS      VARCHAR2(100 BYTE) NOT NULL,
    CEDULA         VARCHAR2(50 BYTE) NOT NULL,
    TELEFONO       VARCHAR2(30 BYTE),
    CORREO         VARCHAR2(100 BYTE),
    ESPECIALIDAD   VARCHAR2(100 BYTE)
);
```

#### 1.5 Tabla Mascota (`TBL_MASCOTA`)
```sql
CREATE TABLE TBL_MASCOTA (
    ID_MASCOTA       NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    NOMBRE           VARCHAR2(100 BYTE) NOT NULL,
    FECHA_NACIMIENTO DATE,
    SEXO             VARCHAR2(20 BYTE),
    PESO             NUMBER,
    ID_RAZA          NUMBER NOT NULL,
    ID_CLIENTE       NUMBER NOT NULL,
    CONSTRAINT TBL_MASCOTA_CLIENTE_FK FOREIGN KEY (ID_CLIENTE) REFERENCES TBL_CLIENTE (ID_CLIENTE),
    CONSTRAINT TBL_MASCOTA_RAZA_FK FOREIGN KEY (ID_RAZA) REFERENCES TBL_RAZA (ID_RAZA)
);
```

---

### Paso 2: Creación de Tablas Transaccionales

#### 2.1 Tabla Cita (`TBL_CITA`)
```sql
CREATE TABLE TBL_CITA (
    ID_CITA        NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    ID_MASCOTA     NUMBER NOT NULL,
    ID_VETERINARIO NUMBER NOT NULL,
    FECHA          DATE NOT NULL,
    HORA           VARCHAR2(20 BYTE) NOT NULL,
    MOTIVO         VARCHAR2(300 BYTE),
    ESTADO         VARCHAR2(50 BYTE) NOT NULL,
    CONSTRAINT TBL_CITA_MASCOTA_FK FOREIGN KEY (ID_MASCOTA) REFERENCES TBL_MASCOTA (ID_MASCOTA),
    CONSTRAINT TBL_CITA_VETERINARIO_FK FOREIGN KEY (ID_VETERINARIO) REFERENCES TBL_VETERINARIO (ID_VETERINARIO)
);
```

#### 2.2 Tabla Consulta (`TBL_CONSULTA`)
```sql
CREATE TABLE TBL_CONSULTA (
    ID_CONSULTA   NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    ID_CITA       NUMBER NOT NULL,
    FECHA         DATE NOT NULL,
    SINTOMAS      VARCHAR2(300 BYTE),
    DIAGNOSTICO   VARCHAR2(300 BYTE) NOT NULL,
    OBSERVACIONES VARCHAR2(300 BYTE),
    CONSTRAINT TBL_CONSULTA_CITA_FK FOREIGN KEY (ID_CITA) REFERENCES TBL_CITA (ID_CITA)
);
```

#### 2.3 Tabla Tratamiento (`TBL_TRATAMIENTO`)
```sql
CREATE TABLE TBL_TRATAMIENTO (
    ID_TRATAMIENTO NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    ID_CONSULTA    NUMBER NOT NULL,
    MEDICAMENTO    VARCHAR2(100 BYTE) NOT NULL,
    DOSIS          VARCHAR2(50 BYTE) NOT NULL,
    FRECUENCIA     VARCHAR2(100 BYTE) NOT NULL,
    DURACION       VARCHAR2(50 BYTE) NOT NULL,
    INDICACIONES   VARCHAR2(300 BYTE),
    CONSTRAINT TBL_TRATAMIENTO_CONSULTA_FK FOREIGN KEY (ID_CONSULTA) REFERENCES TBL_CONSULTA (ID_CONSULTA)
);
```

---

### Paso 3: Carga de Datos Iniciales (`INSERT`)

Inserción de registros de prueba para validar la integridad y almacenamiento de las tablas:

```sql
--------------------------------------------------------
-- Archivo creado  - miércoles-noviembre-11-2026   
--------------------------------------------------------
REM INSERTING into PROYECTO_PROGRA_4.TBL_CITA
SET DEFINE OFF;
Insert into PROYECTO_PROGRA_4.TBL_CITA (ID_CITA,ID_MASCOTA,ID_VETERINARIO,FECHA,HORA,MOTIVO,ESTADO) values ('1','1','1',to_date('12/11/26','DD/MM/RR'),'09:00 AM','Control general y vacunación','Programada');
Insert into PROYECTO_PROGRA_4.TBL_CITA (ID_CITA,ID_MASCOTA,ID_VETERINARIO,FECHA,HORA,MOTIVO,ESTADO) values ('2','2','2',to_date('12/11/26','DD/MM/RR'),'10:00 AM','Limpieza de oídos y revisión felina','Programada');
Insert into PROYECTO_PROGRA_4.TBL_CITA (ID_CITA,ID_MASCOTA,ID_VETERINARIO,FECHA,HORA,MOTIVO,ESTADO) values ('3','3','1',to_date('12/11/26','DD/MM/RR'),'11:00 AM','Revisión por cojera en extremidad','Completada');
Insert into PROYECTO_PROGRA_4.TBL_CITA (ID_CITA,ID_MASCOTA,ID_VETERINARIO,FECHA,HORA,MOTIVO,ESTADO) values ('4','4','3',to_date('13/11/26','DD/MM/RR'),'01:30 PM','Control dermatológico por alergia en piel','Programada');
Insert into PROYECTO_PROGRA_4.TBL_CITA (ID_CITA,ID_MASCOTA,ID_VETERINARIO,FECHA,HORA,MOTIVO,ESTADO) values ('5','5','2',to_date('13/11/26','DD/MM/RR'),'02:30 PM','Desparasitación externa e interna','Programada');
Insert into PROYECTO_PROGRA_4.TBL_CITA (ID_CITA,ID_MASCOTA,ID_VETERINARIO,FECHA,HORA,MOTIVO,ESTADO) values ('6','6','1',to_date('14/11/26','DD/MM/RR'),'09:30 AM','Evaluación preoperatoria para cirugía','Programada');
Insert into PROYECTO_PROGRA_4.TBL_CITA (ID_CITA,ID_MASCOTA,ID_VETERINARIO,FECHA,HORA,MOTIVO,ESTADO) values ('7','7','3',to_date('14/11/26','DD/MM/RR'),'10:30 AM','Control de peso y dieta especializada','Programada');
Insert into PROYECTO_PROGRA_4.TBL_CITA (ID_CITA,ID_MASCOTA,ID_VETERINARIO,FECHA,HORA,MOTIVO,ESTADO) values ('8','8','2',to_date('15/11/26','DD/MM/RR'),'11:30 AM','Vacunación anual obligatoria','Programada');
Insert into PROYECTO_PROGRA_4.TBL_CITA (ID_CITA,ID_MASCOTA,ID_VETERINARIO,FECHA,HORA,MOTIVO,ESTADO) values ('9','9','1',to_date('15/11/26','DD/MM/RR'),'02:00 PM','Revisión general de rutina','Programada');
Insert into PROYECTO_PROGRA_4.TBL_CITA (ID_CITA,ID_MASCOTA,ID_VETERINARIO,FECHA,HORA,MOTIVO,ESTADO) values ('10','10','2',to_date('16/11/26','DD/MM/RR'),'03:00 PM','Consulta por decaimiento y pérdida de apetito','Programada');
REM INSERTING into PROYECTO_PROGRA_4.TBL_CLIENTE
SET DEFINE OFF;
Insert into PROYECTO_PROGRA_4.TBL_CLIENTE (ID_CLIENTE,CEDULA,NOMBRE,APELLIDOS,TELEFONO,CORREO,DIRECCION) values ('1','101110111','Carlos','Gómez Mora','8888-1111','carlos@mail.com','San José');
Insert into PROYECTO_PROGRA_4.TBL_CLIENTE (ID_CLIENTE,CEDULA,NOMBRE,APELLIDOS,TELEFONO,CORREO,DIRECCION) values ('2','202220222','María','Rodríguez Solís','8888-2222','maria@mail.com','Alajuela');
Insert into PROYECTO_PROGRA_4.TBL_CLIENTE (ID_CLIENTE,CEDULA,NOMBRE,APELLIDOS,TELEFONO,CORREO,DIRECCION) values ('3','303330333','José','Fernández Vargas','8888-3333','jose@mail.com','Heredia');
Insert into PROYECTO_PROGRA_4.TBL_CLIENTE (ID_CLIENTE,CEDULA,NOMBRE,APELLIDOS,TELEFONO,CORREO,DIRECCION) values ('4','404440444','Ana','Jiménez Campos','8888-4444','ana@mail.com','Cartago');
Insert into PROYECTO_PROGRA_4.TBL_CLIENTE (ID_CLIENTE,CEDULA,NOMBRE,APELLIDOS,TELEFONO,CORREO,DIRECCION) values ('5','505550555','Luis','Morales Rojas','8888-5555','luis@mail.com','Puntarenas');
Insert into PROYECTO_PROGRA_4.TBL_CLIENTE (ID_CLIENTE,CEDULA,NOMBRE,APELLIDOS,TELEFONO,CORREO,DIRECCION) values ('6','606660666','Sofía','Castro Monge','8888-6666','sofia@mail.com','Limón');
Insert into PROYECTO_PROGRA_4.TBL_CLIENTE (ID_CLIENTE,CEDULA,NOMBRE,APELLIDOS,TELEFONO,CORREO,DIRECCION) values ('7','707770777','David','Ramírez Quirós','8888-7777','david@mail.com','Guanacaste');
Insert into PROYECTO_PROGRA_4.TBL_CLIENTE (ID_CLIENTE,CEDULA,NOMBRE,APELLIDOS,TELEFONO,CORREO,DIRECCION) values ('8','808880888','Valeria','Salas Blanco','8888-8888','valeria@mail.com','San Ramón');
Insert into PROYECTO_PROGRA_4.TBL_CLIENTE (ID_CLIENTE,CEDULA,NOMBRE,APELLIDOS,TELEFONO,CORREO,DIRECCION) values ('9','909990999','Gabriel','Araya Picado','8888-9999','gabriel@mail.com','Naranjo');
Insert into PROYECTO_PROGRA_4.TBL_CLIENTE (ID_CLIENTE,CEDULA,NOMBRE,APELLIDOS,TELEFONO,CORREO,DIRECCION) values ('10','101010101','Lucía','Cordero Ureña','8888-0000','lucia@mail.com','Palmares');
REM INSERTING into PROYECTO_PROGRA_4.TBL_CONSULTA
SET DEFINE OFF;
REM INSERTING into PROYECTO_PROGRA_4.TBL_ESPECIE
SET DEFINE OFF;
Insert into PROYECTO_PROGRA_4.TBL_ESPECIE (ID_ESPECIE,NOMBRE) values ('1','Canino');
Insert into PROYECTO_PROGRA_4.TBL_ESPECIE (ID_ESPECIE,NOMBRE) values ('2','Felino');
REM INSERTING into PROYECTO_PROGRA_4.TBL_MASCOTA
SET DEFINE OFF;
Insert into PROYECTO_PROGRA_4.TBL_MASCOTA (ID_MASCOTA,NOMBRE,FECHA_NACIMIENTO,SEXO,PESO,ID_RAZA,ID_CLIENTE) values ('1','Max',to_date('10/05/22','DD/MM/RR'),'Macho','25,5','1','1');
Insert into PROYECTO_PROGRA_4.TBL_MASCOTA (ID_MASCOTA,NOMBRE,FECHA_NACIMIENTO,SEXO,PESO,ID_RAZA,ID_CLIENTE) values ('2','Luna',to_date('15/01/23','DD/MM/RR'),'Hembra','4,2','4','2');
Insert into PROYECTO_PROGRA_4.TBL_MASCOTA (ID_MASCOTA,NOMBRE,FECHA_NACIMIENTO,SEXO,PESO,ID_RAZA,ID_CLIENTE) values ('3','Thor',to_date('20/11/21','DD/MM/RR'),'Macho','32','2','3');
Insert into PROYECTO_PROGRA_4.TBL_MASCOTA (ID_MASCOTA,NOMBRE,FECHA_NACIMIENTO,SEXO,PESO,ID_RAZA,ID_CLIENTE) values ('4','Mila',to_date('05/06/23','DD/MM/RR'),'Hembra','12','3','4');
Insert into PROYECTO_PROGRA_4.TBL_MASCOTA (ID_MASCOTA,NOMBRE,FECHA_NACIMIENTO,SEXO,PESO,ID_RAZA,ID_CLIENTE) values ('5','Simba',to_date('12/08/22','DD/MM/RR'),'Macho','5,1','5','5');
Insert into PROYECTO_PROGRA_4.TBL_MASCOTA (ID_MASCOTA,NOMBRE,FECHA_NACIMIENTO,SEXO,PESO,ID_RAZA,ID_CLIENTE) values ('6','Kira',to_date('30/03/20','DD/MM/RR'),'Hembra','20','1','6');
Insert into PROYECTO_PROGRA_4.TBL_MASCOTA (ID_MASCOTA,NOMBRE,FECHA_NACIMIENTO,SEXO,PESO,ID_RAZA,ID_CLIENTE) values ('7','Rocky',to_date('18/02/23','DD/MM/RR'),'Macho','14,5','3','7');
Insert into PROYECTO_PROGRA_4.TBL_MASCOTA (ID_MASCOTA,NOMBRE,FECHA_NACIMIENTO,SEXO,PESO,ID_RAZA,ID_CLIENTE) values ('8','Nala',to_date('14/09/21','DD/MM/RR'),'Hembra','4,8','6','8');
Insert into PROYECTO_PROGRA_4.TBL_MASCOTA (ID_MASCOTA,NOMBRE,FECHA_NACIMIENTO,SEXO,PESO,ID_RAZA,ID_CLIENTE) values ('9','Zeus',to_date('01/12/22','DD/MM/RR'),'Macho','35','2','9');
Insert into PROYECTO_PROGRA_4.TBL_MASCOTA (ID_MASCOTA,NOMBRE,FECHA_NACIMIENTO,SEXO,PESO,ID_RAZA,ID_CLIENTE) values ('10','Cleo',to_date('22/04/23','DD/MM/RR'),'Hembra','3,9','4','10');
REM INSERTING into PROYECTO_PROGRA_4.TBL_RAZA
SET DEFINE OFF;
Insert into PROYECTO_PROGRA_4.TBL_RAZA (ID_RAZA,NOMBRE,ID_ESPECIE) values ('1','Golden Retriever','1');
Insert into PROYECTO_PROGRA_4.TBL_RAZA (ID_RAZA,NOMBRE,ID_ESPECIE) values ('2','Pastor Alemán','1');
Insert into PROYECTO_PROGRA_4.TBL_RAZA (ID_RAZA,NOMBRE,ID_ESPECIE) values ('3','Bulldog Francés','1');
Insert into PROYECTO_PROGRA_4.TBL_RAZA (ID_RAZA,NOMBRE,ID_ESPECIE) values ('4','Siamés','2');
Insert into PROYECTO_PROGRA_4.TBL_RAZA (ID_RAZA,NOMBRE,ID_ESPECIE) values ('5','Persa','2');
Insert into PROYECTO_PROGRA_4.TBL_RAZA (ID_RAZA,NOMBRE,ID_ESPECIE) values ('6','Maine Coon','2');
REM INSERTING into PROYECTO_PROGRA_4.TBL_TRATAMIENTO
SET DEFINE OFF;
REM INSERTING into PROYECTO_PROGRA_4.TBL_VETERINARIO
SET DEFINE OFF;
Insert into PROYECTO_PROGRA_4.TBL_VETERINARIO (ID_VETERINARIO,NOMBRE,APELLIDOS,CEDULA,TELEFONO,CORREO,ESPECIALIDAD) values ('1','Esteban','Mora Vargas','102220333','8787-1111','dr.mora@vet.com','Cirugía General');
Insert into PROYECTO_PROGRA_4.TBL_VETERINARIO (ID_VETERINARIO,NOMBRE,APELLIDOS,CEDULA,TELEFONO,CORREO,ESPECIALIDAD) values ('2','Mariana','Picado Rojas','203330444','8787-2222','dra.picado@vet.com','Medicina Felina');
Insert into PROYECTO_PROGRA_4.TBL_VETERINARIO (ID_VETERINARIO,NOMBRE,APELLIDOS,CEDULA,TELEFONO,CORREO,ESPECIALIDAD) values ('3','Roberto','Quesada Soto','304440555','8787-3333','dr.quesada@vet.com','Dermatología');


---

### Paso 4: Procedimientos Almacenados (CRUD Clientes)

#### 4.1 Guardar Cliente (`SP_GUARDAR_CLIENTE`)
```sql
CREATE OR REPLACE PROCEDURE SP_GUARDAR_CLIENTE (
    p_cedula    IN tbl_cliente.cedula%TYPE,
    p_nombre    IN tbl_cliente.nombre%TYPE,
    p_apellidos IN tbl_cliente.apellidos%TYPE,
    p_telefono  IN tbl_cliente.telefono%TYPE,
    p_correo    IN tbl_cliente.correo%TYPE,
    p_direccion IN tbl_cliente.direccion%TYPE,
    p_respuesta OUT NUMBER
) AS 
BEGIN
    INSERT INTO tbl_cliente (
        cedula, nombre, apellidos, telefono, correo, direccion
    ) VALUES (
        p_cedula, p_nombre, p_apellidos, p_telefono, p_correo, p_direccion
    );
    
    p_respuesta := 1;
    COMMIT;

EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        p_respuesta := -1;
        RAISE;
END SP_GUARDAR_CLIENTE;
/
```

#### 4.2 Actualizar Cliente (`SP_ACTUALIZAR_CLIENTE`)
```sql
CREATE OR REPLACE PROCEDURE SP_ACTUALIZAR_CLIENTE (
    p_id_cliente IN tbl_cliente.id_cliente%TYPE,
    p_telefono   IN tbl_cliente.telefono%TYPE,
    p_correo     IN tbl_cliente.correo%TYPE,
    p_direccion  IN tbl_cliente.direccion%TYPE,
    p_respuesta  OUT NUMBER
) AS 
BEGIN
    UPDATE tbl_cliente
    SET telefono  = p_telefono,
        correo    = p_correo,
        direccion = p_direccion
    WHERE id_cliente = p_id_cliente;
    
    IF SQL%ROWCOUNT > 0 THEN
        p_respuesta := 1;
        COMMIT;
    ELSE
        p_respuesta := -1;
    END IF;

EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        p_respuesta := -1;
        RAISE;
END SP_ACTUALIZAR_CLIENTE;
/
```

#### 4.3 Listar Clientes (`SP_LISTAR_CLIENTES`)
```sql
CREATE OR REPLACE PROCEDURE SP_LISTAR_CLIENTES (
    p_cursor OUT SYS_REFCURSOR
) AS 
BEGIN
    OPEN p_cursor FOR
        SELECT 
            id_cliente, cedula, nombre, apellidos, telefono, correo, direccion
        FROM tbl_cliente
        ORDER BY id_cliente;

EXCEPTION
    WHEN OTHERS THEN
        RAISE;
END SP_LISTAR_CLIENTES;
/
```

#### 4.4 Eliminar Cliente (`SP_ELIMINAR_CLIENTE`)
```sql
CREATE OR REPLACE PROCEDURE SP_ELIMINAR_CLIENTE (
    p_id_cliente IN tbl_cliente.id_cliente%TYPE,
    p_respuesta  OUT NUMBER
) AS 
BEGIN
    DELETE FROM tbl_cliente 
    WHERE id_cliente = p_id_cliente;
    
    IF SQL%ROWCOUNT > 0 THEN
        p_respuesta := 1;
        COMMIT;
    ELSE
        p_respuesta := -1;
    END IF;

EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        p_respuesta := -1;
        RAISE;
END SP_ELIMINAR_CLIENTE;
/
```

---

## Funcionalidades Implementadas en Base de Datos
- **Catálogos y Entidades:** Gestión de Especies, Razas, Clientes, Mascotas y Veterinarios.
- **Tablas Transaccionales:** Registro y control de Citas, Consultas Médicas y Tratamientos.
- **Lógica PL/SQL:** Operaciones CRUD estructuradas con parámetros de entrada/salida y bloques `EXCEPTION` para controlar errores de integridad o registros no encontrados.