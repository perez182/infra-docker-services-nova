# 🚀 Infra Nova Services - Microservices Orchestration

Este repositorio contiene la configuración de **Docker Compose** centralizada para orquestar el ecosistema de microservicios de la plataforma. El objetivo es permitir un entorno de desarrollo local que replique la comunicación real en red, eliminando la necesidad de levantar servicios de forma manual o aislada.

## 📁 Estructura de Carpetas

Para que el orquestador funcione correctamente, los repositorios deben estar agrupados de la siguiente manera:

```text
workspace-root/
│
├── infra-nova-services-docker/    # (Este repositorio)
│   └── docker-compose.yml
│
├── customer-service/              # Microservicio de Clientes
├── order-service/                 # Microservicio de Órdenes
└── order-management-service/      # Orquestador 
└── gateway-service/               # Api Gateway


### ¿Por qué esta estructura?
*   **Contexto de Construcción**: El \`docker-compose.yml\` utiliza rutas relativas (\`../\`) para acceder a los \`Dockerfiles\` de cada servicio y construir las imágenes en tiempo real.
*   **Independencia de Código**: Permite que cada microservicio mantenga su propio ciclo de vida y repositorio de Git, mientras que este proyecto actúa como el "pegamento" de infraestructura.

---

## 🛠️ Configuración de Perfiles (Spring Profiles)

El sistema utiliza el perfil de Spring \`docker\`. Esto es fundamental para que el **Order Management Service** (el orquestador) deje de buscar los servicios en \`localhost\` y empiece a buscarlos dentro de la red interna de Docker.

*   **Archivo Local**: \`application.properties\` (apunta a \`localhost\` para desarrollo en IDE).
*   **Archivo Docker**: \`application-docker.properties\` (apunta a nombres de contenedores como \`http://customer-service:8082\`).

---

## 🚀 Cómo Correr el Proyecto

Ya **no es necesario** (y de hecho, causará errores de puertos) ejecutar las instancias de los servicios por separado en tu IDE o mediante comandos de Docker individuales. El Super Docker Compose se encarga de todo.

### Paso 1: Limpieza (Solo si tenías instancias previas)
Detén cualquier contenedor que esté usando los puertos 8082, 8083 o 8084:
\`\`\`bash
docker stop \$(docker ps -q)
\`\`\`

### Paso 2: Levantar la Infraestructura
Desde la carpeta \`infra-nova-services-docker\`, ejecuta:

\`\`\`bash
docker-compose up -d --build
\`\`\`

**Parámetros clave:**
*   \`--build\`: Fuerza la recompilación de los archivos JAR y asegura que el archivo \`application-docker.properties\` se incluya en la imagen.
*   \`-d\`: Corre en segundo plano (detached mode).

---

## 🔄 Flujo de Comunicación en Red

Al utilizar este método, Docker crea una red virtual donde:
1.  **Resolución por Nombre**: Los servicios no necesitan conocer IPs. Se llaman entre si por el nombre definido en el YAML (ej. \`http://order-service\`).
2.  **Aislamiento**: Los servicios están protegidos y solo los puertos definidos en \`ports\` están expuestos hacia tu máquina física.
3.  **Circuit Breaker**: Al estar en la misma red, el **Resilience4j** detectará correctamente si un servicio cae, activando el fallback de manera precisa.

## 🛑 Notas Importantes
*   **No ejecutar duplicados**: Si intentas correr el servicio desde IntelliJ mientras Docker está arriba, recibirás un error de \`Port already allocated\`.
*   **Logs**: Para monitorear la comunicación y ver la activación del perfil docker, usa:
    \`docker logs -f order-management-service\`

---

¡Con esto tu entorno de microservicios está listo para escalar! ☕
EOF