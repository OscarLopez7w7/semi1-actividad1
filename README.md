# Manual Técnico - ¡La Casita!

**Curso:** Seminario de Sistemas 1  
**Sección:** B  
**Grupo:** 3

## Información de los Estudiantes

| No. | Nombre | Carné |
|-----|--------|-------|
| 1   | Aída Alejandra Mansilla Orantes | 202100239 |
| 2   | Kevin Josué Santos Salazar | 202101007 |
| 3   | Oscar Fernando López Pirir | 201908376 |
| 4   | Ander Gilberto Popol Poron | 201801518 |

---
## Objetivos del proyecto

Implementar una solución cloud, para la solicitud de un restaurante, el cual desea digitalizar y automatizar la gestión de dicho restaurante.

Se desea utilizar herramientas de AWS para el manejo de la aplicación, implementar también herramientas de IA para facilitar y ampliar las opciones para poder ordenar algo del menú.

---
## Descripción del proyecto

El sistema de control de ventas para restaurantes *MI CASITA* es una solución cloud innovadora que digitaliza y automatiza la gestión operativa de un restaurante, desde la toma de órdenes hasta el cobro y la generación de reportes.

Con el sistema resuelve los siguientes aspectos del negocio:
- Control manual de órdenes propenso a errores
- Falta de comunicación eficiente entre meseras y cocinas
- Dificultad para generar reportes de ventas
- Pérdida de tiempo en procesos repetitivos

---
## Arquitectura del proyecto

El proyecto **MI CASITA** implementa una aplicación web de gestión de recetas utilizando una arquitectura basada en **servicios distribuidos** desplegados en Azure y AWS:

- **Frontend:** Se utilizó REACT y se alojo en una instancia de EC2.
- **Backend (API):** Servidor Node.js y se alojo en una instancia de EC2.
- **Base de Datos:** MySQL en Amazon RDS.
- **Almacenamiento de archivos estáticos:** Se utilizó Amazon S3
- **Notificación de ordenes:** Se utilizó Amazon Polly para marcar un pedido como "Listo" por medio de un texto y comunicar por audio a los meseros.
- **Atención a turistas:** Se utilizó Amazon Translate para poder traducir el menú del restaurante para turistas en múltiples idiomas.
- **Registro deigital de todas las ventas:** Se utilizó Amazon SNS.

#### Estructura de Base de Datos

**Tabla usuarios:**
```sql
CREATE TABLE Usuarios (
    id_usuario INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    rol ENUM('mesero', 'administrador', 'cocina superior', 'cocina inferior') NOT NULL,
    correo VARCHAR(100) UNIQUE NOT NULL,
    contraseña VARCHAR(255) NOT NULL
);
```

**Tabla Platillos:**
```sql
CREATE TABLE Platillos (
    id_platillo INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    descripcion TEXT NOT NULL,
    precio DECIMAL(10, 2) NOT NULL,
    cocina ENUM('Cocina Superior', 'Cocina Inferior', 'Ambas') NOT NULL,
    tipo VARCHAR(50) DEFAULT 'General',  -- Nueva columna para el tipo de platillo
    activo BOOLEAN DEFAULT TRUE,
    hora_inicio TIME DEFAULT '00:00:00',
    hora_fin TIME DEFAULT '00:00:00'
);
```

**Tabla Platillos por dia:**
```sql
CREATE TABLE Platillo_Dias (
    id_platillo INT,
    dia ENUM('Lunes', 'Martes', 'Miércoles', 'Jueves', 'Viernes', 'Sábado', 'Domingo') NOT NULL,
    activo BOOLEAN DEFAULT TRUE, -- Indica si el platillo está activo o no en el día específico
    PRIMARY KEY (id_platillo, dia), -- Asegura que cada combinación de id_platillo y dia sea única
    FOREIGN KEY (id_platillo) REFERENCES Platillos(id_platillo)
);
```

**Tabla Variantes:**
```sql
CREATE TABLE Variantes (
    id_variante INT AUTO_INCREMENT PRIMARY KEY,
    id_platillo INT NOT NULL, -- Asociar la variante a un platillo específico
    nombre VARCHAR(100) NOT NULL,
    descripcion TEXT NOT NULL,
    cantidad INT NOT NULL, -- Cantidad específica de la variante
    precio_extra DECIMAL(10, 2) DEFAULT 0.00, -- Precio adicional por la variante
    FOREIGN KEY (id_platillo) REFERENCES Platillos(id_platillo)
);
```

**Tabla Mesas:**
```sql
CREATE TABLE Mesas (
    id_mesa INT AUTO_INCREMENT PRIMARY KEY,
    numero_mesa INT NOT NULL,
    activo BOOLEAN DEFAULT TRUE, 
    posicion ENUM('Arriba','Abajo') 
);
```

**Tabla Clientes:**
```sql
CREATE TABLE Clientes (
    id_cliente INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) DEFAULT 'C/F',
    nit VARCHAR(20) DEFAULT 'C/F'
);
```

**Tabla Ordenes:**
```sql
CREATE TABLE Ordenes (
    id_orden INT AUTO_INCREMENT PRIMARY KEY,
    id_usuario INT,
    fecha_hora DATETIME NOT NULL,
    numero_mesa INT NULL, -- Permitir NULL para pedidos "Para llevar"
    tipo_orden ENUM('Para llevar', 'Para comer acá') NOT NULL,
    estado_orden ENUM('En preparación', 'Listo', 'Entregado', 'Pagado') DEFAULT 'En preparación',
    total_consumido DECIMAL(10, 2) NOT NULL DEFAULT 0.00,
    nota TEXT, -- Nueva columna para las notas de las meseras
    FOREIGN KEY (id_usuario) REFERENCES Usuarios(id_usuario),
    FOREIGN KEY (numero_mesa) REFERENCES Mesas(id_mesa),
    CHECK (
        tipo_orden = 'Para llevar' OR (tipo_orden = 'Para comer acá' AND numero_mesa IS NOT NULL)
    ) -- Restricción para asegurar que solo "Para comer acá" requiere un número de mesa
);

DELIMITER //

CREATE TRIGGER set_estado_en_preparacion
BEFORE INSERT ON Ordenes
FOR EACH ROW
BEGIN
  SET NEW.estado_orden = 'En preparación';
END;
//

```

**Tabla Orden Platillo:**
```sql
CREATE TABLE Orden_Platillo (
    id_orden_platillo INT AUTO_INCREMENT PRIMARY KEY,
    id_orden INT,
    id_platillo INT,
    cocina ENUM('Cocina Superior', 'Cocina Inferior') NOT NULL,
    cantidad INT NOT NULL,
    estado_platillo ENUM('En preparación', 'Listo','Entregado') DEFAULT 'En preparación',
    precio_platillo INT NOT NULL,
    estado_pago ENUM('Pendiente de pago', 'Pagado') DEFAULT 'Pendiente de pago', -- Nuevo campo para el estado de pago
    tipo_platillo_orden ENUM('Para llevar', 'Para comer acá') NOT NULL,
    FOREIGN KEY (id_orden) REFERENCES Ordenes(id_orden),
    FOREIGN KEY (id_platillo) REFERENCES Platillos(id_platillo)
);

DROP TRIGGER IF EXISTS set_estado_platillo_en_preparacion;
DELIMITER //

CREATE TRIGGER set_estado_platillo_en_preparacion
BEFORE INSERT ON Orden_Platillo
FOR EACH ROW
BEGIN
  -- Verificar si el platillo está en la lista de exclusión
  IF NEW.id_platillo IN (20, 21, 22, 23, 24, 25, 26, 97, 99, 100, 101, 106, 110, 111, 112, 113, 116, 121, 123, 124, 125, 129, 134, 136) THEN
    SET NEW.estado_platillo = 'Entregado';
  ELSE
    SET NEW.estado_platillo = 'En preparación';
  END IF;

  -- Asegurar que el estado de pago siempre sea 'Pendiente de pago'
  SET NEW.estado_pago = 'Pendiente de pago';
END;
//

DELIMITER ;
```

**Tabla Orden de palatillos variantes:**
```sql
CREATE TABLE Orden_Platillo_Variante (
    id_orden_platillo INT,
    id_variante INT,
    cantidad_variante INT NOT NULL, -- Cantidad de esta variante para esta orden
    precio_extra_variante DECIMAL(10, 2) DEFAULT 0.00, -- Precio adicional de la variante para esta orden
    PRIMARY KEY (id_orden_platillo, id_variante),
    FOREIGN KEY (id_orden_platillo) REFERENCES Orden_Platillo(id_orden_platillo),
    FOREIGN KEY (id_variante) REFERENCES Variantes(id_variante)
);
```

**Tabla Pagos:**
```sql
CREATE TABLE Pagos (
    id_pago INT AUTO_INCREMENT PRIMARY KEY,
    id_orden INT,
    id_usuario INT NOT NULL, -- Usuario que realiza el cobro
    total_a_pagar DECIMAL(10, 2) NOT NULL,
    tipo_pago ENUM('Efectivo', 'Tarjeta') NOT NULL,
    efectivo_recibido DECIMAL(10, 2) DEFAULT NULL,
    vuelto DECIMAL(10, 2) GENERATED ALWAYS AS (
        CASE 
            WHEN tipo_pago = 'Efectivo' THEN efectivo_recibido - total_a_pagar
            ELSE NULL 
        END
    ) STORED,
    fecha_hora DATETIME NOT NULL,
    extras DECIMAL(10, 2) NOT NULL,
    nota_extras TEXT, 
    FOREIGN KEY (id_orden) REFERENCES Ordenes(id_orden),
    FOREIGN KEY (id_usuario) REFERENCES Usuarios(id_usuario)
);
```

**Tabla Pagos Platillo:**
```sql
CREATE TABLE Pagos_Platillo (
    id_pago_platillo INT AUTO_INCREMENT PRIMARY KEY,
    id_pago INT,
    id_orden_platillo INT,
    FOREIGN KEY (id_pago) REFERENCES Pagos(id_pago),
    FOREIGN KEY (id_orden_platillo) REFERENCES Orden_Platillo(id_orden_platillo)
);
```

**Tabla Pagos Cliente:**
```sql
CREATE TABLE Pagos_Clientes (
    id_pago_cliente INT AUTO_INCREMENT PRIMARY KEY,
    id_pago INT,
    id_cliente INT,
    monto_pagado DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (id_pago) REFERENCES Pagos(id_pago),
    FOREIGN KEY (id_cliente) REFERENCES Clientes(id_cliente)
);
```


---
## Presupuesto

| Servicio | Especificaciones | Costo Mensual |
|-----|--------|-------|
| Amazon EC2   | 2 instancias t3.medium (2 vCPU, 4GB RAM) | $60.00 |
| EC2 Load Balancer   | Application Load Balancer | $22.00 |
| Amazon RDS   | MySQL db.t3.micro (1vCPU, 1GB RAM, 20GB) | $15.00 |
| Amazon S3   | 50 GB almacenamiento + 10,000 requests | $1.50 |
| Amazon Polly   | 10,000 caracteres/día (~300k/mes) | $4.00 |
| Amazon Translate   | 5,000 caracteres/día (~150k/mes) | $2.25 |
| Amazon SNS   | 10,000 emails/mes | $2.00 |
| Transferencia de Datos   | ~50 GB/mes | $4.50 |
| Total Estimado   | Costo mensual aproximado por todos los servicios | $111.25 |

---
## Servicios utilizados

-**Amazon EC2 (Elastic Compute Cloud):** Servicio que permite ejecutar servidores virtuales (instancias) en la nube para alojar aplicaciones, sitios web o procesos de cómputo.

-**EC2 Load Balancer (Elastic Load Balancing):** Distribuye automáticamente el tráfico entrante entre varias instancias EC2 para mejorar la disponibilidad y el rendimiento de las aplicaciones.

-**Amazon RDS (Relational Database Service):** Servicio administrado que facilita la configuración, operación y escalado de bases de datos relacionales como MySQL, PostgreSQL, MariaDB, Oracle o SQL Server.

-**Amazon S3 (Simple Storage Service):** Almacenamiento de objetos altamente escalable y seguro para guardar y recuperar cualquier cantidad de datos (como copias de seguridad, archivos o contenido web).

-**Amazon Polly:** Servicio de texto a voz que convierte texto en discurso natural usando tecnología de aprendizaje profundo.

-**Amazon Translate:** Servicio de traducción automática que convierte texto entre múltiples idiomas con alta precisión.

-**Amazon SNS (Simple Notification Service):** Servicio de mensajería y notificaciones que permite enviar mensajes o alertas a múltiples destinatarios o sistemas (como correos, SMS o colas).

-**Transferencia de Datos (Data Transfer):** Se refiere al movimiento de datos dentro o fuera de los servicios de AWS; puede implicar costos dependiendo del volumen y la dirección del tráfico (entrada o salida).

---


