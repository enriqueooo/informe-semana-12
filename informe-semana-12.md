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

### Paso 2: Crear un microservicio base (ejemplo auth-service)

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

```
