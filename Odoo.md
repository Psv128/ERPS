# Memoria del Proyecto Odoo

## Desarrollo de una Tienda Digital con Odoo y Creación de Módulos Personalizados

---

## Introducción

Este documento recoge de forma detallada el trabajo realizado con **Odoo**, centrado en dos grandes pilares:

- El uso de la **aplicación Sitio Web y Comercio Electrónico (website, website_sale)** para la creación de una tienda digital completa.
- El **desarrollo de módulos personalizados** que permiten extender las funcionalidades del sistema (modelos, vistas, seguridad, controladores, QWeb, integraciones y pruebas).

La memoria está pensada para ser leída desde **GitHub en formato Markdown**. Se cuida tanto el contenido técnico como la **estructura visual**, incluyendo tablas, diagramas, bloques de código completos y enlaces útiles.

---

# Automatización de Correos Electrónicos en Odoo

> ✉️ La comunicación automática es clave en cualquier aplicación empresarial moderna.

## 1. Concepto y eventos de disparo

Los correos automatizados permiten enviar mensajes cuando ocurre un evento dentro del sistema:

| Evento                          | Acción                         | Destinatario    |
|---------------------------------|--------------------------------|-----------------|
| Confirmación de pedido (venta)  | Enviar correo de confirmación  | Cliente         |
| Registro de nuevo usuario       | Enviar correo de bienvenida    | Usuario         |
| Cambio de estado de venta       | Notificar al administrador     | Administrador   |
| Pedido con retraso (cron)       | Recordatorio                    | Cliente/ventas  |

Los eventos pueden ser:
- Acciones del usuario (crear/escribir en un registro).
- Cambios automáticos del sistema (transiciones de estado).
- Tareas programadas mediante `ir.cron`.

---

## 2. Plantillas de correo (`mail.template`)

Las plantillas permiten definir asunto, destinatarios y cuerpo del mensaje usando Jinja2 con datos del registro (objeto) y del entorno.

```xml
<odoo>
  <data noupdate="0">
    <!-- Plantilla de confirmación de pedido -->
    <record id="mail_template_sale_confirmation" model="mail.template">
      <field name="name">Confirmación de Pedido</field>
      <field name="model_id" ref="sale.model_sale_order"/>
      <field name="subject">Confirmación de pedido {{ object.name }}</field>
      <field name="email_from">${(object.company_id.email or 'ventas@example.com')}</field>
      <field name="email_to">${object.partner_id.email}</field>
      <field name="body_html" type="html">
        <![CDATA[
          <div>
            <p>Hola {{ object.partner_id.name }},</p>
            <p>Tu pedido <strong>{{ object.name }}</strong> ha sido confirmado correctamente.</p>
            <p>Importe total: {{ format_amount(object.amount_total, object.currency_id) }}</p>
            <p>Gracias por confiar en nuestra tienda.</p>
          </div>
        ]]>
      </field>
    </record>
  </data>
</odoo>
```

Notas:
- `model_id` vincula la plantilla al modelo (`sale.order`).
- `email_to` puede ser dinámico según el registro (`object.partner_id.email`).
- `body_html` soporta HTML y variables Jinja2 (`object`, `env`, helpers como `format_amount`).

---

## 3. Envío desde Python

Desde código, podemos cargar y enviar una plantilla con el registro actual:

```python
from odoo import models

class SaleOrder(models.Model):
    _inherit = 'sale.order'

    def action_send_confirmation_email(self):
        template = self.env.ref('mi_modulo.mail_template_sale_confirmation')
        for order in self:
            # Enviar inmediatamente (crea mail.mail y lo procesa)
            template.send_mail(order.id, force_send=True)
            # O postear mensaje en el chatter
            order.message_post(body="Correo de confirmación enviado.")
```

También podemos construir y enviar mensajes sin plantilla:

```python
from odoo import models

class SaleOrder(models.Model):
    _inherit = 'sale.order'

    def _send_manual_email(self, subject, body):
        Mail = self.env['mail.mail']
        for order in self:
            mail = Mail.create({
                'subject': subject,
                'body_html': body,
                'email_to': order.partner_id.email,
                'email_from': order.company_id.email or self.env.user.email,
            })
            mail.send()
```

---

## 4. Acciones automatizadas (`base.automation`) y servidor (`ir.actions.server`)

Podemos automatizar el envío al confirmar un pedido. Dos formas complementarias:

1) Acción de servidor que llama al método de envío:

```xml
<odoo>
  <data>
    <record id="server_action_send_sale_confirmation" model="ir.actions.server">
      <field name="name">Enviar correo de confirmación</field>
      <field name="model_id" ref="sale.model_sale_order"/>
      <field name="state">code</field>
      <field name="code">
        <![CDATA[
# record es sale.order
if record:
    record.action_send_confirmation_email()
        ]]>
      </field>
    </record>
  </data>
</odoo>
```

2) Automatización que dispara la acción al cambiar el estado:

```xml
<odoo>
  <data>
    <record id="automation_send_on_confirmation" model="base.automation">
      <field name="name">Auto email al confirmar pedido</field>
      <field name="model_id" ref="sale.model_sale_order"/>
      <field name="trigger">on_write</field>
      <field name="filter_domain">[("state", "=", "sale")]</field>
      <field name="action_server_id" ref="mi_modulo.server_action_send_sale_confirmation"/>
      <field name="active">True</field>
    </record>
  </data>
</odoo>
```

Notas:
- `trigger=on_write` y `filter_domain` controlan cuándo ejecutar.
- También se pueden usar `on_create`, `on_timer` (con `ir.cron`).

---

## 5. Tareas programadas (`ir.cron`)

Para recordatorios periódicos:

```xml
<odoo>
  <data noupdate="1">
    <record id="cron_order_delay_reminder" model="ir.cron">
      <field name="name">Recordatorio pedidos retrasados</field>
      <field name="model_id" ref="sale.model_sale_order"/>
      <field name="state">code</field>
      <field name="code">
        <![CDATA[
# Buscar pedidos confirmados sin entrega tras 7 días
orders = env['sale.order'].search([
    ('state', '=', 'sale'),
    ('commitment_date', '!=', False),
    ('commitment_date', '<', (datetime.datetime.now() - datetime.timedelta(days=7)).date())
])
for o in orders:
    o.message_post(body="Tu pedido está en proceso. Gracias por tu paciencia.")
        ]]>
      </field>
      <field name="interval_number">1</field>
      <field name="interval_type">days</field>
      <field name="active">True</field>
    </record>
  </data>
</odoo>
```

---

# PARTE I – Creación de una Tienda Digital con Odoo

## 1. Aplicaciones necesarias y configuración inicial

| Aplicación            | Función                                   |
|----------------------|--------------------------------------------|
| `website`            | Base para crear páginas web                |
| `website_sale`       | Funcionalidades de tienda online           |
| `stock`              | Gestión de inventario y movimientos        |
| `sale` / `account`   | Gestión de pedidos, facturas y pagos       |
| `delivery`           | Envíos y transportistas                    |
| `payment`            | Pasarelas de pago (Stripe, PayPal, etc.)   |

Pasos:
1. Instalar módulos desde Apps.
2. Configurar idioma, moneda y zona horaria.
3. Ajustar impuestos y precios (incluido/excluido).
4. Configurar pasarelas de pago y métodos de envío.

---

## 2. Creación y personalización del sitio web

El editor visual permite usar bloques drag & drop: texto, imágenes, botones y productos. Se pueden personalizar colores, tipografía y diseño.

Páginas típicas:
- Inicio (destacados, categorías, promociones).
- Catálogo y categorías.
- Detalle de producto.
- Carrito y Checkout.
- Contacto y política de privacidad.

---

## 3. Productos y variantes

Campos clave:
- Nombre, precio, impuestos, imagen, descripción.
- Categoría, etiquetas.
- Stock, rutas (fabricación, compra), tipo de producto.

Variantes (atributos): color, talla, material.

```python
from odoo import models, fields

class ProductTemplate(models.Model):
    _inherit = 'product.template'

    is_featured = fields.Boolean(string='Destacado en web')
```

Configuración de atributos: `product.attribute` y `product.attribute.value` vinculan variantes a `product.template`.

---

## 4. Carrito y proceso de compra

Flujo básico:

```
[ Cliente añade producto ]
          ↓
[ Carrito ]
          ↓
[ Checkout ]
          ↓
[ Confirmación de pedido ]
```

Opciones:
- Checkout invitado o registro.
- Métodos de pago (Stripe, PayPal, transferencia).
- Métodos de envío: tarifas por peso, precio, reglas.

---

## 5. Pago y envío

- Configurar `payment.provider` (Stripe/PayPal) con credenciales y estados.
- Configurar `delivery.carrier` para tarifas dinámicas o fijas.

Ejemplo de tarifa personalizada (regla simple):

```python
from odoo import models, fields

class DeliveryCarrier(models.Model):
    _inherit = 'delivery.carrier'

    free_over_amount = fields.Float(string='Envío gratis sobre importe')

    def rate_shipment(self, order):
        price = super().rate_shipment(order)
        if self.free_over_amount and order.amount_total >= self.free_over_amount:
            price['price'] = 0.0
        return price
```

---

# PARTE II – Desarrollo de Módulos Personalizados en Odoo

## 1. Estructura básica de un módulo

```
mi_modulo/
├── __manifest__.py
├── __init__.py
├── models/
│   ├── __init__.py
│   └── product_extra.py
├── views/
│   ├── product_extra_views.xml
│   └── templates.xml
├── security/
│   ├── ir.model.access.csv
│   └── security.xml
├── data/
│   ├── mail_templates.xml
│   └── automation.xml
```

---

## 2. Manifest

```python
{
    'name': 'Módulo Tienda Personalizada',
    'version': '1.0.0',
    'author': 'Tu Nombre',
    'category': 'Website',
    'depends': ['base', 'website', 'website_sale', 'sale', 'mail'],
    'data': [
        'security/security.xml',
        'security/ir.model.access.csv',
        'views/product_extra_views.xml',
        'views/templates.xml',
        'data/mail_templates.xml',
        'data/automation.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'mi_modulo/static/src/css/style.css',
            'mi_modulo/static/src/js/extra.js',
        ],
    },
    'license': 'LGPL-3',
}
```

---

## 3. Modelos: relaciones, campos computados y restricciones

```python
from odoo import models, fields, api
from odoo.exceptions import ValidationError

class ProductoExtra(models.Model):
    _name = 'producto.extra'
    _description = 'Producto Extra'

    name = fields.Char(required=True)
    descripcion = fields.Text()
    activo = fields.Boolean(default=True)

    product_id = fields.Many2one('product.product', string='Producto', required=True)
    tags_ids = fields.Many2many('product.tag', string='Etiquetas')

    precio_base = fields.Float(default=0.0)
    impuesto = fields.Float(default=0.0)
    precio_total = fields.Float(compute='_compute_precio_total', store=True)

    linea_ids = fields.One2many('producto.extra.line', 'extra_id', string='Líneas')

    @api.depends('precio_base', 'impuesto')
    def _compute_precio_total(self):
        for rec in self:
            rec.precio_total = rec.precio_base + rec.impuesto

    @api.constrains('precio_base', 'impuesto')
    def _check_precio(self):
        for rec in self:
            if rec.precio_base < 0 or rec.impuesto < 0:
                raise ValidationError('Los importes no pueden ser negativos')

class ProductoExtraLine(models.Model):
    _name = 'producto.extra.line'
    _description = 'Línea Producto Extra'

    extra_id = fields.Many2one('producto.extra', required=True, ondelete='cascade')
    name = fields.Char(required=True)
    cantidad = fields.Integer(default=1)
    precio_unitario = fields.Float(default=0.0)
    subtotal = fields.Float(compute='_compute_subtotal', store=True)

    @api.depends('cantidad', 'precio_unitario')
    def _compute_subtotal(self):
        for rec in self:
            rec.subtotal = rec.cantidad * rec.precio_unitario
```

---

## 4. Vistas: árbol, formulario, búsqueda y acción

```xml
<odoo>
  <data>
    <!-- Acción y menú -->
    <record id="action_producto_extra" model="ir.actions.act_window">
      <field name="name">Productos Extra</field>
      <field name="res_model">producto.extra</field>
      <field name="view_mode">tree,form</field>
    </record>

    <menuitem id="menu_producto_extra_root" name="Extras" parent="product.menu_products"/>
    <menuitem id="menu_producto_extra" name="Productos Extra" parent="menu_producto_extra_root" action="action_producto_extra"/>

    <!-- Vista árbol -->
    <record id="view_producto_extra_tree" model="ir.ui.view">
      <field name="name">producto.extra.tree</field>
      <field name="model">producto.extra</field>
      <field name="arch" type="xml">
        <tree>
          <field name="name"/>
          <field name="product_id"/>
          <field name="precio_total"/>
          <field name="activo"/>
        </tree>
      </field>
    </record>

    <!-- Vista formulario -->
    <record id="view_producto_extra_form" model="ir.ui.view">
      <field name="name">producto.extra.form</field>
      <field name="model">producto.extra</field>
      <field name="arch" type="xml">
        <form string="Producto Extra">
          <sheet>
            <group>
              <field name="name"/>
              <field name="product_id"/>
              <field name="descripcion"/>
              <field name="precio_base"/>
              <field name="impuesto"/>
              <field name="precio_total" readonly="1"/>
              <field name="activo"/>
            </group>
            <notebook>
              <page string="Líneas">
                <field name="linea_ids">
                  <tree editable="bottom">
                    <field name="name"/>
                    <field name="cantidad"/>
                    <field name="precio_unitario"/>
                    <field name="subtotal" readonly="1"/>
                  </tree>
                </field>
              </page>
              <page string="Etiquetas">
                <field name="tags_ids" widget="many2many_tags"/>
              </page>
            </notebook>
          </sheet>
        </form>
      </field>
    </record>

    <!-- Vista de búsqueda -->
    <record id="view_producto_extra_search" model="ir.ui.view">
      <field name="name">producto.extra.search</field>
      <field name="model">producto.extra</field>
      <field name="arch" type="xml">
        <search>
          <field name="name"/>
          <field name="product_id"/>
          <filter string="Activos" domain="[('activo','=',True)]"/>
        </search>
      </field>
    </record>
  </data>
</odoo>
```

---

## 5. Seguridad: grupos, reglas y accesos

`security.xml`:

```xml
<odoo>
  <data>
    <record id="group_producto_extra_user" model="res.groups">
      <field name="name">Usuario Productos Extra</field>
    </record>
    <record id="group_producto_extra_manager" model="res.groups">
      <field name="name">Gestor Productos Extra</field>
      <field name="implied_ids" eval="[(4, ref('group_producto_extra_user'))]"/>
    </record>

    <!-- Regla: Usuarios solo ven activos -->
    <record id="rule_producto_extra_user_active" model="ir.rule">
      <field name="name">Ver solo activos</field>
      <field name="model_id" ref="model_producto_extra"/>
      <field name="groups" eval="[(4, ref('group_producto_extra_user'))]"/>
      <field name="domain_force">[("activo", "=", True)]</field>
    </record>
  </data>
</odoo>
```

`ir.model.access.csv`:

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_producto_extra_user,producto.extra user,model_producto_extra,group_producto_extra_user,1,0,1,0
access_producto_extra_manager,producto.extra manager,model_producto_extra,group_producto_extra_manager,1,1,1,1
access_producto_extra_line_manager,producto.extra.line manager,model_producto_extra_line,group_producto_extra_manager,1,1,1,1
```

---

## 6. Controladores y rutas web

Definir endpoints para páginas o APIs:

```python
from odoo import http
from odoo.http import request

class ExtraController(http.Controller):

    @http.route(['/extras'], type='http', auth='public', website=True)
    def extras_page(self, **kwargs):
        extras = request.env['producto.extra'].sudo().search([('activo', '=', True)])
        return request.render('mi_modulo.extras_template', {'extras': extras})

    @http.route(['/api/extras'], type='json', auth='user')
    def extras_api(self):
        extras = request.env['producto.extra'].search([])
        return [{'id': e.id, 'name': e.name, 'precio_total': e.precio_total} for e in extras]
```

---

## 7. QWeb y plantillas para frontend

```xml
<odoo>
  <template id="extras_template" name="Página Extras">
    <t t-call="website.layout">
      <div class="container">
        <h1>Productos Extra</h1>
        <div class="row">
          <t t-foreach="extras" t-as="ex">
            <div class="col-md-4">
              <div class="card mb-3">
                <div class="card-body">
                  <h5 t-esc="ex.name"/>
                  <p t-esc="ex.descripcion"/>
                  <p><strong t-esc="ex.precio_total"/> €</p>
                </div>
              </div>
            </div>
          </t>
        </div>
      </div>
    </t>
  </template>
</odoo>
```

Crear snippets personalizados para el editor web (opcional):

```xml
<odoo>
  <template id="snippet_extra" inherit_id="website.snippets" name="Snippet Extra">
    <xpath expr="//div[@id='snippet_structure']" position="inside">
      <div class="o_snippet_body" data-snippet="snippet_extra_block" data-name="Bloque Extra">
        <section class="o_extra_block">
          <h2>Bloque Extra</h2>
          <p>Contenido personalizado</p>
        </section>
      </div>
    </xpath>
  </template>
</odoo>
```

---

## 8. Wizards (modelos transitorios)

Para acciones guiadas con diálogos:

```python
from odoo import models, fields

class ExtraWizard(models.TransientModel):
    _name = 'extra.wizard'
    _description = 'Asistente de operación'

    extra_id = fields.Many2one('producto.extra', required=True)
    comentario = fields.Text()

    def action_apply(self):
        self.ensure_one()
        self.extra_id.message_post(body=f"Operación aplicada: {self.comentario}")
        return {'type': 'ir.actions.act_window_close'}
```

Vista del wizard y botón:

```xml
<odoo>
  <record id="view_extra_wizard" model="ir.ui.view">
    <field name="name">extra.wizard.form</field>
    <field name="model">extra.wizard</field>
    <field name="arch" type="xml">
      <form string="Asistente">
        <group>
          <field name="extra_id"/>
          <field name="comentario"/>
        </group>
        <footer>
          <button string="Aplicar" type="object" name="action_apply" class="btn-primary"/>
          <button string="Cancelar" class="btn-secondary" special="cancel"/>
        </footer>
      </form>
    </field>
  </record>
</odoo>
```

---

## 9. Pruebas de módulos

Pruebas unitarias y transaccionales:

```python
from odoo.tests import TransactionCase

class TestProductoExtra(TransactionCase):
    def test_precio_total(self):
        extra = self.env['producto.extra'].create({
            'name': 'Prueba',
            'product_id': self.env.ref('product.product_product_1').id,
            'precio_base': 10,
            'impuesto': 2,
        })
        self.assertEqual(extra.precio_total, 12)
```

---

## 10. Integraciones externas (API)

Uso de `requests` y rutas JSON seguras:

```python
import requests
from odoo import models

class ExternalSync(models.Model):
    _name = 'external.sync'

    def action_fetch(self):
        resp = requests.get('https://api.example.com/data', timeout=10)
        resp.raise_for_status()
        data = resp.json()
        # Procesar datos...
```

---

## 11. Rendimiento y buenas prácticas

- Usar `store=True` en campos computados para evitar recomputar.
- Añadir índices (`index=True`) en campos de búsqueda frecuente.
- Evitar N+1: usar `read_group`, `search_read`, o precargar relaciones.
- Limitar escrituras a cambios necesarios.
- Usar crons para tareas pesadas fuera de la transacción del usuario.
- Cachear datos en `ir.config_parameter` cuando proceda.

---

## 12. Seguridad y hardening

- Revisar grupos y reglas de acceso; principio de mínimo privilegio.
- Evitar exponer endpoints JSON públicos con datos sensibles.
- Validar entradas de usuario en controladores.
- Usar `sudo()` solo cuando sea imprescindible.

---

## 13. Despliegue y configuración

Ejemplo `odoo.conf`:

```ini
[options]
db_host = 127.0.0.1
db_port = 5432
db_user = odoo
db_password = ********
addons_path = /opt/odoo/addons,/opt/odoo/custom
workers = 4
limit_memory_hard = 1024
limit_time_cpu = 120
logfile = /var/log/odoo/odoo.log
```

Pasos:
- Entornos separados (dev/stage/prod).
- Migraciones con `-u mi_modulo` tras cambios.
- Backups de base de datos y filestore.

---
