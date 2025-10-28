# Mejores Prácticas - Proyecto NestJS + PostgreSQL

Esta guía complementa las reglas de Cursor con mejores prácticas específicas para el desarrollo del proyecto.

## Índice
1. [Seguridad](#seguridad)
2. [Performance](#performance)
3. [Manejo de Errores](#manejo-de-errores)
4. [Base de Datos](#base-de-datos)
5. [Testing](#testing)
6. [Documentación](#documentación)
7. [Git y Versionado](#git-y-versionado)

---

## Seguridad

### ✅ Passwords

**SIEMPRE:**
```typescript
// Hash passwords antes de guardar
import * as bcrypt from 'bcrypt';

@BeforeInsert()
@BeforeUpdate()
async hashPassword() {
    if (this.password) {
        this.password = await bcrypt.hash(this.password, 10);
    }
}
```

**NUNCA:**
```typescript
// ❌ Guardar passwords en texto plano
@Column('text')
password: string; // Sin hash

// ❌ Retornar passwords en responses
return user; // Incluye password
```

### ✅ JWT Tokens

**SIEMPRE:**
```typescript
// Tokens con expiración
JwtModule.register({
    secret: process.env.JWT_SECRET,
    signOptions: { 
        expiresIn: '2h' // O más corto según necesidad
    }
})

// Validar tokens en cada request protegido
@UseGuards(AuthGuard())
```

**NUNCA:**
```typescript
// ❌ Tokens sin expiración
signOptions: {} // Sin expiresIn

// ❌ Secretos hardcodeados
secret: 'mi-secreto-123'
```

### ✅ Validación de Inputs

**SIEMPRE:**
```typescript
// Validar TODOS los inputs
@IsString()
@IsNotEmpty()
@MinLength(3)
@MaxLength(100)
nombre: string;

// Whitelist y forbidNonWhitelisted
app.useGlobalPipes(
    new ValidationPipe({
        whitelist: true,
        forbidNonWhitelisted: true,
        transform: true
    })
);
```

### ✅ Prevención de SQL Injection

**SIEMPRE:**
```typescript
// Usar query builders con parámetros
queryBuilder
    .where('name = :name', { name })
    .andWhere('price > :price', { price });
```

**NUNCA:**
```typescript
// ❌ Concatenar strings en queries
queryBuilder
    .where(`name = '${name}'`) // VULNERABLE
```

### ✅ CORS

```typescript
// Configurar CORS apropiadamente
app.enableCors({
    origin: process.env.ALLOWED_ORIGINS?.split(',') || '*',
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE']
});
```

### ✅ Rate Limiting

```typescript
import { ThrottlerModule } from '@nestjs/throttler';

// Limitar requests por IP
ThrottlerModule.forRoot({
    ttl: 60,
    limit: 10,
})
```

---

## Performance

### ✅ Paginación

**SIEMPRE implementar paginación:**
```typescript
async findAll(paginationDto: PaginationDto) {
    const { limit = 10, offset = 0 } = paginationDto;
    
    return this.repository.find({
        take: limit,
        skip: offset,
        order: { createdAt: 'DESC' }
    });
}
```

### ✅ Eager Loading vs Lazy Loading

**Usar eager loading con moderación:**
```typescript
// ✅ Bien: Solo cuando SIEMPRE necesitas la relación
@OneToMany(() => Review, review => review.product, { 
    eager: true // Solo si SIEMPRE necesitas reviews
})
reviews: Review[];

// ✅ Mejor: Cargar selectivamente
async findOneWithReviews(id: string) {
    return this.repository.findOne({
        where: { id },
        relations: ['reviews'] // Solo cuando se necesita
    });
}
```

### ✅ Select Específico

```typescript
// ✅ Seleccionar solo campos necesarios
return this.repository.find({
    select: ['id', 'name', 'price'], // No cargar todo
    take: limit,
    skip: offset
});
```

### ✅ Índices

```typescript
// Agregar índices en campos de búsqueda frecuente
@Entity()
@Index(['email']) // Búsquedas por email
@Index(['createdAt']) // Ordenamiento por fecha
export class User {
    @Column({ unique: true })
    email: string;
}
```

### ✅ Caché

```typescript
import { CacheModule } from '@nestjs/cache-manager';

// Cachear queries costosas
@Injectable()
export class ProductService {
    @Inject(CACHE_MANAGER)
    private cacheManager: Cache;

    async findPopular() {
        const cached = await this.cacheManager.get('popular-products');
        if (cached) return cached;

        const products = await this.repository.find({
            order: { views: 'DESC' },
            take: 10
        });

        await this.cacheManager.set('popular-products', products, 3600);
        return products;
    }
}
```

---

## Manejo de Errores

### ✅ Errores Específicos

```typescript
// ✅ Usar excepciones específicas de NestJS
throw new NotFoundException(`Product with id ${id} not found`);
throw new BadRequestException('Invalid input data');
throw new UnauthorizedException('Invalid credentials');
throw new ForbiddenException('Access denied');
throw new ConflictException('Email already exists');
```

### ✅ Global Exception Filter

```typescript
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
    catch(exception: unknown, host: ArgumentsHost) {
        const ctx = host.switchToHttp();
        const response = ctx.getResponse();
        const request = ctx.getRequest();

        const status = exception instanceof HttpException
            ? exception.getStatus()
            : 500;

        const message = exception instanceof HttpException
            ? exception.message
            : 'Internal server error';

        response.status(status).json({
            statusCode: status,
            timestamp: new Date().toISOString(),
            path: request.url,
            message,
        });
    }
}
```

### ✅ Logging de Errores

```typescript
private handleDBExceptions(error: any) {
    if (error.code === '23505') {
        throw new BadRequestException(error.detail);
    }
    
    // Log completo para debugging
    console.error('Database error:', {
        code: error.code,
        detail: error.detail,
        stack: error.stack
    });
    
    throw new InternalServerErrorException('Unexpected error');
}
```

---

## Base de Datos

### ✅ Migraciones en Producción

**NUNCA usar synchronize: true en producción:**
```typescript
// development
TypeOrmModule.forRoot({
    synchronize: true // Solo en desarrollo
})

// production - usar migraciones
TypeOrmModule.forRoot({
    synchronize: false,
    migrationsRun: true,
    migrations: ['dist/migrations/*.js']
})
```

### ✅ Transacciones

```typescript
async transferStock(fromId: string, toId: string, amount: number) {
    const queryRunner = this.dataSource.createQueryRunner();
    
    await queryRunner.connect();
    await queryRunner.startTransaction();
    
    try {
        const from = await queryRunner.manager.findOne(Product, { 
            where: { id: fromId } 
        });
        const to = await queryRunner.manager.findOne(Product, { 
            where: { id: toId } 
        });

        from.stock -= amount;
        to.stock += amount;

        await queryRunner.manager.save(from);
        await queryRunner.manager.save(to);

        await queryRunner.commitTransaction();
    } catch (err) {
        await queryRunner.rollbackTransaction();
        throw err;
    } finally {
        await queryRunner.release();
    }
}
```

### ✅ Connection Pooling

```typescript
TypeOrmModule.forRoot({
    type: 'postgres',
    // ...
    poolSize: 10, // Número de conexiones
    extra: {
        max: 10,
        min: 2,
        idleTimeoutMillis: 30000
    }
})
```

### ✅ Soft Delete

```typescript
@Entity()
export class Product {
    @DeleteDateColumn()
    deletedAt?: Date;
}

// En el servicio
async remove(id: string) {
    const product = await this.findOne(id);
    return this.repository.softDelete(id);
}

// Recuperar borrados
async restore(id: string) {
    return this.repository.restore(id);
}
```

---

## Testing

### ✅ Unit Tests

```typescript
describe('ProductService', () => {
    let service: ProductService;
    let repository: Repository<Product>;

    beforeEach(async () => {
        const module: TestingModule = await Test.createTestingModule({
            providers: [
                ProductService,
                {
                    provide: getRepositoryToken(Product),
                    useValue: {
                        find: jest.fn(),
                        findOne: jest.fn(),
                        create: jest.fn(),
                        save: jest.fn(),
                    },
                },
            ],
        }).compile();

        service = module.get<ProductService>(ProductService);
        repository = module.get<Repository<Product>>(getRepositoryToken(Product));
    });

    it('should find all products', async () => {
        const mockProducts = [{ id: '1', name: 'Test' }];
        jest.spyOn(repository, 'find').mockResolvedValue(mockProducts as any);

        const result = await service.findAll({});
        expect(result).toEqual(mockProducts);
    });
});
```

### ✅ E2E Tests

```typescript
describe('Products (e2e)', () => {
    let app: INestApplication;
    let token: string;

    beforeAll(async () => {
        const moduleFixture: TestingModule = await Test.createTestingModule({
            imports: [AppModule],
        }).compile();

        app = moduleFixture.createNestApplication();
        await app.init();

        // Obtener token
        const loginResponse = await request(app.getHttpServer())
            .post('/auth/login')
            .send({ email: 'test@test.com', password: 'password' });
        
        token = loginResponse.body.token;
    });

    it('/products (POST)', () => {
        return request(app.getHttpServer())
            .post('/products')
            .set('Authorization', `Bearer ${token}`)
            .send({ name: 'Test Product', price: 100 })
            .expect(201);
    });
});
```

---

## Documentación

### ✅ Swagger Completo

```typescript
@ApiTags('products')
@Controller('products')
export class ProductsController {
    
    @Post()
    @ApiOperation({ 
        summary: 'Create a new product',
        description: 'Creates a new product with the provided data'
    })
    @ApiResponse({ 
        status: 201, 
        description: 'Product created successfully',
        type: Product 
    })
    @ApiResponse({ 
        status: 400, 
        description: 'Invalid input data' 
    })
    @ApiResponse({ 
        status: 401, 
        description: 'Unauthorized' 
    })
    @ApiBearerAuth('JWT-auth')
    create(@Body() dto: CreateProductDto) {
        return this.service.create(dto);
    }
}
```

### ✅ Comentarios en Código

```typescript
/**
 * Calcula el precio final con descuento e impuestos
 * @param basePrice - Precio base del producto
 * @param discountPercent - Porcentaje de descuento (0-100)
 * @param taxPercent - Porcentaje de impuesto (0-100)
 * @returns Precio final calculado
 */
private calculateFinalPrice(
    basePrice: number,
    discountPercent: number,
    taxPercent: number
): number {
    const discount = basePrice * (discountPercent / 100);
    const priceAfterDiscount = basePrice - discount;
    const tax = priceAfterDiscount * (taxPercent / 100);
    return priceAfterDiscount + tax;
}
```

### ✅ README del Proyecto

```markdown
# Nombre del Proyecto

## Descripción
Breve descripción del proyecto

## Tecnologías
- NestJS
- PostgreSQL
- TypeORM
- JWT

## Instalación
\`\`\`bash
npm install
cp .env.example .env
docker-compose up -d
npm run start:dev
\`\`\`

## Documentación API
http://localhost:3000/api

## Testing
\`\`\`bash
npm run test
npm run test:e2e
\`\`\`
```

---

## Git y Versionado

### ✅ Commits Semánticos

```bash
feat: agregar módulo de productos
fix: corregir validación de email
docs: actualizar README
refactor: mejorar servicio de usuarios
test: agregar tests para auth
perf: optimizar query de búsqueda
style: formatear código
chore: actualizar dependencias
```

### ✅ .gitignore

```
# Dependencies
node_modules/

# Environment
.env
.env.local
.env.*.local

# Database
postgres/

# Compiled
dist/
build/

# Logs
*.log
logs/

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Testing
coverage/
.nyc_output/
```

### ✅ Branch Strategy

```
main (production)
├── develop (staging)
    ├── feature/user-auth
    ├── feature/product-crud
    └── bugfix/login-issue
```

### ✅ Pull Request Template

```markdown
## Descripción
Describe los cambios realizados

## Tipo de cambio
- [ ] Bug fix
- [ ] Nueva feature
- [ ] Breaking change
- [ ] Documentación

## Checklist
- [ ] Tests pasan
- [ ] Código documentado
- [ ] README actualizado
- [ ] Sin conflictos con develop
```

---

## Checklist de Producción

Antes de deploy a producción, verificar:

- [ ] Variables de entorno configuradas correctamente
- [ ] synchronize: false en TypeORM
- [ ] Secrets seguros (JWT, DB password, etc.)
- [ ] Rate limiting configurado
- [ ] CORS configurado apropiadamente
- [ ] Logging configurado
- [ ] Error handling robusto
- [ ] Tests passing
- [ ] Documentación actualizada
- [ ] Migraciones de BD listas
- [ ] Health check endpoint
- [ ] Monitoring configurado
- [ ] Backup strategy definida

---

## Recursos Adicionales

- [NestJS Best Practices](https://docs.nestjs.com/fundamentals)
- [TypeORM Best Practices](https://orkhan.gitbook.io/typeorm/docs/best-practices)
- [Node.js Security Checklist](https://cheatsheetseries.owasp.org/cheatsheets/Nodejs_Security_Cheat_Sheet.html)
- [PostgreSQL Performance Tips](https://wiki.postgresql.org/wiki/Performance_Optimization)

---

**Recuerda:** Estas son guías, no reglas absolutas. Adapta según las necesidades específicas de tu proyecto.
