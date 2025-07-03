
# Práctica: Contenerización de Arquitectura de Microservicios para Examen de Ubicación de Inglés

## 1. Título  
**Contenerización de microservicios para una aplicación de examen de ubicación de inglés con API Gateway y servicio descubridor usando Docker**

## 2. Tiempo de duración  
**120 minutos**

## 3. Fundamentos  

Esta práctica aplica los principios de arquitectura de microservicios a una aplicación práctica de examen de ubicación de inglés. Cada microservicio gestiona una responsabilidad específica, como la administración de usuarios, la creación de cuestionarios, la gestión de preguntas y respuestas, y los catálogos del sistema. Para garantizar una comunicación eficiente entre servicios, se incorpora un servicio descubridor (Eureka) y un API Gateway que enruta las peticiones del cliente. Todos los servicios son contenerizados con Docker y orquestados mediante Docker Compose, asegurando independencia, escalabilidad y facilidad de despliegue. La base de datos utilizada es PostgreSQL.

## 4. Conocimientos previos

- Fundamentos de Docker y `docker-compose`.
- Arquitectura de microservicios.
- Conocimiento básico de APIs REST (preferible NestJS o Spring Boot).
- Manejo básico de PostgreSQL.
- Conceptos de API Gateway y servicio descubridor (Eureka, Consul).
- Comandos básicos en terminal.

## 5. Objetivos a alcanzar

- Implementar microservicios independientes para el sistema de examen.
- Usar un servicio descubridor para registrar dinámicamente los microservicios.
- Implementar un API Gateway como punto de entrada único.
- Contenerizar todos los servicios con Docker.
- Orquestar y comunicar los servicios mediante Docker Compose.

## 6. Equipo necesario

- Docker y `docker-compose`.
- Node.js y NestJS o Java y Spring Boot.
- Editor de código (Visual Studio Code).
- PostgreSQL (como contenedor).
- Navegador web moderno.
- Conexión a internet.

## 7. Material de apoyo

- [Docker Docs](https://docs.docker.com/)
- [NestJS Docs](https://docs.nestjs.com/)
- [Spring Cloud Netflix (Eureka)](https://spring.io/projects/spring-cloud-netflix)
- [PostgreSQL Docs](https://www.postgresql.org/docs/)
- [API Gateway Concepts](https://microservices.io/patterns/apigateway.html)

## 8. Procedimiento

### Paso 1: Definir microservicios y sus dominios

- `auth-service`: usuarios, roles, estados.
- `cuestionario-service`: creación y asignación de exámenes.
- `pregunta-service`: preguntas y tipos de pregunta.
- `respuesta-service`: respuestas disponibles y respuestas dadas por los estudiantes.
- `catalogo-service`: acceso a catálogos como tipo de pregunta, tipo de acceso y estado.
- `discovery-service`: Eureka como servicio de descubrimiento.
- `api-gateway`: acceso unificado mediante Spring Cloud Gateway.

Cada microservicio tiene su propia base de datos PostgreSQL.

### Paso 2: Crear microservicio base (auth-service con NestJS)

**Archivo principal:** `auth-service/src/main.ts`

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.setGlobalPrefix('api/auth');
  await app.listen(3001);
}
bootstrap();
```

**Dockerfile para auth-service**

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
EXPOSE 3001
CMD ["node", "dist/main.js"]
```

### Paso 3: Configurar el servicio descubridor (Eureka Server)

**Dockerfile para discovery-service (Spring Boot)**

```dockerfile
FROM openjdk:17-jdk-alpine
VOLUME /tmp
COPY target/discovery-service.jar app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
EXPOSE 8761
```

### Paso 4: Configurar el API Gateway

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: auth-service
          uri: lb://AUTH-SERVICE
          predicates:
            - Path=/api/auth/**
        - id: cuestionario-service
          uri: lb://CUESTIONARIO-SERVICE
          predicates:
            - Path=/api/cuestionarios/**
```

**Dockerfile del API Gateway**

```dockerfile
FROM openjdk:17-jdk-alpine
COPY target/api-gateway.jar app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
EXPOSE 8080
```

### Paso 5: Crear archivo `docker-compose.yml`

```yaml
version: '3.8'

services:
  discovery-service:
    build: ./discovery-service
    ports:
      - "8761:8761"
    networks:
      - microservices-net

  api-gateway:
    build: ./api-gateway
    ports:
      - "8080:8080"
    depends_on:
      - discovery-service
    networks:
      - microservices-net

  auth-service:
    build: ./auth-service
    ports:
      - "3001:3001"
    depends_on:
      - discovery-service
    environment:
      - EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://discovery-service:8761/eureka/
      - DB_HOST=auth-db
      - DB_PORT=5432
    networks:
      - microservices-net

  auth-db:
    image: postgres:15
    environment:
      POSTGRES_DB: authdb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    volumes:
      - auth_db_data:/var/lib/postgresql/data
    networks:
      - microservices-net

networks:
  microservices-net:

volumes:
  auth_db_data:
```

## 9. Resultados esperados

- Eureka muestra el registro dinámico de los microservicios.
- El API Gateway enruta las solicitudes correctamente.
- Cada microservicio puede ser escalado y actualizado individualmente.
- Los contenedores se comunican de forma eficiente.

## 10. Bibliografía

- [Docker Docs](https://docs.docker.com/)
- [NestJS Docs](https://docs.nestjs.com/)
- [Spring Cloud Netflix Eureka](https://spring.io/projects/spring-cloud-netflix)
- [API Gateway Pattern](https://microservices.io/patterns/apigateway.html)
- [PostgreSQL Docs](https://www.postgresql.org/docs/)

## 11. Link del Audio

🔊 [Escuchar audio explicativo] (https://drive.google.com/file/d/1Krk1QxyIAD9s_j0xZWokUThbI9qBjfBE/view?usp=sharing)
