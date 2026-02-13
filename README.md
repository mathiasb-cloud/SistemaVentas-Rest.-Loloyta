# 🍽️ Loloyta - Sistema de Gestión de Restaurante

Sistema web completo para la gestión de pedidos, cocina, ventas y administración de un restaurante desarrollado con **Spring Boot** y **MySQL**.

![Java](https://img.shields.io/badge/Java-17+-orange)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.5.0-brightgreen)
![MySQL](https://img.shields.io/badge/MySQL-8.0-blue)
![WebSocket](https://img.shields.io/badge/WebSocket-STOMP-red)

## 📋 Características Principales

### 🎯 Módulos del Sistema

1. **👨‍🍳 Módulo de Cocina**
   - Recepción de pedidos en tiempo real vía WebSocket
   - Actualización de estado de pedidos (Recibido/Preparando)
   - Visualización de pedidos por mesa
   - Sincronización automática con todas las pantallas

2. **🍽️ Módulo de Mesas (Mozo)**
   - 6 mesas independientes con sus propios pedidos
   - Selección de items desde la carta digital
   - Edición de detalles de pedidos (ej: "sin cebolla")
   - Cálculo automático de totales
   - Confirmación de pago
   - Envío de pedidos a cocina en tiempo real

3. **💰 Módulo de Ventas**
   - Registro automático de ventas al confirmar pago
   - Filtrado por rango de fechas
   - Cálculo de totales de ventas
   - Historial completo de transacciones
   - Exportación de reportes con JasperReports

4. **📋 Módulo de Carta**
   - CRUD completo de items del menú
   - Clasificación por tipo (Platillo/Bebida/Postre)
   - Control de disponibilidad en tiempo real
   - Gestión de precios
   - Solo items disponibles aparecen en las mesas

5. **🔐 Sistema de Autenticación**
   - Login con roles diferenciados:
     - **Admin**: Acceso a ventas y reportes
     - **Mozo**: Acceso a mesas y toma de pedidos
     - **Jefe de Cocina**: Gestión de pedidos en cocina
     - **Administrador**: Gestión completa de la carta

## 🛠️ Tecnologías Utilizadas

### Backend
- **Spring Boot 3.5.0**
  - Spring Web
  - Spring Data JPA
  - Spring WebSocket
  - Spring Validation
- **MySQL 8.0**
- **Hibernate 6.6.15**
- **STOMP Protocol** para WebSocket
- **JasperReports** para generación de reportes

### Frontend
- **Thymeleaf** (Template Engine)
- **Bootstrap 5.3.3**
- **JavaScript ES6+**
- **SockJS & STOMP.js** para WebSocket
- **CSS personalizado** con efectos glassmorphism

## 📦 Estructura del Proyecto

```
loloytaApp/
├── src/main/java/com/example/demo/
│   ├── controladores/
│   │   ├── CocinaController.java          # Endpoints de cocina
│   │   ├── CocinaWebSocketController.java # WebSocket de pedidos
│   │   ├── VentaController.java           # Gestión de ventas
│   │   ├── CartaController.java           # CRUD de carta
│   │   └── UsuariosController.java        # Autenticación
│   ├── modelos/
│   │   ├── Cocina.java                    # Entidad de pedidos
│   │   ├── Venta.java                     # Entidad de ventas
│   │   ├── Carta.java                     # Entidad de items del menú
│   │   └── Usuario.java                   # Entidad de usuarios
│   ├── repositorio/
│   │   ├── CocinaRepository.java
│   │   ├── VentaRepository.java
│   │   └── CartaRepository.java
│   ├── servicios/
│   │   └── CocinaService.java
│   └── config/
│       └── WebSocketConfig.java           # Configuración STOMP
├── src/main/resources/
│   ├── templates/
│   │   ├── mesa1.html - mesa6.html        # Vistas de mesas
│   │   ├── cocina.html                    # Panel de cocina
│   │   ├── ventas.html                    # Módulo de ventas
│   │   ├── carta.html                     # Gestión de carta
│   │   ├── mesas.html                     # Selector de mesas
│   │   └── registro.html                  # Login
│   ├── static/
│   │   └── img/                           # Imágenes y logos
│   └── application.properties
└── pom.xml
```

## 🚀 Instalación y Configuración

### Prerrequisitos

- **JDK 17** o superior
- **MySQL 8.0** o superior
- **Maven 3.6+**
- IDE recomendado: IntelliJ IDEA, Eclipse o VS Code

### 1. Clonar el Repositorio

```bash
git clone https://github.com/tuusuario/loloyta-app.git
cd loloyta-app
```

### 2. Configurar Base de Datos

Crear la base de datos en MySQL:

```sql
CREATE DATABASE loloyta_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE loloyta_db;
```

### 3. Configurar `application.properties`

Editar `src/main/resources/application.properties`:

```properties
spring.application.name=loloytaApp
server.port=8020

# Configuración de MySQL
spring.datasource.url=jdbc:mysql://localhost:3306/loloyta_db?useTimezone=true&serverTimezone=UTC
spring.datasource.username=TU_USUARIO
spring.datasource.password=TU_PASSWORD
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA/Hibernate
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.database-platform=org.hibernate.dialect.MySQLDialect
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl

# Pool de conexiones
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
```

### 4. Estructura de las Tablas

Las tablas se crean automáticamente con `ddl-auto=update`, pero puedes crearlas manualmente:

```sql
-- Tabla Cocina (Pedidos)
CREATE TABLE cocina (
    cocina_id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(255),
    detalle VARCHAR(255),
    numero_mesa INT NOT NULL,
    precio DECIMAL(10,2) NOT NULL,
    activo TINYINT(1) DEFAULT 1
);

-- Tabla Venta
CREATE TABLE venta (
    venta_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100),
    detalle VARCHAR(255),
    precio DECIMAL(10,2),
    numero_mesa INT,
    fecha DATETIME DEFAULT CURRENT_TIMESTAMP,
    cocina_id INT,
    CONSTRAINT fk_venta_cocina FOREIGN KEY (cocina_id) 
        REFERENCES cocina(cocina_id)
);

-- Tabla Carta
CREATE TABLE carta (
    carta_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(255),
    tipo ENUM('PLATILLO', 'BEBIDA', 'POSTRE'),
    precio DOUBLE,
    disponible TINYINT(1) DEFAULT 1
);
```

### 5. Compilar y Ejecutar

```bash
# Con Maven
mvn clean install
mvn spring-boot:run

# O usando el JAR
java -jar target/loloytaApp-0.0.1-SNAPSHOT.jar
```

La aplicación estará disponible en: **http://localhost:8020**

## 👥 Usuarios por Defecto

| Usuario        | Email                | Password   | Rol                |
|----------------|----------------------|------------|---------------------|
| admin          | admin@correo.com     | 12345      | Ventas y Reportes   |
| mozo           | mozo@loloyta.com     | 10         | Toma de Pedidos     |
| jefe           | cocina@loloyta.com   | 11         | Gestión de Cocina   |
| administrador  | admin@admin.com      | adminpass  | Gestión de Carta    |

## 📱 Flujo de Uso

### 1. Configurar la Carta (Administrador)
1. Login con usuario `administrador`
2. Ir a `/admin/carta/carta`
3. Agregar platillos, bebidas y postres
4. Marcar items como disponibles

### 2. Tomar Pedidos (Mozo)
1. Login con usuario `mozo`
2. Seleccionar una mesa (1-6)
3. Agregar items desde la carta
4. Agregar detalles opcionales
5. Presionar "Enviar" → El pedido llega a cocina vía WebSocket
6. Al terminar, presionar "Confirmar Pago" → Se registra en ventas

### 3. Gestionar Cocina (Jefe)
1. Login con usuario `jefe`
2. Ver pedidos entrantes en tiempo real
3. Cambiar estado: "Recibido" → "Preparando"
4. Los cambios se sincronizan con las mesas

### 4. Ver Ventas (Admin)
1. Login con usuario `admin`
2. Ver historial completo
3. Filtrar por rango de fechas
4. Ver totales acumulados
5. Exportar reportes (JasperReports)

## 🔌 WebSocket - Endpoints

| Endpoint                  | Tipo        | Descripción                          |
|---------------------------|-------------|--------------------------------------|
| `/ws`                     | Conexión    | Endpoint principal WebSocket         |
| `/app/enviarPedido`       | Enviar      | Crear nuevo pedido                   |
| `/app/actualizarPedido`   | Enviar      | Actualizar pedido existente          |
| `/app/eliminarPedido`     | Enviar      | Eliminar pedido                      |
| `/topic/recibirPedido`    | Suscripción | Recibir pedidos nuevos/actualizados  |
| `/topic/estadoCambiado`   | Suscripción | Cambios de estado de pedidos         |
| `/topic/pedidoEliminado`  | Suscripción | Notificación de pedido eliminado     |

## 🎨 Características de UI/UX

- **Diseño glassmorphism** con efectos de blur
- **Navegación estilo Dynamic Island** inspirado en iOS
- **Responsive design** para móviles y tablets
- **Actualizaciones en tiempo real** sin recargar página
- **Animaciones suaves** en transiciones
- **Indicadores visuales** de estado de pedidos
- **Botones flotantes** para acciones principales

## 📊 Reportes (JasperReports)

El sistema incluye generación de reportes PDF con:
- Ventas por período
- Ventas por mesa
- Resumen de productos más vendidos
- Totales y estadísticas

## 🔒 Seguridad

> ⚠️ **Importante**: Este proyecto es una demostración educativa. Para producción:
> - Implementar Spring Security
> - Hash de contraseñas con BCrypt
> - Tokens JWT para autenticación
> - Validación de entrada robusta
> - HTTPS obligatorio
> - Variables de entorno para credenciales

## 🐛 Troubleshooting

### Error: "Field 'numero_mesa' doesn't have a default value"
- Verificar que `@Column(name = "numero_mesa")` esté en la entidad Cocina
- Verificar estrategia de naming en `application.properties`

### Error: WebSocket no conecta
- Verificar que el puerto 8020 no esté bloqueado
- Revisar configuración de `WebSocketConfig.java`
- Comprobar CORS si se accede desde otro dominio

### Error: "A '@JoinColumn' references a column named 'cocina_id'"
- Verificar que el ID en Cocina tenga `@Column(name = "cocina_id")`
- Verificar que la tabla Venta tenga la FK correcta

## 🚀 Despliegue en Producción

### Railway (Recomendado para demostración)

1. Crear cuenta en [Railway.app](https://railway.app)
2. Crear proyecto nuevo
3. Agregar servicio MySQL
4. Agregar servicio Spring Boot desde GitHub
5. Configurar variables de entorno:
   ```
   SPRING_DATASOURCE_URL=jdbc:mysql://[MYSQL_HOST]:3306/railway
   SPRING_DATASOURCE_USERNAME=[RAILWAY_USER]
   SPRING_DATASOURCE_PASSWORD=[RAILWAY_PASSWORD]
   ```

### Heroku

```bash
# Crear app
heroku create loloyta-app

# Agregar MySQL (ClearDB o JawsDB)
heroku addons:create cleardb:ignite

# Configurar variables
heroku config:set SPRING_DATASOURCE_URL=[URL_CLEARDB]

# Desplegar
git push heroku main
```

## 🤝 Contribuciones

Las contribuciones son bienvenidas. Por favor:
1. Fork el proyecto
2. Crea una rama para tu feature (`git checkout -b feature/AmazingFeature`)
3. Commit tus cambios (`git commit -m 'Add some AmazingFeature'`)
4. Push a la rama (`git push origin feature/AmazingFeature`)
5. Abre un Pull Request

## 📝 Roadmap

- [ ] Implementar Spring Security completo
- [ ] Sistema de reservas de mesas
- [ ] Dashboard con gráficos de ventas
- [ ] Notificaciones push para cocina
- [ ] Sistema de propinas
- [ ] Multi-idioma (i18n)
- [ ] App móvil nativa (React Native / Flutter)
- [ ] Integración con impresoras de tickets
- [ ] Sistema de inventario
- [ ] Modo offline con sincronización

## 📄 Licencia

Este proyecto es de código abierto y está disponible bajo la licencia MIT.

## 👨‍💻 Autor

Desarrollado como proyecto educativo para demostración de:
- Spring Boot + JPA
- WebSocket con STOMP
- Diseño de UI moderna
- Arquitectura MVC
- Integración frontend-backend en tiempo real

## 📞 Soporte

Para preguntas o soporte:
- Abrir un issue en GitHub
- Email: soporte@loloyta.com (demo)

---

⭐ Si este proyecto te fue útil, considera darle una estrella en GitHub!

**Happy Coding! 🍔🍕🍰**
