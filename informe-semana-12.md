# Práctica: Contenerización de Arquitectura de Microservicios para Sistema de Cuestionarios con Docker

## 1. Título  
**Contenerización de microservicios para sistema de cuestionarios con API Gateway y servicio descubridor usando Docker**

## 2. Tiempo de duración  
**120 minutos**

## 3. Fundamentos  

Esta práctica busca aplicar una arquitectura de microservicios a un sistema de cuestionarios, donde cada microservicio gestiona una responsabilidad clara: usuarios y roles, cuestionarios, preguntas, respuestas y catálogos. Se implementan servicios independientes, desplegables y escalables, registrados en un servicio descubridor (Eureka) y expuestos mediante un API Gateway. Se usan contenedores Docker para cada microservicio, el gateway y el descubridor, y `docker-compose` para orquestar la comunicación dentro de una red compartida. Se utiliza PostgreSQL como base de datos en cada servicio.

## 4. Conocimientos previos

- Fundamentos de Docker y `docker-compose`.
- Arquitectura de microservicios.
- Conocimiento básico de APIs REST (preferible NestJS o Spring Boot).
- Manejo básico de PostgreSQL.
- Conceptos de API Gateway y servicio descubridor (Eureka, Consul).
- Manejo de terminal y comandos Docker.

## 5. Objetivos a alcanzar

- Crear microservicios independientes para cada módulo (usuario, cuestionario, pregunta, respuesta, catálogo).
- Implementar un servicio descubridor para registro dinámico de microservicios.
- Implementar un API Gateway para unificar puntos de acceso.
- Contenerizar cada microservicio, el gateway y el descubridor con Docker.
- Orquestar los contenedores con `docker-compose` para que se comuniquen dentro de una red Docker.

## 6. Equipo necesario

- Docker y `docker-compose` instalados.
- Node.js con NestJS (o Java con Spring Boot).
- Editor de código (Visual Studio Code recomendado).
- Navegador web para probar endpoints.
- PostgreSQL instalado o contenedor para base de datos.
- Conexión a internet para descargar imágenes base.

## 7. Material de apoyo

- [Docker Docs](https://docs.docker.com/)
- [NestJS Docs](https://docs.nestjs.com/)
- [Spring Cloud Netflix (Eureka)](https://spring.io/projects/spring-cloud-netflix)
- [PostgreSQL Docs](https://www.postgresql.org/docs/)
- [API Gateway Concepts](https://microservices.io/patterns/apigateway.html)

## 8. Procedimiento

### Paso 1: Definir microservicios y base de datos

- `auth-service`: maneja usuarios, roles, estados.
- `cuestionario-service`: maneja cuestionarios y asignaciones.
- `pregunta-service`: maneja preguntas y tipos.
- `respuesta-service`: maneja respuestas y respuestas de usuario.
- `catalogo-service`: maneja datos comunes (estado, tipo acceso, tipo pregunta).
- `discovery-service`: servicio Eureka para descubrimiento.
- `api-gateway`: API Gateway para enrutamiento de peticiones.

Cada servicio tendrá su propia base de datos PostgreSQL o esquema separado.

---

### Paso 2: Crear un microservicio base 

**Archivo principal:** `auth-service/src/main.ts` (NestJS)

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.setGlobalPrefix('api');
  await app.listen(3001);
}
bootstrap();

# Dockerfile para auth-service
```
```dockerfile  
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
EXPOSE 3001
CMD ["node", "dist/main.js"]


# Paso 3: Configurar servicio descubridor 

Crear proyecto Eureka Server que escuche en el puerto **8761**.

Configurar para que otros servicios se registren en:  
`http://discovery-service:8761/eureka/`

---

## Dockerfile discovery-service

```dockerfile
FROM openjdk:17-jdk-alpine
VOLUME /tmp
COPY target/discovery-service.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
EXPOSE 8761

```
# Paso 4: Configurar API Gateway

Configurar rutas a servicios con balanceo de carga y descubrimiento.

Ejemplo de configuración en `application.yml` (Spring Cloud Gateway):

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
# Dockerfile para api-gateway

```dockerfile
FROM openjdk:17-jdk-alpine
COPY target/api-gateway.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
EXPOSE 8080

```
# Paso 5: Crear archivo docker-compose.yml

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

  # Se repetiría para los demás servicios: cuestionario-service, pregunta-service, respuesta-service, catalogo-service

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
# 10. Bibliografía

- [Docker Docs](https://docs.docker.com/)
- [NestJS Docs](https://docs.nestjs.com/)
- [Spring Cloud Netflix Eureka](https://spring.io/projects/spring-cloud-netflix)
- [API Gateway Pattern](https://microservices.io/patterns/apigateway.html)
- [PostgreSQL Docs](https://www.postgresql.org/docs/)

```
```
# 11. Link del Audio
[PostgreSQL Docs](https://www.postgresql.org/docs/)
```
