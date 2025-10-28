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

- **Frontend:** Aplicación web estática alojada en Azure Blob Storage.
- **Backend (API):** Servidor Node.js (Persona A y B) ejecutándose en Azure VM.
- **Base de Datos:** MySQL en Amazon RDS.
- **Balanceador de Carga:** Azure Load Balancer para distribuir peticiones entre APIs.
- **Almacenamiento de Imágenes:** Azure Blob Storage para fotos de perfil y recetas.
- **Gestor de Identidad:** AWS IAM para permisos de acceso a RDS.

#### Estructura de Base de Datos

**Tabla usuarios:**
```sql
CREATE TABLE usuarios (
    id_usuario INT AUTO_INCREMENT PRIMARY KEY,
    nombre_usuario VARCHAR(50) UNIQUE NOT NULL,
    correo_electronico VARCHAR(100) UNIQUE NOT NULL,
    contraseña_encriptada VARCHAR(255) NOT NULL,
    url_foto_perfil VARCHAR(500),
    fecha_registro TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_nombre_usuario (nombre_usuario),
    INDEX idx_correo (correo_electronico)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;  

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


