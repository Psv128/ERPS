# Memoria del Proyecto Odoo

## Desarrollo de una Tienda Digital con Odoo y Creación de Módulos Personalizados

---

## Introducción

Este documento recoge de forma detallada el trabajo realizado con **Odoo**, centrado en dos grandes pilares. Por un lado, el uso de la **aplicación Sitio Web y Comercio Electrónico** para la creación de una tienda digital completa. Por otro, el **desarrollo de módulos personalizados**, explicando cómo extender Odoo para añadir nuevas funcionalidades adaptadas a necesidades concretas.

La memoria está pensada para ser leída desde **GitHub en formato Markdown**, por lo que se ha cuidado no solo el contenido técnico, sino también la **estructura visual**, incluyendo **tablas, imágenes referenciadas**, diagramas conceptuales y bloques destacados para que la lectura no sea monótona.

---

# Automatización de Correos Electrónicos en Odoo

> ✉️ *La comunicación automática es clave en cualquier aplicación empresarial moderna.*

## 1. ¿Qué son los correos automatizados en Odoo?

Los correos automatizados permiten enviar mensajes cuando ocurre un evento determinado dentro del sistema:

| Evento                    | Acción            | Destinatario  |
| ------------------------- | ----------------- | ------------- |
| Confirmación de pedido    | Enviar correo     | Cliente       |
| Registro de nuevo usuario | Enviar bienvenida | Usuario       |
| Cambio de estado de venta | Notificación      | Administrador |

---

## 2. Plantillas de correo

Cada plantilla define:

| Campo     | Descripción                                                        |
| --------- | ------------------------------------------------------------------ |
| Asunto    | Título del correo, puede incluir variables dinámicas               |
| Contenido | Mensaje principal con formato HTML o texto plano                   |
| Variables | Permiten insertar datos del sistema, por ejemplo {{ object.name }} |

Ejemplo conceptual:

```
📧 Asunto: Confirmación de pedido {{ object.name }}

Hola {{ object.partner_id.name }},

Tu pedido ha sido confirmado correctamente.
Gracias por confiar en nuestra tienda.
```

---

## 3. Automatización mediante acciones

Se utilizan **acciones automatizadas** para que los correos se envíen al producirse un evento:

```
[ Pedido confirmado ]
          ↓
[ Acción automática ]
          ↓
[ Enviar correo al cliente ]
```

💡 *Estas acciones pueden aplicarse a múltiples eventos y se configuran desde el backend de Odoo.*

---

# PARTE I – Creación de una Tienda Digital con Odoo

## 1. Activación de aplicaciones necesarias

| Aplicación           | Función                                |
| -------------------- | -------------------------------------- |
| Sitio Web            | Base para crear páginas web            |
| Comercio Electrónico | Añade funcionalidades de tienda online |
| Inventario           | Gestión de stock                       |
| Ventas / Facturación | Gestión de pedidos y pagos             |

---

## 2. Creación del sitio web

![Ejemplo de editor de Odoo](https://odoocdn.com/openerp_website/static/src/img/apps/website/hero_image.webp)

El editor visual permite usar **bloques drag & drop**: texto, imágenes, botones y productos. Se pueden personalizar colores, tipografía y diseño de la página.

---

## 3. Creación de productos

| Campo       | Descripción                     |
| ----------- | ------------------------------- |
| Nombre      | Nombre del producto             |
| Precio      | Precio en la moneda configurada |
| Imagen      | Imagen principal del producto   |
| Descripción | Detalle de características      |
| Categoría   | Organización en la tienda       |
| Stock       | Cantidad disponible             |

---

## 4. Gestión del carrito y proceso de compra

El flujo de compra se puede representar con un diagrama simple:

```
[ Cliente añade producto ]
          ↓
[ Carrito ]
          ↓
[ Checkout ]
          ↓
[ Confirmación de pedido ]
```

---

# PARTE II – Desarrollo de Módulos Personalizados en Odoo

## 1. Estructura básica de un módulo

```
mi_modulo/
├── __manifest__.py
├── __init__.py
├── models/
│   └── __init__.py
├── views/
│   └── vistas.xml
├── security/
│   └── ir.model.access.csv
```

## 2. Archivo manifest

```python
{
    'name': 'Modulo Tienda Personalizada',
    'version': '1.0',
    'depends': ['base', 'website_sale'],
    'data': [
        'views/vistas.xml',
    ],
}
```

---

## 3. Creación de modelos

```python
from odoo import models, fields

class ProductoExtra(models.Model):
    _name = 'producto.extra'

    name = fields.Char(string='Nombre')
    descripcion = fields.Text(string='Descripción')
    activo = fields.Boolean(default=True)
```

---

## 4. Vistas y formularios

```xml
<form string="Producto Extra">
    <sheet>
        <group>
            <field name="name"/>
            <field name="descripcion"/>
            <field name="activo"/>
        </group>
    </sheet>
</form>
```

---

## 5. Seguridad y permisos

| Grupo         | Modelo         | Permisos                        |
| ------------- | -------------- | ------------------------------- |
| Administrador | producto.extra | Crear, Leer, Escribir, Eliminar |
| Usuario       | producto.extra | Leer, Crear                     |

---

## 6. Extensión del sitio web

Los módulos pueden modificar el sitio web añadiendo nuevas páginas, bloques o funcionalidades personalizadas. Esto se hace combinando XML, Python y **QWeb** para plantillas web.

![Ejemplo de módulo personalizado en sitio web](https://i.ytimg.com/vi/gPuAFqxpgng/hq720.jpg?sqp=-oaymwE7CK4FEIIDSFryq4qpAy0IARUAAAAAGAElAADIQj0AgKJD8AEB-AH-CYAC0AWKAgwIABABGE8gXShlMA8=&rs=AOn4CLDCWzxUSFoSE8iNTtEMInjDMeDfQQ)

---

## Conclusión

Odoo permite crear **tiendas digitales completas** y desarrollar **módulos personalizados**. Con las automatizaciones, plantillas de correo, flujo de compras y desarrollo modular, se logra una solución profesional, centralizada y adaptable a cualquier negocio.


