# Inicio Rápido - Proyecto NestJS con Cursor Rules

Esta guía te ayudará a iniciar un nuevo proyecto NestJS aplicando las reglas de Cursor desde el principio.

## Prerrequisitos

- Node.js (v20 o superior)
- pnpm, npm o yarn
- Docker y Docker Compose
- Cursor IDE con las reglas instaladas

## Paso 1: Crear Nuevo Proyecto NestJS

```bash
# Instalar NestJS CLI si no lo tienes
npm i -g @nestjs/cli

# Crear nuevo proyecto
nest new nombre-proyecto

# Entrar al directorio
cd nombre-proyecto
```

## Paso 2: Copiar Reglas de Cursor

```bash
# Copiar .cursorrules a la raíz del proyecto
cp /ruta/a/.cursorrules .
```

## Paso 3: Instalar Dependencias Necesarias

```bash
# Dependencias principales
npm install @nestjs/typeorm typeorm pg
npm install @nestjs/config
npm install @nestjs/swagger
npm install @nestjs/jwt @nestjs/passport passport passport-jwt
npm install class-validator class-transformer
npm install bcrypt

# Tipos para TypeScript
npm install -D @types/passport-jwt
npm install -D @types/bcrypt
```

## Paso 4: Configurar Variables de Entorno

Crea un archivo `.env` en la raíz:

```env
# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=nombre_db
DB_USERNAME=postgres
DB_PASSWORD=tu_password_seguro

# JWT
JWT_SECRET=tu_secreto_muy_seguro_y_largo_de_al_menos_32_caracteres

# App
PORT=3000
```

## Paso 5: Configurar Docker Compose

Crea `docker-compose.yml`:

```yaml
version: '3'

services:
  db:
    image: postgres:14.3
    restart: always
    ports:
      - "5432:5432"
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    container_name: postgres_nombre_proyecto
    volumes:
      - ./postgres:/var/lib/postgresql/data
```

Agregar `postgres/` a `.gitignore`:
```bash
echo "postgres/" >> .gitignore
```

## Paso 6: Configurar main.ts

Ahora puedes pedirle a Cursor:

```
Configura main.ts siguiendo las reglas establecidas:
- ValidationPipe global con whitelist
- Swagger con autenticación Bearer
- Prefijo global 'api'
```

## Paso 7: Configurar app.module.ts

Prompt para Cursor:

```
Configura app.module.ts con:
- ConfigModule para variables de entorno
- TypeORM conectado a PostgreSQL
- Import de módulos futuros
```

## Paso 8: Crear Módulo de Autenticación

Prompt para Cursor:

```
Crea un módulo completo de autenticación con:
- Entidad User (email, fullName, password, roles, isActive)
- JWT strategy con Passport
- Endpoints de registro y login
- Guard de autenticación
- Decorador @Auth para proteger rutas
- Enum ValidRoles con roles: admin, teacher, student
```

## Paso 9: Iniciar Docker y Base de Datos

```bash
# Iniciar PostgreSQL
docker-compose up -d

# Verificar que esté corriendo
docker ps
```

## Paso 10: Crear Primer Módulo de Negocio

Prompt para Cursor:

```
Crea un módulo de [tu dominio, ej: productos, usuarios, cursos] con:
- CRUD completo
- Validaciones apropiadas
- Documentación Swagger
- Paginación en listados
- Búsqueda por diferentes criterios
- Endpoints protegidos con autenticación
```

## Paso 11: Crear Seed (Opcional)

Prompt para Cursor:

```
Crea un módulo seed para insertar datos de prueba de:
- Usuarios con diferentes roles
- [Entidades de tu dominio]
Incluye un endpoint GET /api/seed para ejecutarlo
```

## Paso 12: Ejecutar y Probar

```bash
# Modo desarrollo
npm run start:dev

# La aplicación estará en http://localhost:3000
# Swagger disponible en http://localhost:3000/api
```

## Estructura Final del Proyecto

```
nombre-proyecto/
├── .cursorrules                 # ✅ Reglas de Cursor
├── .env                         # ✅ Variables de entorno
├── .gitignore
├── docker-compose.yml           # ✅ PostgreSQL
├── package.json
├── nest-cli.json
├── tsconfig.json
├── src/
│   ├── main.ts                  # ✅ Configurado
│   ├── app.module.ts            # ✅ Configurado
│   ├── auth/                    # ✅ Módulo de autenticación
│   │   ├── decorators/
│   │   │   └── auth.decorator.ts
│   │   ├── dto/
│   │   │   ├── login.dto.ts
│   │   │   └── register.dto.ts
│   │   ├── entities/
│   │   │   └── user.entity.ts
│   │   ├── enums/
│   │   │   └── roles.enum.ts
│   │   ├── guards/
│   │   │   └── user-role.guard.ts
│   │   ├── interfaces/
│   │   │   └── jwt-payload.interface.ts
│   │   ├── strategies/
│   │   │   └── jwt.strategy.ts
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   └── auth.module.ts
│   ├── seed/                    # ✅ Módulo de seeds (opcional)
│   │   ├── seed.controller.ts
│   │   ├── seed.service.ts
│   │   └── seed.module.ts
│   └── [tu-modulo]/            # ✅ Módulos de negocio
│       ├── dto/
│       ├── entities/
│       ├── [modulo].controller.ts
│       ├── [modulo].service.ts
│       └── [modulo].module.ts
└── postgres/                    # Datos de PostgreSQL (git ignored)
```

## Comandos Útiles Durante el Desarrollo

```bash
# Generar nuevo recurso
nest g resource nombre --no-spec

# Generar solo servicio
nest g service modulo/nombre --no-spec

# Generar solo controlador
nest g controller modulo/nombre --no-spec

# Generar guard
nest g guard auth/guards/nombre --no-spec

# Generar decorator
nest g decorator auth/decorators/nombre --no-spec

# Ver logs de Docker
docker-compose logs -f

# Reiniciar base de datos
docker-compose down
docker-compose up -d
```

## Testing con Swagger

1. Ve a http://localhost:3000/api
2. Prueba el endpoint de registro: `POST /api/auth/register`
3. Copia el token JWT recibido
4. Click en "Authorize" en la esquina superior derecha
5. Pega el token en formato: `Bearer tu_token_aqui`
6. Ahora puedes probar endpoints protegidos

## Prompts Útiles para Cursor

### Agregar nuevo campo a entidad
```
Agrega el campo "telefono" a la entidad User:
- Debe ser opcional
- Formato de validación para teléfonos
- Documentación Swagger
```

### Crear relación entre entidades
```
Crea una relación OneToMany entre User y Posts.
Un usuario puede tener muchos posts.
Incluye cascade y eager loading.
```

### Implementar filtros
```
Agrega filtros al endpoint de listar usuarios:
- Por rol
- Por estado (activo/inactivo)
- Búsqueda por nombre o email
```

### Agregar paginación personalizada
```
Mejora la paginación para incluir:
- Total de páginas
- Página actual
- Siguiente/anterior
```

## Solución de Problemas Comunes

### Error de conexión a PostgreSQL
```bash
# Verificar que el contenedor esté corriendo
docker ps

# Ver logs del contenedor
docker logs postgres_nombre_proyecto

# Reiniciar contenedor
docker-compose restart
```

### Error de validación en DTOs
```
Asegúrate de tener instalado:
npm install class-validator class-transformer

Y que main.ts tenga el ValidationPipe configurado
```

### JWT no funciona
```
Verifica:
1. JWT_SECRET en .env
2. JwtModule configurado en AuthModule
3. JwtStrategy implementado correctamente
```

## Siguientes Pasos

1. ✅ Implementar más módulos de negocio
2. ✅ Agregar testing (unit y e2e)
3. ✅ Implementar logging avanzado
4. ✅ Agregar cache con Redis
5. ✅ Implementar file uploads
6. ✅ Agregar rate limiting
7. ✅ Configurar CI/CD
8. ✅ Preparar para producción (migrations, etc.)

## Resources

- [NestJS Docs](https://docs.nestjs.com)
- [TypeORM Docs](https://typeorm.io)
- [Swagger/OpenAPI](https://swagger.io/specification/)
- [Passport.js](http://www.passportjs.org/)

---

**¡Listo!** Ahora tienes un proyecto NestJS completamente configurado y listo para desarrollar con las reglas de Cursor que te ayudarán a mantener código consistente y de calidad.

## Checklist de Inicio ✓

- [ ] Proyecto NestJS creado
- [ ] Dependencias instaladas
- [ ] .cursorrules copiado
- [ ] Variables de entorno configuradas
- [ ] Docker Compose configurado
- [ ] PostgreSQL corriendo
- [ ] main.ts configurado
- [ ] app.module.ts configurado
- [ ] Módulo de autenticación creado
- [ ] Primer módulo de negocio creado
- [ ] Swagger accesible
- [ ] Seeds creados (opcional)
- [ ] Todo funcionando correctamente

**¡A desarrollar! 🚀**
