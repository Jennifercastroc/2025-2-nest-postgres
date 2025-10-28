# Reglas de Cursor para Proyectos NestJS + PostgreSQL


## ¿Qué son las Reglas de Cursor?

Las reglas de Cursor (`.cursorrules`) son un archivo de configuración que le indica a Cursor AI cómo debe generar y estructurar el código en tus proyectos. Actúa como un conjunto de "mejores prácticas" que el asistente de IA seguirá automáticamente.

## Instalación

### Opción 1: Archivo Global (Recomendado)

Para que estas reglas se apliquen a todos tus proyectos NestJS:

1. Copia el archivo `.cursorrules` a tu directorio home:
   ```bash
   # En Linux/Mac
   cp .cursorrules ~/.cursorrules
   
   # En Windows (PowerShell)
   Copy-Item .cursorrules $HOME\.cursorrules
   ```

### Opción 2: Por Proyecto

Para aplicar estas reglas solo a un proyecto específico:

1. Copia el archivo `.cursorrules` a la raíz de tu proyecto NestJS
2. El archivo debe estar al mismo nivel que `package.json`

```
mi-proyecto/
├── .cursorrules          ← Aquí
├── package.json
├── src/
└── ...
```

## ¿Qué incluyen estas reglas?

### ✅ Estructura de Proyecto
- Organización de carpetas por módulos
- Separación de concerns (dto, entities, guards, etc.)
- Estructura estandarizada para auth, seed, y otros módulos comunes

### ✅ Convenciones de Código
- **Entidades TypeORM**: Uso de decoradores, hooks (@BeforeInsert/@BeforeUpdate), y relaciones
- **DTOs**: Validaciones con class-validator, documentación Swagger
- **Controladores**: Endpoints RESTful, decoradores de autenticación, documentación
- **Servicios**: Manejo de errores, CRUD completo, búsquedas flexibles

### ✅ Autenticación y Autorización
- Implementación de JWT con Passport
- Sistema de roles y guards personalizados
- Decoradores personalizados (@Auth)

### ✅ Configuración
- Setup de main.ts con ValidationPipe y Swagger
- Configuración de TypeORM con PostgreSQL
- Variables de entorno y Docker Compose

### ✅ Mejores Prácticas
- Nombrado consistente
- Manejo de errores estandarizado
- Documentación con Swagger
- Seguridad y performance

## Uso con Cursor

Una vez instalado el archivo `.cursorrules`, simplemente usa Cursor normalmente. Por ejemplo:

### Ejemplo 1: Crear un nuevo módulo
```
Prompt: "Crea un módulo completo de productos con CRUD"
```
Cursor generará automáticamente:
- La entidad Product con decoradores apropiados
- DTOs con validaciones
- Servicio con manejo de errores
- Controlador con documentación Swagger
- Módulo configurado correctamente

### Ejemplo 2: Agregar autenticación
```
Prompt: "Protege el endpoint de actualización para que solo admins puedan acceder"
```
Cursor agregará automáticamente:
- El decorador @Auth(ValidRoles.admin)
- @ApiBearerAuth('JWT-auth')
- Imports necesarios

### Ejemplo 3: Crear relaciones
```
Prompt: "Agrega una relación OneToMany entre Usuario y Productos"
```
Cursor seguirá las convenciones establecidas para relaciones TypeORM.

## Personalización

Puedes editar el archivo `.cursorrules` para:
- Agregar reglas específicas de tu proyecto
- Modificar convenciones de nombrado
- Agregar más patrones de código
- Incluir librerías adicionales

## Comandos útiles incluidos

El archivo incluye referencia a comandos NestJS útiles:
```bash
# Generar recurso completo
nest g resource [nombre] --no-spec

# Generar servicio
nest g service [nombre] --no-spec

# Generar controlador
nest g controller [nombre] --no-spec
```

## Stack Tecnológico Cubierto

- ✅ NestJS 11.x
- ✅ TypeORM 0.3.x
- ✅ PostgreSQL 14.x
- ✅ JWT Authentication
- ✅ Passport
- ✅ Swagger/OpenAPI
- ✅ class-validator
- ✅ class-transformer
- ✅ Docker Compose

## Checklist de Nuevo Proyecto

Cuando inicies un nuevo proyecto, estas reglas te guiarán para:

1. ✅ Configurar TypeORM con PostgreSQL
2. ✅ Implementar sistema de autenticación
3. ✅ Crear módulos siguiendo la estructura estándar
4. ✅ Configurar Swagger para documentación
5. ✅ Implementar validaciones en DTOs
6. ✅ Manejar errores consistentemente
7. ✅ Configurar Docker para desarrollo

## Soporte

Este archivo fue generado analizando el proyecto base:
https://github.com/profe-gus/2025-2-nest-postgres.git

Para más información sobre:
- **NestJS**: https://docs.nestjs.com
- **TypeORM**: https://typeorm.io
- **Cursor Rules**: https://cursor.sh/docs

## Contribuir

Si encuentras mejoras o quieres agregar nuevos patrones:
1. Edita el archivo `.cursorrules`
2. Agrega comentarios explicativos
3. Prueba los cambios con Cursor
4. Comparte con el equipo

---

**Nota:** Recuerda que las reglas son guías, no restricciones absolutas. Cursor las seguirá por defecto, pero siempre puedes pedir explícitamente algo diferente en tus prompts.
