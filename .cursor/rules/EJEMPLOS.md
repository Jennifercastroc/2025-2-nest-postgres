# Ejemplos Prácticos de Uso - Cursor Rules NestJS

Este documento contiene ejemplos prácticos de cómo las reglas de Cursor te ayudarán en diferentes escenarios.

## Índice
1. [Crear un módulo completo](#1-crear-un-módulo-completo)
2. [Agregar validaciones personalizadas](#2-agregar-validaciones-personalizadas)
3. [Implementar relaciones entre entidades](#3-implementar-relaciones-entre-entidades)
4. [Agregar autenticación a endpoints](#4-agregar-autenticación-a-endpoints)
5. [Crear seeds para datos de prueba](#5-crear-seeds-para-datos-de-prueba)
6. [Implementar búsquedas avanzadas](#6-implementar-búsquedas-avanzadas)

---

## 1. Crear un Módulo Completo

### Prompt para Cursor:
```
Crea un módulo completo de "productos" con las siguientes características:
- Campos: nombre, descripción, precio, stock, categoría
- CRUD completo
- Validaciones apropiadas
- Documentación Swagger
```

### Lo que Cursor generará automáticamente:

#### Entidad (product.entity.ts):
```typescript
import { BeforeInsert, BeforeUpdate, Column, Entity, PrimaryGeneratedColumn } from 'typeorm';
import { ApiProperty } from '@nestjs/swagger';

@Entity()
export class Product {
    @ApiProperty({
        example: '38c8f3fe-xxxx-xxxx-81ff-77fe9846f1dd',
        description: 'Product ID',
        uniqueItems: true
    })
    @PrimaryGeneratedColumn('uuid')
    id: string;

    @ApiProperty({
        example: 'Laptop Dell XPS 15',
        description: 'Product name'
    })
    @Column('text')
    nombre: string;

    @ApiProperty()
    @Column('text')
    descripcion: string;

    @ApiProperty()
    @Column('decimal', { precision: 10, scale: 2 })
    precio: number;

    @ApiProperty()
    @Column('int')
    stock: number;

    @Column('text')
    categoria: string;
}
```

#### DTO (create-product.dto.ts):
```typescript
import { IsString, IsNumber, IsPositive, Min } from 'class-validator';
import { ApiProperty } from '@nestjs/swagger';

export class CreateProductDto {
    @ApiProperty({
        example: 'Laptop Dell XPS 15'
    })
    @IsString()
    nombre: string;

    @ApiProperty()
    @IsString()
    descripcion: string;

    @ApiProperty()
    @IsNumber()
    @IsPositive()
    precio: number;

    @ApiProperty()
    @IsNumber()
    @Min(0)
    stock: number;

    @ApiProperty()
    @IsString()
    categoria: string;
}
```

---

## 2. Agregar Validaciones Personalizadas

### Prompt para Cursor:
```
En el módulo de productos, agrega las siguientes validaciones:
- El precio debe estar entre 1 y 1000000
- El nombre debe tener mínimo 3 caracteres
- La categoría debe ser una de: 'electronica', 'ropa', 'alimentos'
- El stock no puede ser negativo
```

### Cursor actualizará el DTO:
```typescript
import { IsString, IsNumber, IsPositive, Min, Max, MinLength, IsIn } from 'class-validator';

export class CreateProductDto {
    @IsString()
    @MinLength(3)
    nombre: string;

    @IsString()
    descripcion: string;

    @IsNumber()
    @IsPositive()
    @Min(1)
    @Max(1000000)
    precio: number;

    @IsNumber()
    @Min(0)
    stock: number;

    @IsString()
    @IsIn(['electronica', 'ropa', 'alimentos'])
    categoria: string;
}
```

---

## 3. Implementar Relaciones entre Entidades

### Prompt para Cursor:
```
Crea una relación OneToMany entre Product y Review (reseñas).
Cada producto puede tener múltiples reseñas.
Una reseña debe tener: calificación (1-5), comentario, y fecha.
```

### Entidad Review generada:
```typescript
import { Column, Entity, ManyToOne, PrimaryGeneratedColumn } from 'typeorm';
import { Product } from '../../products/entities/product.entity';
import { ApiProperty } from '@nestjs/swagger';

@Entity()
export class Review {
    @ApiProperty()
    @PrimaryGeneratedColumn('uuid')
    id: string;

    @ApiProperty()
    @Column('int')
    calificacion: number;

    @ApiProperty()
    @Column('text')
    comentario: string;

    @ApiProperty()
    @Column('timestamp', { default: () => 'CURRENT_TIMESTAMP' })
    fecha: Date;

    @ManyToOne(
        () => Product,
        (product) => product.reviews,
        { onDelete: 'CASCADE' }
    )
    product: Product;
}
```

### Actualización de Product entity:
```typescript
@OneToMany(
    () => Review,
    (review) => review.product,
    { cascade: true, eager: true }
)
reviews?: Review[];
```

---

## 4. Agregar Autenticación a Endpoints

### Prompt para Cursor:
```
Protege los endpoints de crear, actualizar y eliminar productos:
- Crear: solo usuarios autenticados
- Actualizar: solo role 'admin'
- Eliminar: solo role 'admin'
```

### Cursor actualizará el controlador:
```typescript
import { Auth } from 'src/auth/decorators/auth.decorator';
import { ValidRoles } from 'src/auth/enums/roles.enum';
import { ApiBearerAuth } from '@nestjs/swagger';

@Controller('products')
export class ProductsController {
    @Post()
    @ApiBearerAuth('JWT-auth')
    @Auth() // Solo autenticados
    create(@Body() createProductDto: CreateProductDto) {
        return this.productsService.create(createProductDto);
    }

    @Patch(':id')
    @ApiBearerAuth('JWT-auth')
    @Auth(ValidRoles.admin) // Solo admin
    update(@Param('id') id: string, @Body() updateProductDto: UpdateProductDto) {
        return this.productsService.update(id, updateProductDto);
    }

    @Delete(':id')
    @ApiBearerAuth('JWT-auth')
    @Auth(ValidRoles.admin) // Solo admin
    remove(@Param('id') id: string) {
        return this.productsService.remove(id);
    }
}
```

---

## 5. Crear Seeds para Datos de Prueba

### Prompt para Cursor:
```
Crea un seed para insertar 10 productos de ejemplo en la base de datos
```

### Cursor generará:
```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Product } from '../products/entities/product.entity';

@Injectable()
export class SeedService {
    constructor(
        @InjectRepository(Product)
        private readonly productRepository: Repository<Product>,
    ) {}

    async runSeed() {
        await this.insertProducts();
        return 'Seed ejecutado correctamente';
    }

    private async insertProducts() {
        await this.productRepository.delete({});

        const products = [
            {
                nombre: 'Laptop Dell XPS 15',
                descripcion: 'Laptop de alto rendimiento',
                precio: 1500,
                stock: 10,
                categoria: 'electronica'
            },
            {
                nombre: 'Camiseta Nike',
                descripcion: 'Camiseta deportiva',
                precio: 50,
                stock: 100,
                categoria: 'ropa'
            },
            // ... más productos
        ];

        await this.productRepository.save(
            this.productRepository.create(products)
        );
    }
}
```

---

## 6. Implementar Búsquedas Avanzadas

### Prompt para Cursor:
```
Implementa un endpoint de búsqueda que permita:
- Buscar por nombre (case insensitive)
- Filtrar por categoría
- Filtrar por rango de precios
- Ordenar por precio o nombre
- Paginación
```

### DTO de búsqueda generado:
```typescript
import { IsOptional, IsString, IsNumber, IsIn, Min } from 'class-validator';
import { Type } from 'class-transformer';
import { ApiProperty } from '@nestjs/swagger';

export class SearchProductDto {
    @ApiProperty({ required: false })
    @IsOptional()
    @IsString()
    nombre?: string;

    @ApiProperty({ required: false })
    @IsOptional()
    @IsString()
    categoria?: string;

    @ApiProperty({ required: false })
    @IsOptional()
    @IsNumber()
    @Type(() => Number)
    @Min(0)
    precioMin?: number;

    @ApiProperty({ required: false })
    @IsOptional()
    @IsNumber()
    @Type(() => Number)
    precioMax?: number;

    @ApiProperty({ required: false })
    @IsOptional()
    @IsIn(['precio', 'nombre'])
    orderBy?: string;

    @ApiProperty({ required: false })
    @IsOptional()
    @IsIn(['ASC', 'DESC'])
    order?: 'ASC' | 'DESC';

    @ApiProperty({ required: false })
    @IsOptional()
    @IsNumber()
    @Type(() => Number)
    limit?: number;

    @ApiProperty({ required: false })
    @IsOptional()
    @IsNumber()
    @Type(() => Number)
    offset?: number;
}
```

### Método de servicio generado:
```typescript
async search(searchDto: SearchProductDto) {
    const {
        nombre,
        categoria,
        precioMin,
        precioMax,
        orderBy = 'nombre',
        order = 'ASC',
        limit = 10,
        offset = 0
    } = searchDto;

    const queryBuilder = this.productRepository
        .createQueryBuilder('product');

    if (nombre) {
        queryBuilder.andWhere('UPPER(product.nombre) LIKE :nombre', {
            nombre: `%${nombre.toUpperCase()}%`
        });
    }

    if (categoria) {
        queryBuilder.andWhere('product.categoria = :categoria', { categoria });
    }

    if (precioMin !== undefined) {
        queryBuilder.andWhere('product.precio >= :precioMin', { precioMin });
    }

    if (precioMax !== undefined) {
        queryBuilder.andWhere('product.precio <= :precioMax', { precioMax });
    }

    queryBuilder
        .orderBy(`product.${orderBy}`, order)
        .take(limit)
        .skip(offset);

    const [products, total] = await queryBuilder.getManyAndCount();

    return {
        data: products,
        total,
        limit,
        offset
    };
}
```

---

## Más Ejemplos de Prompts Útiles

### Agregar soft delete
```
Implementa soft delete en el módulo de productos.
Los productos eliminados deben marcarse como inactivos en lugar de borrarse.
```

### Agregar auditoría
```
Agrega campos de auditoría (createdAt, updatedAt) a la entidad Product
```

### Implementar transacciones
```
Crea un método en el servicio de productos que actualice el stock 
y cree un registro de historial en una sola transacción
```

### Agregar caché
```
Implementa caché en el método findAll de productos usando @nestjs/cache-manager
```

### Exportar a Excel
```
Crea un endpoint para exportar la lista de productos a Excel
```

---

## Tips para Mejores Prompts

1. **Sé específico**: Menciona campos, validaciones y requisitos exactos
2. **Incluye contexto**: "En el módulo de productos..." o "Para la entidad User..."
3. **Menciona relaciones**: "Relaciona X con Y usando OneToMany"
4. **Especifica roles**: "Solo admin debe poder..." o "Usuarios autenticados pueden..."
5. **Incluye ejemplos**: "Por ejemplo, el nombre podría ser 'Laptop Dell XPS 15'"

## Solución de Problemas

### Si Cursor no sigue las reglas:
1. Verifica que el archivo `.cursorrules` esté en la ubicación correcta
2. Reinicia Cursor
3. Asegúrate de que el archivo no tenga errores de sintaxis
4. Sé más explícito en tus prompts

### Si necesitas sobreescribir una regla temporalmente:
```
Ignora las reglas de cursor y genera X de la siguiente manera...
```

---

**Nota:** Estos ejemplos son punto de partida. Cursor puede adaptar y mejorar el código según tu contexto específico.
