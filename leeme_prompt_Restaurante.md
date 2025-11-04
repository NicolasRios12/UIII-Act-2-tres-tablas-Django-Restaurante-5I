
Perfecto — te entrego **todo** lo que pediste: procedimientos paso a paso y el código necesario para crear el proyecto Django **UIII_Restaurante_0676 / backend_Restaurante** y la app **app_Restaurante**, implementar CRUD para **Menu** (sin `forms.py`), plantillas Bootstrap, rutas, admin, migraciones y ejecución en el puerto **8076**. Todo en **español** y listo para copiar/pegar. No creo ningún documento: aquí está el contenido directo.

---

# 1 — Estructura inicial (qué crear)

Al inicio crea esta estructura (puedes crear carpetas manualmente o con comandos):

```
UIII_Restaurante_0676/
├─ .venv/                   (entorno virtual)
├─ backend_Restaurante/     (proyecto django)
│  ├─ backend_Restaurante/
│  │  ├─ __init__.py
│  │  ├─ settings.py
│  │  ├─ urls.py
│  │  └─ wsgi.py / asgi.py
│  └─ manage.py
└─ app_Restaurante/
   ├─ migrations/
   ├─ templates/
   │  ├─ base.html
   │  ├─ header.html
   │  ├─ navbar.html
   │  ├─ footer.html
   │  ├─ inicio.html
   │  └─ Menu/
   │     ├─ agregar_Menu.html
   │     ├─ ver_Menu.html
   │     ├─ actualizar_Menu.html
   │     └─ borrar_Menu.html
   ├─ __init__.py
   ├─ admin.py
   ├─ apps.py
   ├─ models.py
   ├─ views.py
   └─ urls.py
```

---

# 2 — Procedimientos paso a paso

### 1) Crear carpeta del proyecto

En tu explorador de archivos o terminal crea la carpeta:

```
mkdir UIII_Restaurante_0676
cd UIII_Restaurante_0676
```

### 2) Abrir VS Code sobre la carpeta

* Abre VS Code y selecciona *File → Open Folder...* y abre `UIII_Restaurante_0676`.
* O desde terminal (si tienes code en PATH):

```
code .
```

### 3) Abrir terminal en VS Code

* Menú: *View → Terminal*.
* O atajo: ``Ctrl+` `` (control + tilde).

### 4) Crear entorno virtual `.venv` desde terminal de VS Code

En la raíz `UIII_Restaurante_0676`:

* En Windows (PowerShell):

```
python -m venv .venv
```

* En macOS / Linux:

```
python3 -m venv .venv
```

### 5) Activar el entorno virtual

* Windows PowerShell:

```
.venv\Scripts\Activate.ps1
```

(si da error por ejecución: `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser` en PowerShell con permisos de administrador)

* Windows CMD:

```
.venv\Scripts\activate
```

* macOS / Linux:

```
source .venv/bin/activate
```

Verifica: `(.venv)` aparecerá en el prompt.

### 6) Activar intérprete de Python en VS Code

* En VS Code: `Ctrl+Shift+P` → `Python: Select Interpreter` → elige la ruta a `UIII_Restaurante_0676/.venv/.../python`.
* También puedes abrir un archivo .py y VS Code sugerirá seleccionar el intérprete.

### 7) Instalar Django

Con el entorno activo:

```
pip install django
```

(Verifica con `django-admin --version` o `python -m django --version`.)

### 8) Crear proyecto `backend_Restaurante` **sin duplicar carpeta**

Para evitar crear una carpeta extra, haz **esto dentro** de `UIII_Restaurante_0676`:

```
django-admin startproject backend_Restaurante .
```

**IMPORTANTE:** el punto (`.`) al final crea el proyecto dentro de la carpeta actual sin generar otra carpeta contenedora.

### 9) Ejecutar servidor en el puerto 8076

Después de crear proyecto y app (ver abajo):

```
python manage.py runserver 8076
```

El servidor arrancará en `http://127.0.0.1:8076/`.

### 10) Copiar y pegar el link en el navegador

Abre el navegador y pega: `http://127.0.0.1:8076/`

### 11) Crear aplicación `app_Restaurante`

Desde la raíz del proyecto:

```
python manage.py startapp app_Restaurante
```

### 12) Aquí el `models.py`

Ya lo compartiste; úsalo tal cual en `app_Restaurante/models.py` (solo trabajaremos por ahora con `Menu`).

> Modelo que pegaste (asegúrate de colocarlo en `app_Restaurante/models.py`):

```python
from django.db import models

class Menu(models.Model):
    nombre = models.CharField(max_length=100)
    descripcion = models.TextField()
    precio = models.DecimalField(max_digits=6, decimal_places=2)
    categoria = models.CharField(max_length=50)
    disponible = models.BooleanField(default=True)
    fecha_creacion = models.DateTimeField(auto_now_add=True)
    imagen = models.URLField(blank=True, null=True)

    def __str__(self):
        return self.nombre

class Pedido(models.Model):
    menu = models.ForeignKey(Menu, on_delete=models.CASCADE, related_name='pedidos')
    nombre_cliente = models.CharField(max_length=100)
    cantidad = models.PositiveIntegerField()
    total = models.DecimalField(max_digits=8, decimal_places=2)
    metodo_pago = models.CharField(max_length=50)
    fecha_pedido = models.DateTimeField(auto_now_add=True)
    entregado = models.BooleanField(default=False)

    def __str__(self):
        return f"Pedido #{self.id} - {self.nombre_cliente}"

class Reservacion(models.Model):
    nombre_cliente = models.CharField(max_length=100)
    correo = models.EmailField()
    telefono = models.CharField(max_length=15)
    fecha_reservacion = models.DateField()
    hora = models.TimeField()
    numero_personas = models.PositiveIntegerField()
    menus = models.ManyToManyField(Menu, related_name='reservaciones')
    notas = models.TextField(blank=True, null=True)

    def __str__(self):
        return f"Reservación de {self.nombre_cliente} ({self.fecha_reservacion})"
```

### 12.5) Realizar las migraciones (makemigrations y migrate)

```
python manage.py makemigrations
python manage.py migrate
```

### 13) Primero trabajamos con el MODELO: MENU (lo haremos en views y plantillas)

---

# 3 — Código: `views.py` (CRUD para Menu, sin forms.py)

Coloca este `views.py` en `app_Restaurante/views.py`:

```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Menu
from django.urls import reverse

def inicio_Restaurante(request):
    return render(request, 'inicio.html')

# Crear / Agregar Menu
def agregar_Menu(request):
    if request.method == 'POST':
        nombre = request.POST.get('nombre')
        descripcion = request.POST.get('descripcion')
        precio = request.POST.get('precio') or 0
        categoria = request.POST.get('categoria')
        disponible = True if request.POST.get('disponible') == 'on' else False
        imagen = request.POST.get('imagen')
        Menu.objects.create(
            nombre=nombre,
            descripcion=descripcion,
            precio=precio,
            categoria=categoria,
            disponible=disponible,
            imagen=imagen
        )
        return redirect('ver_Menu')
    return render(request, 'Menu/agregar_Menu.html')

# Ver todos los menus
def ver_Menu(request):
    menus = Menu.objects.all().order_by('-fecha_creacion')
    return render(request, 'Menu/ver_Menu.html', {'menus': menus})

# Form para actualizar (carga datos)
def actualizar_Menu(request, menu_id):
    menu = get_object_or_404(Menu, id=menu_id)
    return render(request, 'Menu/actualizar_Menu.html', {'menu': menu})

# Procesa la actualización
def realizar_actualizacion_Menu(request, menu_id):
    menu = get_object_or_404(Menu, id=menu_id)
    if request.method == 'POST':
        menu.nombre = request.POST.get('nombre')
        menu.descripcion = request.POST.get('descripcion')
        menu.precio = request.POST.get('precio') or 0
        menu.categoria = request.POST.get('categoria')
        menu.disponible = True if request.POST.get('disponible') == 'on' else False
        menu.imagen = request.POST.get('imagen')
        menu.save()
        return redirect('ver_Menu')
    return redirect('actualizar_Menu', menu_id=menu.id)

# Confirmar borrado
def borrar_Menu(request, menu_id):
    menu = get_object_or_404(Menu, id=menu_id)
    if request.method == 'POST':
        menu.delete()
        return redirect('ver_Menu')
    return render(request, 'Menu/borrar_Menu.html', {'menu': menu})
```

---

# 4 — Templates (Bootstrap + archivos solicitados)

Crea carpeta `app_Restaurante/templates/` y dentro los archivos.

### `base.html`

```html
<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{% block title %}Restaurante{% endblock %}</title>
    <!-- Bootstrap CSS (CDN) -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
      /* Colores suaves y moderno */
      body { background-color: #f7fafc; color:#2d3748; }
      .footer { position: fixed; bottom: 0; left: 0; right: 0; background:#ffffff; border-top:1px solid #e2e8f0; padding:10px 0; }
      .card { border-radius:12px; box-shadow: 0 2px 6px rgba(16,24,40,0.04); }
    </style>
    {% block extra_head %}{% endblock %}
  </head>
  <body>
    {% include 'header.html' %}
    <div class="container-fluid">
      <div class="row">
        <nav class="col-md-2 d-none d-md-block bg-light sidebar p-3">
          {% include 'navbar.html' %}
        </nav>
        <main class="col-md-9 ms-sm-auto col-lg-10 px-md-4 py-4">
          {% block content %}{% endblock %}
        </main>
      </div>
    </div>

    {% include 'footer.html' %}

    <!-- Bootstrap JS (CDN) -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
    {% block extra_js %}{% endblock %}
  </body>
</html>
```

### `header.html`

```html
<header class="py-3 mb-4 border-bottom bg-white">
  <div class="container d-flex flex-wrap justify-content-between align-items-center">
    <a href="{% url 'inicio_Restaurante' %}" class="d-flex align-items-center text-dark text-decoration-none">
      <span class="fs-4 fw-bold">Sistema de Administración Restaurante</span>
    </a>
    <div class="text-muted">Bienvenido</div>
  </div>
</header>
```

### `navbar.html`

```html
<ul class="nav flex-column">
  <li class="nav-item mb-2">
    <a class="nav-link d-flex align-items-center" href="{% url 'inicio_Restaurante' %}">
      🏠 Inicio
    </a>
  </li>

  <li class="nav-item mb-1">
    <a class="nav-link d-flex align-items-center" data-bs-toggle="collapse" href="#menuSub" role="button" aria-expanded="false">
      📋 Menu
    </a>
    <div class="collapse" id="menuSub">
      <ul class="nav flex-column ms-3">
        <li><a class="nav-link" href="{% url 'agregar_Menu' %}">Agregar Menu</a></li>
        <li><a class="nav-link" href="{% url 'ver_Menu' %}">Ver Menu</a></li>
      </ul>
    </div>
  </li>

  <li class="nav-item mb-1">
    <a class="nav-link d-flex align-items-center" data-bs-toggle="collapse" href="#pedidosSub">
      🧾 Pedidos
    </a>
    <div class="collapse" id="pedidosSub">
      <ul class="nav flex-column ms-3">
        <li><span class="nav-link disabled">Agregar Pedidos</span></li>
        <li><span class="nav-link disabled">Ver Pedidos</span></li>
      </ul>
    </div>
  </li>

  <li class="nav-item mb-1">
    <a class="nav-link d-flex align-items-center" data-bs-toggle="collapse" href="#reservSub">
      📅 Reservaciones
    </a>
    <div class="collapse" id="reservSub">
      <ul class="nav flex-column ms-3">
        <li><span class="nav-link disabled">Agregar Reservaciones</span></li>
        <li><span class="nav-link disabled">Ver Reservaciones</span></li>
      </ul>
    </div>
  </li>
</ul>
```

> Nota: los submenús usan `collapse` de Bootstrap. Los íconos están en el texto principal, no en subitems como pediste.

### `footer.html`

```html
<footer class="footer text-center">
  <div class="container">
    <small>
      &copy; {{ now.year }} Sistema Restaurante • Creado por Nicolas Rios, Cbtis 128
    </small>
  </div>
</footer>
```

> Para que `now` funcione, habilita el contexto en settings o usa `django.template.context_processors.request`. Alternativamente cambia a `{{ ""|date:"Y" }}`:

```html
&copy; {{ ""|date:"Y" }} ...
```

### `inicio.html`

```html
{% extends 'base.html' %}
{% block title %}Inicio - Restaurante{% endblock %}
{% block content %}
  <div class="container">
    <div class="card p-4 mb-4">
      <h2>Bienvenido al Sistema de Administración del Restaurante</h2>
      <p>Este sistema permite gestionar el Menú, Pedidos y Reservaciones.</p>
      <img src="https://upload.wikimedia.org/wikipedia/commons/4/45/Cinepolis_logo.png" alt="Cinepolis" class="img-fluid" style="max-width:300px;">
    </div>
  </div>
{% endblock %}
```

> La imagen la tomé de la red (ejemplo). Cambia la URL si deseas otra.

---

# 5 — Templates `Menu` (CRUD)

Crea `app_Restaurante/templates/Menu/` y los archivos:

### `agregar_Menu.html`

```html
{% extends 'base.html' %}
{% block content %}
<div class="card p-4">
  <h3>Agregar Menu</h3>
  <form method="post">
    {% csrf_token %}
    <div class="mb-3">
      <label class="form-label">Nombre</label>
      <input class="form-control" name="nombre" required>
    </div>
    <div class="mb-3">
      <label class="form-label">Descripción</label>
      <textarea class="form-control" name="descripcion"></textarea>
    </div>
    <div class="mb-3">
      <label class="form-label">Precio</label>
      <input type="number" step="0.01" class="form-control" name="precio">
    </div>
    <div class="mb-3">
      <label class="form-label">Categoría</label>
      <input class="form-control" name="categoria">
    </div>
    <div class="form-check mb-3">
      <input class="form-check-input" type="checkbox" id="disponible" name="disponible">
      <label class="form-check-label" for="disponible">Disponible</label>
    </div>
    <div class="mb-3">
      <label class="form-label">URL imagen</label>
      <input class="form-control" name="imagen">
    </div>
    <button class="btn btn-primary">Agregar</button>
  </form>
</div>
{% endblock %}
```

### `ver_Menu.html`

```html
{% extends 'base.html' %}
{% block content %}
<div class="card p-3">
  <h3>Lista de Menús</h3>
  <table class="table table-hover">
    <thead>
      <tr>
        <th>Nombre</th><th>Categoría</th><th>Precio</th><th>Disponible</th><th>Acciones</th>
      </tr>
    </thead>
    <tbody>
      {% for menu in menus %}
      <tr>
        <td>{{ menu.nombre }}</td>
        <td>{{ menu.categoria }}</td>
        <td>{{ menu.precio }}</td>
        <td>{% if menu.disponible %}Sí{% else %}No{% endif %}</td>
        <td>
          <a class="btn btn-sm btn-info" href="{% url 'actualizar_Menu' menu.id %}">Editar</a>
          <a class="btn btn-sm btn-danger" href="{% url 'borrar_Menu' menu.id %}">Borrar</a>
        </td>
      </tr>
      {% empty %}
      <tr><td colspan="5">No hay menús aún.</td></tr>
      {% endfor %}
    </tbody>
  </table>
</div>
{% endblock %}
```

### `actualizar_Menu.html`

```html
{% extends 'base.html' %}
{% block content %}
<div class="card p-4">
  <h3>Actualizar Menu</h3>
  <form method="post" action="{% url 'realizar_actualizacion_Menu' menu.id %}">
    {% csrf_token %}
    <div class="mb-3">
      <label class="form-label">Nombre</label>
      <input class="form-control" name="nombre" value="{{ menu.nombre }}">
    </div>
    <div class="mb-3">
      <label class="form-label">Descripción</label>
      <textarea class="form-control" name="descripcion">{{ menu.descripcion }}</textarea>
    </div>
    <div class="mb-3">
      <label class="form-label">Precio</label>
      <input type="number" step="0.01" class="form-control" name="precio" value="{{ menu.precio }}">
    </div>
    <div class="mb-3">
      <label class="form-label">Categoría</label>
      <input class="form-control" name="categoria" value="{{ menu.categoria }}">
    </div>
    <div class="form-check mb-3">
      <input class="form-check-input" type="checkbox" id="disponible" name="disponible" {% if menu.disponible %}checked{% endif %}>
      <label class="form-check-label" for="disponible">Disponible</label>
    </div>
    <div class="mb-3">
      <label class="form-label">URL imagen</label>
      <input class="form-control" name="imagen" value="{{ menu.imagen }}">
    </div>
    <button class="btn btn-success">Guardar</button>
    <a class="btn btn-secondary" href="{% url 'ver_Menu' %}">Cancelar</a>
  </form>
</div>
{% endblock %}
```

### `borrar_Menu.html`

```html
{% extends 'base.html' %}
{% block content %}
<div class="card p-4">
  <h3>Confirmar Borrado</h3>
  <p>¿Deseas borrar el menú <strong>{{ menu.nombre }}</strong>?</p>
  <form method="post">
    {% csrf_token %}
    <button type="submit" class="btn btn-danger">Sí, borrar</button>
    <a class="btn btn-secondary" href="{% url 'ver_Menu' %}">Cancelar</a>
  </form>
</div>
{% endblock %}
```

---

# 6 — `urls.py` en la app y en el proyecto

### `app_Restaurante/urls.py` (crear)

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_Restaurante, name='inicio_Restaurante'),
    path('menu/agregar/', views.agregar_Menu, name='agregar_Menu'),
    path('menu/', views.ver_Menu, name='ver_Menu'),
    path('menu/actualizar/<int:menu_id>/', views.actualizar_Menu, name='actualizar_Menu'),
    path('menu/realizar_actualizacion/<int:menu_id>/', views.realizar_actualizacion_Menu, name='realizar_actualizacion_Menu'),
    path('menu/borrar/<int:menu_id>/', views.borrar_Menu, name='borrar_Menu'),
]
```

### `backend_Restaurante/urls.py` (vincular la app)

Abre el `urls.py` del proyecto y modifícalo así:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Restaurante.urls')),
]
```

---

# 7 — Agregar `app_Restaurante` en `settings.py`

En `backend_Restaurante/settings.py` añade la app en `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    # apps default...
    'django.contrib.admin',
    'django.contrib.auth',
    # ...
    'app_Restaurante',
]
```

Asegúrate también de tener `'django.template.context_processors.request'` en `TEMPLATES` (por defecto está).

---

# 8 — Registrar modelos en `admin.py` y volver a migrar

`app_Restaurante/admin.py`:

```python
from django.contrib import admin
from .models import Menu, Pedido, Reservacion

@admin.register(Menu)
class MenuAdmin(admin.ModelAdmin):
    list_display = ('id', 'nombre', 'categoria', 'precio', 'disponible', 'fecha_creacion')
    search_fields = ('nombre', 'categoria')

@admin.register(Pedido)
class PedidoAdmin(admin.ModelAdmin):
    list_display = ('id','menu','nombre_cliente','cantidad','total','fecha_pedido')

@admin.register(Reservacion)
class ReservacionAdmin(admin.ModelAdmin):
    list_display = ('id','nombre_cliente','fecha_reservacion','hora','numero_personas')
```

Después:

```
python manage.py makemigrations app_Restaurante
python manage.py migrate
```

Crea superusuario para acceder a admin:

```
python manage.py createsuperuser
```

---

# 9 — Otras configuraciones recomendadas (rápidas)

* En `settings.py` añade `TIME_ZONE = 'America/Ciudad_Juarez'` si quieres la zona horaria que mencionaste.
* Para servir archivos estáticos en desarrollo no es necesario ahora; Bootstrap viene por CDN.
* Si la footer fija cubre contenido, añade `body { padding-bottom: 60px; }` en `base.html` CSS.

---

# 10 — Crear la estructura completa de carpetas al inicio (comando rápido)

Si quieres crear archivos vacíos por consola (opcional):

```bash
mkdir -p app_Restaurante/templates/Menu
touch app_Restaurante/__init__.py app_Restaurante/admin.py app_Restaurante/apps.py app_Restaurante/models.py app_Restaurante/views.py app_Restaurante/urls.py
touch app_Restaurante/templates/base.html app_Restaurante/templates/header.html app_Restaurante/templates/navbar.html app_Restaurante/templates/footer.html app_Restaurante/templates/inicio.html
touch app_Restaurante/templates/Menu/agregar_Menu.html app_Restaurante/templates/Menu/ver_Menu.html app_Restaurante/templates/Menu/actualizar_Menu.html app_Restaurante/templates/Menu/borrar_Menu.html
```

Luego llena los archivos con el código proporcionado.

---

# 11 — Finalmente: ejecutar servidor en puerto 8076

Con el entorno activo y todo migrado:

```
python manage.py runserver 8076
```

Abre `http://127.0.0.1:8076/`.

---

# 12 — Resumen rápido de comandos en orden (copiar/pegar)

```bash
cd /ruta/a/UIII_Restaurante_0676
python -m venv .venv
# activar .venv (según tu OS)
.venv\Scripts\activate   # Windows CMD
# o
source .venv/bin/activate  # mac/linux

pip install django

# crear proyecto sin duplicar carpeta
django-admin startproject backend_Restaurante .

# crear app
python manage.py startapp app_Restaurante

# editar archivos: models.py (pega tu modelo), views.py (pega código), templates (crea archivos), urls.py app, settings.py (agrega app)
# luego migraciones
python manage.py makemigrations
python manage.py migrate

# crear superusuario (opcional)
python manage.py createsuperuser

# ejecutar en puerto 8076
python manage.py runserver 8076
```

---

# 13 — Notas y restricciones que pediste

* **No se usó `forms.py`**: las vistas manejan `request.POST` directamente.
* **Por ahora solo trabajamos con `Menu`**; `Pedido` y `Reservacion` quedan pendientes.
* **No validar entrada de datos** (tal como pediste).
* **Colores suaves y diseño sencillo**: CSS embebido en `base.html`.
* **Plantillas separadas** (`base`, `header`, `navbar`, `footer`, `inicio`, y la carpeta `Menu/`).

---

Si quieres, puedo ahora:

* Generar todo el código listo para pegar (por ejemplo `views.py`, `urls.py`, `admin.py`, y cada template) en bloques separados para que copies exactamente (ya lo incluí, pero puedo entregártelos como archivos listos).
* O darte instrucciones para añadir estilos CSS extra, íconos (fontawesome), o manejo de imágenes subidas (Media) — aunque pediste URL para imagen, por eso usamos `URLField`.

¿Te entrego los archivos listos para copiar (uno por uno) o prefieres que haga algún ajuste en el diseño o en rutas?
