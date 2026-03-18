# Documentación de Arquitectura C4 - ERP Iglesias

## 1. Introducción
El proyecto **ERP Iglesias** es un sistema web diseñado para la gestión administrativa y operativa de congregaciones religiosas. La solución está construida sobre una arquitectura moderna y escalable que utiliza las siguientes tecnologías:
- **Frontend:** Angular (SPA)
- **Backend:** Spring Boot (Java)
- **Base de Datos:** PostgreSQL
- **Despliegue:** Docker y Docker Compose

---

## 2. Diagrama de Contexto (Nivel 1)
Este diagrama muestra el sistema ERP Iglesias en su entorno, identificando a los actores principales y su interacción con el sistema como una "caja negra".

```mermaid
C4Context
    title Diagrama de Contexto - ERP Iglesias
    
    Person(admin, "Administrador", "Gestiona usuarios, roles y configuración global")
    Person(staff, "Personal Administrativo", "Registra personas, ofrendas y pagos")
    Person(lider, "Líderes de Iglesia", "Consulta reportes y gestiona cursos/inscripciones")
    
    System(erp, "ERP Iglesias", "Sistema de Gestión para congregaciones religiosas")

    Rel(admin, erp, "Gestiona configuración y usuarios")
    Rel(staff, erp, "Gestiona finanzas y registros")
    Rel(lider, erp, "Consulta reportes y educación")
```

**Explicación:**
El sistema centraliza la información para tres tipos de usuarios. Los administradores controlan el acceso, el personal administrativo maneja el flujo financiero y de miembros, y los líderes supervisan las actividades educativas y reportes.

---

## 3. Diagrama de Contenedores (Nivel 2)
Desglosa el sistema en sus contenedores principales, mostrando las tecnologías utilizadas y los flujos de datos.

```mermaid
C4Container
    title Diagrama de Contenedores - ERP Iglesias
    
    Person(user, "Usuario", "Navegador Web")
    
    System_Boundary(c1, "ERP Iglesias") {
        Container(web, "Frontend", "Angular", "SPA que provee la interfaz de usuario")
        Container(api, "Backend", "Spring Boot", "REST API que maneja la lógica de negocio")
        ContainerDb(db, "Base de Datos", "PostgreSQL", "Almacena persistentemente la información")
    }

    Rel(user, web, "Usa", "HTTPS")
    Rel(web, api, "Consume", "JSON/REST")
    Rel(api, db, "Lee/Escribe", "JDBC")
```

**Explicación:**
El usuario interactúa con una aplicación de página única (Angular). Esta se comunica mediante peticiones REST con un servidor de aplicaciones (Spring Boot), el cual procesa la lógica y persiste los datos en un motor relacional (PostgreSQL).

---

## 4. Diagrama de Componentes del Backend (Nivel 3)
Muestra la organización interna del contenedor Backend (Spring Boot), detallando la interacción entre controladores, servicios y repositorios.

```mermaid
C4Component
    title Diagrama de Componentes - Backend (Spring Boot)
    
    Container(web, "Frontend", "Angular", "Interfaz de usuario")
    ContainerDb(db, "Base de Datos", "PostgreSQL", "Almacena datos")

    Container_Boundary(api, "Backend Application") {
        Component(security, "Security Filter", "JWT & SecurityConfig", "Filtra y autoriza peticiones")
        
        Component(authCtrl, "AuthController", "Spring RestController", "Maneja autenticación")
        Component(personCtrl, "PersonController", "Spring RestController", "CRUD de personas")
        Component(courseCtrl, "CourseController", "Spring RestController", "Gestión de cursos")
        Component(enrollCtrl, "EnrollmentController", "Spring RestController", "Proceso de inscripciones")
        Component(financeCtrl, "Finance Controllers", "Spring RestControllers", "Pagos y Ofrendas")
        Component(dashCtrl, "DashboardController", "Spring RestController", "Estadísticas")

        Component(personServ, "PersonService", "Spring Service", "Lógica de negocio de personas")
        Component(courseServ, "CourseService", "Spring Service", "Lógica de negocio de cursos")
        Component(dashFacade, "DashboardFacade", "Facade Pattern", "Simplifica acceso a datos del tablero")
        Component(authServ, "AuthUserDetailsService", "Spring Service", "Carga detalles de usuario")

        Component(repo, "Repositories", "Spring Data JPA", "Abstracción de acceso a datos")
    }

    %% Flujo de Peticiones y Seguridad
    Rel(web, security, "Peticiones", "HTTPS/JWT")
    Rel(security, authCtrl, "Enruta", "Interno")
    Rel(security, personCtrl, "Enruta", "Interno")
    Rel(security, courseCtrl, "Enruta", "Interno")
    Rel(security, enrollCtrl, "Enruta", "Interno")
    Rel(security, financeCtrl, "Enruta", "Interno")
    Rel(security, dashCtrl, "Enruta", "Interno")

    %% Relación Controladores -> Servicios/Fachadas
    Rel(authCtrl, authServ, "Usa")
    Rel(personCtrl, personServ, "Usa")
    Rel(courseCtrl, courseServ, "Usa")
    Rel(dashCtrl, dashFacade, "Usa")
    
    %% Relación Directa a Repositorios (Casos simples)
    Rel(enrollCtrl, repo, "Usa CRUD")
    Rel(financeCtrl, repo, "Usa CRUD")

    %% Relación Servicios -> Repositorios
    Rel(personServ, repo, "Usa")
    Rel(courseServ, repo, "Usa")
    Rel(dashFacade, repo, "Usa")
    Rel(authServ, repo, "Usa")

    %% Persistencia
    Rel(repo, db, "SQL", "JDBC")
```

**Explicación:**
El backend sigue un patrón de capas. La seguridad (JWT) intercepta las peticiones antes de llegar a los **Controladores**. Algunos controladores utilizan **Servicios** para lógica compleja (como `PersonService`), mientras que otros interactúan directamente con los **Repositorios** para operaciones CRUD simples. Los repositorios finalmente se comunican con la base de datos PostgreSQL.

---

## 5. Conclusión
La arquitectura del ERP Iglesias sigue los principios de separación de responsabilidades a través del modelo C4. El uso de contenedores aislados mediante Docker facilita su despliegue y mantenimiento, mientras que la estructura interna del backend asegura un desarrollo organizado y seguro mediante Spring Security y JPA.
