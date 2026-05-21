# 🌌 Plan de Implementación de Ingeniería Backend: NebriAmazon 🛍️

Este documento representa el **Plan de Implementación de Ingeniería del Lado del Servidor (Backend)** para **NebriAmazon**, derivado de las especificaciones maestras del *PRD_Global_NebriAmazon.md*. Establece la arquitectura del backend, el diseño físico y de optimización de la base de datos relacional PostgreSQL, el diseño detallado de los *Service Objects* para el desacoplamiento de la lógica de negocio, y las directrices de seguridad y concurrencia transaccional, sirviendo como la guía técnica definitiva para los agentes `backend-developer`, `backend-tester` y `code-inspector`.

---

## 🏛️ 1. Arquitectura de Software y Pila Tecnológica

El backend se construirá como una **API REST desacoplada y sin estado (Stateless API)** de alto rendimiento, optimizada para responder con latencias mínimas y capaz de manejar transacciones atómicas concurrentes de forma robusta.

### 🛠️ Core Stack del Backend
*   **Framework base:** Ruby on Rails (configurado exclusivamente en modo `--api` para omitir middleware de procesamiento HTML innecesario).
*   **Gestor de Base de Datos:** PostgreSQL (con soporte nativo para identificadores UUID y búsquedas altamente indexadas).
*   **Gestor de Autenticación:** JSON Web Tokens (JWT) firmados mediante algoritmo simétrico HMAC (SHA-256) y contraseñas hasheadas en base de datos mediante la gema **Bcrypt**.
*   **Patrón de Diseño:** **Service Objects** independientes de ActiveRecord para la abstracción de la lógica transaccional y el cumplimiento del principio de responsabilidad única (SRP).
*   **Suite de Pruebas:** RSpec para BDD (Behavior-Driven Development), complementado con `FactoryBot` para la generación limpia de registros de prueba y `DatabaseCleaner` para mantener la consistencia transaccional de los datos durante la ejecución de los tests.

---

## 🐘 2. Modelo de Datos Físico y Estrategia de Indexación (PostgreSQL)

Para garantizar integridad referencial absoluta a nivel de base de datos, aislamiento transaccional y búsquedas eficientes a velocidad constante $O(1)$, la persistencia física en PostgreSQL se estructurará siguiendo los esquemas definidos en el PRD:

### 2.1. Estructura de las Tablas y Migraciones

Se habilitará la extensión `pgcrypto` en PostgreSQL para permitir la generación nativa de identificadores UUIDv4 para las entidades principales (`users`, `products`, `carts` y `orders`).

```ruby
# db/migrate/20260521000000_enable_pgcrypto.rb
class EnablePgcrypto < ActiveRecord::Migration[7.1]
  def change
    enable_extension 'pgcrypto' unless extension_enabled?('pgcrypto')
  end
end
```

#### 👤 Tabla `users`
*   `id`: `uuid` PK (autogenerado mediante `gen_random_uuid()`).
*   `name`: `string` (validación de presencia, longitud mínima 2).
*   `email`: `string` UK (validación estricta de formato regex, guardado en minúsculas, índice único).
*   `password_digest`: `string` (cifrado con Bcrypt).
*   `role`: `string` (restricción por base de datos: `admin` o `customer`).
*   `created_at` / `updated_at`: `timestamptz`.

#### 📂 Tabla `categories`
*   `id`: `bigint` PK (autoincremental).
*   `name`: `string` UK (índice único, no nulo).

#### 📦 Tabla `products`
*   `id`: `uuid` PK.
*   `name`: `string` (no nulo).
*   `description`: `text`.
*   `price`: `decimal` (precisión: 10, escala: 2, no nulo, mayor o igual a 0).
*   `stock`: `integer` (no nulo, mayor o igual a 0).
*   `category_id`: `bigint` FK (índice, eliminación en cascada desactivada para conservar historiales).
*   `sku`: `string` UK (índice único, restricción de formato).
*   `image_urls`: `string` array (para admitir múltiples imágenes estáticas por artículo).
*   `deleted_at`: `timestamptz` (índice parcial para control de *Soft Delete*).

#### 🛒 Tabla `carts`
*   `id`: `uuid` PK.
*   `user_id`: `uuid` FK (índice único, restricción 1:1, eliminación en cascada activa).

#### 🍏 Tabla `cart_items`
*   `id`: `bigint` PK.
*   `cart_id`: `uuid` FK (índice indexado, eliminación en cascada activa).
*   `product_id`: `uuid` FK (índice indexado).
*   `quantity`: `integer` (no nulo, mayor que 0).
*   *Restricción:* Índice único compuesto `[cart_id, product_id]` para evitar la duplicidad física de un producto en el mismo carrito.

#### 🧾 Tabla `orders`
*   `id`: `uuid` PK.
*   `user_id`: `uuid` FK (índice indexado).
*   `status`: `string` (restricción por base de datos: `pending`, `paid`, `shipped`, `delivered`, `cancelled`).
*   `total_price`: `decimal` (precisión: 12, escala: 2, no nulo).
*   `created_at` / `updated_at`: `timestamptz` (índice indexado en `created_at` para historiales).

#### 🔍 Tabla `order_items`
*   `id`: `bigint` PK.
*   `order_id`: `uuid` FK (índice indexado, eliminación en cascada activa).
*   `product_id`: `uuid` FK (índice).
*   `quantity`: `integer` (no nulo).
*   `price_snapshot`: `decimal` (precisión: 10, escala: 2, no nulo, captura histórica del precio del producto al momento de comprar).

---

### 2.2. Migración Crítica de Productos e Índices

A continuación, se detalla la estructura física recomendada para la migración de productos, incluyendo el índice parcial para *Soft Delete*:

```ruby
# db/migrate/20260521000002_create_products.rb
class CreateProducts < ActiveRecord::Migration[7.1]
  def change
    create_table :products, id: :uuid do |t|
      t.string :name, null: false
      t.text :description
      t.decimal :price, precision: 10, scale: 2, null: false
      t.integer :stock, null: false, default: 0
      t.references :category, type: :bigint, null: false, foreign_key: true
      t.string :sku, null: false
      t.string :image_urls, array: true, default: []
      t.datetime :deleted_at

      t.timestamps
    end

    add_index :products, :sku, unique: true
    # Índice parcial: Excluye de los escaneos de búsqueda los productos borrados lógicamente
    add_index :products, :deleted_at, where: "deleted_at IS NULL"
  end
end
```

---

## 🔗 3. Contrato de API REST Completo e Implementación de Controladores

Todos los controladores del backend heredarán de `ApplicationController` y retornarán respuestas JSON consistentes. Los errores de validación o fallos del sistema utilizarán esquemas de respuesta normalizados:

```json
{
  "error": "Tipo de error semántico (ej. ValidationError / Unauthenticated)",
  "message": "Descripción detallada del error",
  "details": { "campo": ["motivo del fallo"] }
}
```

### 3.1. Enrutamiento del Backend (`config/routes.rb`)

```ruby
Rails.application.routes.draw do
  namespace :api, defaults: { format: :json } do
    # Módulo de Autenticación
    post 'auth/register', to: 'authentication#register'
    post 'auth/login', to: 'authentication#login'
    get 'auth/me', to: 'authentication#me'

    # Módulo de Productos (CRUD completo para Administradores)
    resources :products, only: [:index, :show, :create, :update, :destroy]

    # Módulo del Carrito (Sincronizado en BD)
    resource :cart, only: [:show] do
      resources :items, controller: 'cart_items', only: [:create, :update, :destroy]
    end

    # Módulo de Pedidos e Historiales
    resources :orders, only: [:index, :show, :create]

    # Módulo de Soporte Pasivo (Chatbot)
    post 'chatbot', to: 'chatbot#process_message'
  end
end
```

---

## 🛠️ 4. Lógica de Negocio Desacoplada (Service Objects)

Con el fin de evitar modelos sobrecargados ("Fat Models") y controladores saturados de lógica procedimental, toda operación de negocio compleja se implementará en clases de servicio dedicadas bajo la ruta `app/services/`.

### 4.1. `CartMergeService` (Sincronización Híbrida de Carrito)
Ejecutado al iniciar sesión. Recibe el ID del usuario y un listado de ítems locales de `localStorage`.
*   **Lógica:**
    1.  Recuperar o inicializar el carrito en base de datos para el `user_id`.
    2.  Por cada ítem recibido del frontend:
        *   Si el producto ya existe en la base de datos, sumarizar las cantidades (respetando los límites de stock).
        *   Si no existe, asociarlo físicamente al registro.
    3.  Persistir los cambios en una única transacción de ActiveRecord.

### 4.2. `ProcessCheckoutService` (Procesamiento de Pedidos)
Servicio transaccional encargado de consolidar una orden de compra atómica a partir del carrito persistido.
*   **Lógica:**
    1.  Abrir un bloque de transacción SQL (`ActiveRecord::Base.transaction`).
    2.  Recuperar el carrito del usuario bloqueando las filas de stock de los productos incluidos en él.
    3.  Instanciar el servicio de stock (`StockManagementService`) para decrementar de manera segura las unidades físicas.
    4.  Calcular los montos exactos y crear el registro `Order`.
    5.  Duplicar los registros de `CartItem` hacia la tabla histórica `OrderItem`, capturando el `price_snapshot` del producto actual.
    6.  Eliminar físicamente los registros del carrito (`CartItem`).
    7.  Retornar el objeto `Order` completo. Si hay un fallo de stock, levantar excepción para hacer rollback inmediato.

---

## 🛡️ 5. Gestión de Concurrencia y Prevención de Venta Excesiva (Overselling)

Para evitar condiciones de carrera (*race conditions*) donde dos transacciones concurrentes lean simultáneamente la última unidad en stock de un producto y ambas completen la compra, el backend implementará un bloqueo pesimista mediante la instrucción `SELECT FOR UPDATE` de base de datos dentro de un bloque transaccional aislado de SQL.

### 5.1. Implementación del Servicio de Gestión de Stock

```ruby
# app/services/stock_management_service.rb
class StockManagementService
  class InsufficientStockError < StandardError; end

  # Ejecuta el decremento de stock atómico
  def self.decrement_stock!(product_id, requested_quantity)
    # Se bloquea la fila del producto específico con "FOR UPDATE" en la base de datos
    product = Product.lock("FOR UPDATE").find(product_id)

    if product.deleted_at.present?
      raise InsufficientStockError, "El producto #{product.name} ya no está disponible."
    end

    if product.stock < requested_quantity
      raise InsufficientStockError, "Stock insuficiente para #{product.name}. Disponible: #{product.stock}, Solicitado: #{requested_quantity}"
    end

    # Decremento atómico
    product.decrement!(:stock, requested_quantity)
  end
end
```

### 5.2. Bloque Transaccional de Consolidación de Pedidos

```ruby
# app/services/create_order_service.rb
class CreateOrderService
  def initialize(user)
    @user = user
  end

  def call
    ActiveRecord::Base.transaction do
      cart = @user.cart
      raise StandardError, "El carrito está vacío" if cart.nil? || cart.cart_items.empty?

      total_price = 0
      order_items_to_create = []

      # 1. Validar y descontar stock con bloqueo pesimista
      cart.cart_items.each do |item|
        product = item.product
        
        # Decrementa el stock de forma segura bloqueando la base de datos
        StockManagementService.decrement_stock!(product.id, item.quantity)
        
        # Calcular el monto unitario exacto
        item_total = product.price * item.quantity
        total_price += item_total

        # Generar snapshot histórico
        order_items_to_create << OrderItem.new(
          product_id: product.id,
          quantity: item.quantity,
          price_snapshot: product.price
        )
      end

      # 2. Crear la orden en estado 'pending'
      order = @user.orders.create!(
        status: 'pending',
        total_price: total_price
      )

      # 3. Guardar las líneas de pedido
      order_items_to_create.each do |order_item|
        order_item.order_id = order.id
        order_item.save!
      end

      # 4. Vaciar el carrito
      cart.cart_items.destroy_all

      # Retorno exitoso
      order
    end
  rescue StockManagementService::InsufficientStockError => e
    # Reraise para capturar en el controlador y responder con HTTP 422 Unprocessable Entity
    raise e
  rescue => e
    # Rollback automático al salir del bloque transaction
    raise StandardError, "Error al procesar la compra: #{e.message}"
  end
end
```

---

## 🔒 6. Seguridad Core y Capa de Autenticación sin Estado (JWT)

La autenticación de la API utilizará tokens criptográficos JWT firmados de forma segura con una clave secreta de Rails (`Rails.application.credentials.secret_key_base` o una variable de entorno `JWT_SECRET`).

### 6.1. Servicio de Codificación y Decodificación de Tokens

```ruby
# app/services/json_web_token.rb
class JsonWebToken
  SECRET_KEY = ENV['JWT_SECRET'] || Rails.application.credentials.secret_key_base || 'fallback_secret_key'

  def self.encode(payload, exp = 24.hours.from_now)
    payload[:exp] = exp.to_i
    JWT.encode(payload, SECRET_KEY, 'HS256')
  end

  def self.decode(token)
    decoded = JWT.decode(token, SECRET_KEY, true, { algorithm: 'HS256' })[0]
    HashWithIndifferentAccess.new decoded
  rescue JWT::DecodeError => e
    nil
  end
end
```

### 6.2. Autenticación y Control de Acceso en `ApplicationController`

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::API
  attr_reader :current_user

  protected

  # Verifica que la petición contenga un token JWT válido y carga @current_user
  def authenticate_user!
    header = request.headers['Authorization']
    header = header.split(' ').last if header.present?
    
    if header.present?
      decoded = JsonWebToken.decode(header)
      if decoded.present?
        @current_user = User.find_by(id: decoded[:user_id])
        return if @current_user.present?
      end
    end

    render json: { error: 'Unauthenticated', message: 'Debe iniciar sesión para continuar' }, status: :unauthorized
  end

  # Valida que el usuario autenticado posea el rol de administrador
  def authorize_admin!
    authenticate_user! unless @current_user.present?
    
    if @current_user.present? && @current_user.role != 'admin'
      render json: { error: 'Unauthorized', message: 'No tiene privilegios de administrador' }, status: :forbidden
    end
  end
end
```

---

## 🤖 7. Arquitectura Pasiva del Chatbot de Asistencia

Para cumplir con la arquitectura pasiva definida en el PRD y aislar responsabilidades de red, el chatbot se implementará en el controlador `api/chatbot_controller.rb`.

### Controlador del Chatbot

```ruby
# app/controllers/api/chatbot_controller.rb
class Api::ChatbotController < ApplicationController
  # Permite consultas anónimas y de usuarios registrados de manera segura
  def process_message
    message = params[:message]
    
    if message.blank?
      return render json: { error: 'BadRequest', message: 'El mensaje no puede estar vacío' }, status: :bad_request
    end

    # Delegación a un servicio de procesamiento de lenguaje natural local o mock
    reply = ChatbotAssistantService.call(message, current_user: @current_user)

    render json: { reply: reply }
  end
end
```

El servicio `ChatbotAssistantService` limpiará el input, procesará reglas semánticas básicas de NebriAmazon (ej. responder sobre stock de productos, estado de órdenes del `current_user` si está autenticado, o políticas de devolución) y devolverá un texto formateado libre de marcas HTML.

---

## 🚀 8. Roadmap de Desarrollo del Backend (6 Iteraciones Secuenciales)

De acuerdo al cronograma maestro, el desarrollo del servidor se estructurará de la siguiente manera:

### 📅 Iteración 1: Inicialización, Base de Datos e Ingestión PRD (Fase 1)
*   **Objetivos:** Crear el andamiaje del servidor y el diseño físico del esquema relacional de base de datos.
*   **Tareas:**
    1.  Inicializar el proyecto API ejecutando `rails new backend-nebri-amazon --api --database=postgresql`.
    2.  Configurar las dependencias iniciales en el `Gemfile`: `bcrypt`, `jwt`, `rack-cors`, e inicializar `rspec-rails`.
    3.  Habilitar la extensión de encriptación `pgcrypto` en base de datos.
    4.  Crear y ejecutar las migraciones iniciales para las tablas: `users`, `categories`, `products` (con índice parcial de Soft-Delete).
    5.  Implementar validaciones robustas y asociaciones en los modelos `User`, `Category` y `Product`.

### 📅 Iteración 2: Registro, Login e Infraestructura JWT (Fase 2)
*   **Objetivos:** Implementar un sistema robusto de autenticación sin estado basado en roles.
*   **Tareas:**
    1.  Configurar la capa de codificación y descodificación de tokens en `app/services/json_web_token.rb`.
    2.  Codificar los métodos de autenticación centralizados y roles (`authenticate_user!`, `authorize_admin!`) en `ApplicationController`.
    3.  Implementar el controlador de autenticación `api/authentication_controller.rb` con endpoints `/api/auth/register` y `/api/auth/login`.
    4.  Establecer rutas seguras y verificar perfiles en `/api/auth/me`.
    5.  Escribir pruebas de Request Specs en RSpec validando flujos correctos e incorrectos de login.

### 📅 Iteración 3: Catálogo Completo y CRUD de Administrador (Fase 3)
*   **Objetivos:** Exponer el catálogo general e implementar la consola de administración de stock.
*   **Tareas:**
    1.  Configurar controladores REST para productos `/api/products` admitiendo búsquedas semánticas y query params por categorías.
    2.  Implementar el control de eliminación lógica (*Soft Delete*) en el modelo `Product`.
    3.  Proteger endpoints de escritura (`create`, `update`, `destroy`) bajo el guardián de autorización `authorize_admin!`.
    4.  Optimizar las consultas del catálogo evitando problemas de consultas N+1 mediante `.includes(:category)`.
    5.  Ejecutar auditorías de planes de ejecución con `EXPLAIN ANALYZE` en consultas del catálogo.

### 📅 Iteración 4: Persistencia y Sincronización del Carrito (Fase 4)
*   **Objetivos:** Implementar la cesta persistente sincronizada para usuarios autenticados.
*   **Tareas:**
    1.  Codificar las migraciones y modelos para `carts` y `cart_items` (con restricciones únicas compuestas).
    2.  Implementar endpoints CRUD para ítems del carrito (`GET /api/cart`, `POST /api/cart/items`, `PUT /api/cart/items/:id`, `DELETE /api/cart/items/:id`).
    3.  Codificar `CartMergeService` para gestionar la unificación de carritos del cliente tras la autenticación.
    4.  Escribir pruebas en RSpec garantizando el cálculo correcto de desgloses y límites físicos de stock al actualizar la cesta.

### 📅 Iteración 5: Compra y Bloqueo de Concurrencia (Fase 5)
*   **Objetivos:** Asegurar la integridad de compras transaccionales concurrentes libre de overselling.
*   **Tareas:**
    1.  Crear migraciones y modelos estructurados para `orders` y `order_items` (con snapshots de precios).
    2.  Codificar `StockManagementService` implementando el bloqueo pesimista `SELECT FOR UPDATE` de PostgreSQL.
    3.  Desarrollar `CreateOrderService` encapsulando la confirmación de la compra, decremento atómico, vaciado del carrito y persistencia en bloques de transacciones SQL.
    4.  Implementar el controlador de pedidos `/api/orders` y proteger sus llamadas mediante `authenticate_user!`.

### 📅 Iteración 6: Chatbot de Soporte, QA y Certificación (Fase 6)
*   **Objetivos:** Añadir la capa de soporte virtual pasivo, auditoría completa SOLID y pruebas de concurrencia.
*   **Tareas:**
    1.  Implementar `ChatbotAssistantService` para procesar y responder semánticamente a consultas de productos y estados de pedidos.
    2.  Crear el endpoint pasivo `/api/chatbot` libre de CORS.
    3.  Codificar la prueba de concurrencia extrema en RSpec (`spec/services/create_order_service_spec.rb`) ejecutando hilos en paralelo para certificar la inmunidad al overselling.
    4.  Resolver problemas de calidad estructural de código mediante auditoría estricta de `code-inspector` logrando la calificación de **10.0** en base a Clean Code.

---

## 🏆 9. Criterios de Aceptación y QA (Definition of Done)

Para considerar certificado el backend de NebriAmazon y autorizar su despliegue final, el código deberá cumplir rigurosamente lo siguiente:

1.  **Cobertura de RSpec de 95% o superior:** La suite de pruebas debe asegurar todos los controladores de API (*Request Specs*), modelos (*Model Specs*), y los servicios transaccionales (*Service Specs*).
2.  **Inmunidad Probada contra Overselling:** Se debe contar con un caso de prueba concurrente robusto que simule la compra paralela del último artículo disponible por múltiples hilos concurrentes y verifique que solo una de las peticiones sea aceptada (`201 Created`), mientras que las demás fallen de manera controlada devolviendo un error HTTP `422 Unprocessable Entity`.
3.  **Seguridad contra Inyección SQL y XSS:** Queda prohibida la interpolación directa de parámetros del usuario en cadenas SQL. Toda consulta dinámica debe ser ejecutada mediante placeholders tipados de ActiveRecord.
4.  **Desacoplamiento Estricto:** La lógica de negocio no debe residir en controladores o en modelos de ActiveRecord. Cualquier cálculo financiero de órdenes o procesamiento de stock debe residir en servicios específicos testeables individualmente.
5.  **Calidad Estructural Limpia (SOLID):** El código debe pasar la revisión de arquitectura del `code-inspector` sin reportar problemas de acoplamiento, logrando una calificación final perfecta de **10.0**.
