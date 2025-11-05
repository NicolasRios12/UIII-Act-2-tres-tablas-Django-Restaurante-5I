

---

# — Estructura inicial (qué crear)

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

# Guía Completa: Proyecto Restaurante con Django

## 1. Crear Carpeta del Proyecto
```bash
mkdir UIII_Restaurante_0676
```

## 2. Abrir VS Code en la Carpeta
```bash
cd UIII_Restaurante_0676
code .
```

## 3. Abrir Terminal en VS Code
- `Ctrl + ù` (Windows/Linux) o `Ctrl + Shift + ù` (Mac)
- O menú: View → Terminal

## 4. Crear Entorno Virtual
```bash
python -m venv .venv
```

## 5. Activar Entorno Virtual
**Windows:**
```bash
.venv\Scripts\activate
```
**Mac/Linux:**
```bash
source .venv/bin/activate
```

## 6. Activar Intérprete de Python
- `Ctrl + Shift + P`
- Buscar: "Python: Select Interpreter"
- Seleccionar: `./.venv/Scripts/python.exe`

## 7. Instalar Django
```bash
pip install django
```

## 8. Crear Proyecto Django
```bash
django-admin startproject backend_Restaurante .
```

## 9. Ejecutar Servidor en Puerto 8076
```bash
python manage.py runserver 8076
```

## 10. Verificar en Navegador
- Abrir: `http://localhost:8076`

## 11. Crear Aplicación
```bash
python manage.py startapp app_Restaurante
```

## 12. Configurar models.py
En `app_Restaurante/models.py`:

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

## 12.5 Realizar Migraciones
```bash
python manage.py makemigrations
python manage.py migrate
```

## 13. Configurar Views (Solo Menu)
En `app_Restaurante/views.py`:

```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Menu

def inicio_Restaurante(request):
    return render(request, 'inicio.html')

def agregar_Menu(request):
    if request.method == 'POST':
        Menu.objects.create(
            nombre=request.POST['nombre'],
            descripcion=request.POST['descripcion'],
            precio=request.POST['precio'],
            categoria=request.POST['categoria'],
            disponible=request.POST.get('disponible', False),
            imagen=request.POST['imagen']
        )
        return redirect('ver_Menu')
    return render(request, 'Menu/agregar_Menu.html')

def ver_Menu(request):
    menus = Menu.objects.all()
    return render(request, 'Menu/ver_Menu.html', {'menus': menus})

def actualizar_Menu(request, id):
    menu = get_object_or_404(Menu, id=id)
    return render(request, 'Menu/actualizar_Menu.html', {'menu': menu})

def realizar_actualizacion_Menu(request, id):
    if request.method == 'POST':
        menu = get_object_or_404(Menu, id=id)
        menu.nombre = request.POST['nombre']
        menu.descripcion = request.POST['descripcion']
        menu.precio = request.POST['precio']
        menu.categoria = request.POST['categoria']
        menu.disponible = request.POST.get('disponible', False)
        menu.imagen = request.POST['imagen']
        menu.save()
        return redirect('ver_Menu')
    return redirect('ver_Menu')

def borrar_Menu(request, id):
    menu = get_object_or_404(Menu, id=id)
    if request.method == 'POST':
        menu.delete()
        return redirect('ver_Menu')
    return render(request, 'Menu/borrar_Menu.html', {'menu': menu})
```

## 15-22. Estructura de Templates

### Crear carpetas:
```
app_Restaurante/
└── templates/
    ├── base.html
    ├── header.html
    ├── navbar.html
    ├── footer.html
    ├── inicio.html
    └── Menu/
        ├── agregar_Menu.html
        ├── ver_Menu.html
        ├── actualizar_Menu.html
        └── borrar_Menu.html
```

### base.html
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistema Restaurante</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        body {
            background-color: #f8f9fa;
            padding-top: 70px;
        }
        .navbar-brand {
            font-weight: bold;
        }
        .footer {
            background-color: #343a40;
            color: white;
            padding: 20px 0;
            margin-top: 50px;
        }
        .card {
            border: none;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            margin-bottom: 20px;
        }
        .btn-primary {
            background-color: #4e73df;
            border-color: #4e73df;
        }
    </style>
</head>
<body>
    {% include 'header.html' %}
    {% include 'navbar.html' %}
    
    <div class="container mt-4">
        {% block content %}
        {% endblock %}
    </div>
    
    {% include 'footer.html' %}
    
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

### navbar.html
```html
<nav class="navbar navbar-expand-lg navbar-dark bg-dark fixed-top">
    <div class="container">
        <a class="navbar-brand" href="{% url 'inicio_Restaurante' %}">
            <i class="fas fa-utensils"></i> Sistema de Administración Restaurante
        </a>
        
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
            <span class="navbar-toggler-icon"></span>
        </button>
        
        <div class="collapse navbar-collapse" id="navbarNav">
            <ul class="navbar-nav ms-auto">
                <li class="nav-item">
                    <a class="nav-link" href="{% url 'inicio_Restaurante' %}">
                        <i class="fas fa-home"></i> Inicio
                    </a>
                </li>
                
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" id="menuDropdown" role="button" data-bs-toggle="dropdown">
                        <i class="fas fa-book"></i> Menu
                    </a>
                    <ul class="dropdown-menu">
                        <li><a class="dropdown-item" href="{% url 'agregar_Menu' %}">Agregar Menu</a></li>
                        <li><a class="dropdown-item" href="{% url 'ver_Menu' %}">Ver Menu</a></li>
                        <li><a class="dropdown-item" href="{% url 'ver_Menu' %}">Actualizar Menu</a></li>
                        <li><a class="dropdown-item" href="{% url 'ver_Menu' %}">Borrar Menu</a></li>
                    </ul>
                </li>
                
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" id="pedidosDropdown" role="button" data-bs-toggle="dropdown">
                        <i class="fas fa-shopping-cart"></i> Pedidos
                    </a>
                    <ul class="dropdown-menu">
                        <li><a class="dropdown-item" href="#">Agregar Pedidos</a></li>
                        <li><a class="dropdown-item" href="#">Ver Pedidos</a></li>
                        <li><a class="dropdown-item" href="#">Actualizar Pedidos</a></li>
                        <li><a class="dropdown-item" href="#">Borrar Pedidos</a></li>
                    </ul>
                </li>
                
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" id="reservacionesDropdown" role="button" data-bs-toggle="dropdown">
                        <i class="fas fa-calendar-alt"></i> Reservaciones
                    </a>
                    <ul class="dropdown-menu">
                        <li><a class="dropdown-item" href="#">Agregar Reservaciones</a></li>
                        <li><a class="dropdown-item" href="#">Ver Reservaciones</a></li>
                        <li><a class="dropdown-item" href="#">Actualizar Reservaciones</a></li>
                        <li><a class="dropdown-item" href="#">Borrar Reservaciones</a></li>
                    </ul>
                </li>
            </ul>
        </div>
    </div>
</nav>
```

### footer.html
```html
<footer class="footer mt-auto py-3">
    <div class="container text-center">
        <p class="mb-1">&copy; {% now "Y" %} Sistema Restaurante - Todos los derechos reservados</p>
        <p class="mb-0">Fecha del sistema: {% now "d/m/Y H:i" %}</p>
        <p class="mb-0">Creado por Nicolas Rios, Cbtis 128</p>
    </div>
</footer>
```

### inicio.html
```html
{% extends 'base.html' %}

{% block content %}
<div class="row">
    <div class="col-md-8 mx-auto text-center">
        <h1 class="mb-4">Bienvenido al Sistema de Restaurante</h1>
        <p class="lead">Sistema de administración para gestionar menús, pedidos y reservaciones.</p>
        
        <div class="card mt-4">
            <div class="card-body">
                <img src="https://via.placeholder.com/800x400/4e73df/ffffff?text=Cinepolis+Restaurant" 
                     class="img-fluid rounded" alt="Cinepolis Restaurant">
            </div>
        </div>
        
        <div class="row mt-4">
            <div class="col-md-4">
                <div class="card">
                    <div class="card-body">
                        <i class="fas fa-book fa-3x text-primary mb-3"></i>
                        <h5>Gestión de Menús</h5>
                        <p>Administra el menú del restaurante</p>
                    </div>
                </div>
            </div>
            <div class="col-md-4">
                <div class="card">
                    <div class="card-body">
                        <i class="fas fa-shopping-cart fa-3x text-success mb-3"></i>
                        <h5>Control de Pedidos</h5>
                        <p>Gestiona pedidos de clientes</p>
                    </div>
                </div>
            </div>
            <div class="col-md-4">
                <div class="card">
                    <div class="card-body">
                        <i class="fas fa-calendar-alt fa-3x text-warning mb-3"></i>
                        <h5>Reservaciones</h5>
                        <p>Administra reservas de mesas</p>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

### Menu/agregar_Menu.html
```html
{% extends 'base.html' %}

{% block content %}
<div class="row">
    <div class="col-md-8 mx-auto">
        <div class="card">
            <div class="card-header">
                <h4 class="card-title mb-0">
                    <i class="fas fa-plus-circle"></i> Agregar Nuevo Menú
                </h4>
            </div>
            <div class="card-body">
                <form method="POST">
                    {% csrf_token %}
                    <div class="mb-3">
                        <label class="form-label">Nombre</label>
                        <input type="text" class="form-control" name="nombre" required>
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Descripción</label>
                        <textarea class="form-control" name="descripcion" rows="3" required></textarea>
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Precio</label>
                        <input type="number" step="0.01" class="form-control" name="precio" required>
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Categoría</label>
                        <input type="text" class="form-control" name="categoria" required>
                    </div>
                    <div class="mb-3 form-check">
                        <input type="checkbox" class="form-check-input" name="disponible" value="True">
                        <label class="form-check-label">Disponible</label>
                    </div>
                    <div class="mb-3">
                        <label class="form-label">URL de Imagen</label>
                        <input type="url" class="form-control" name="imagen">
                    </div>
                    <button type="submit" class="btn btn-primary">Guardar Menú</button>
                    <a href="{% url 'ver_Menu' %}" class="btn btn-secondary">Cancelar</a>
                </form>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

### Menu/ver_Menu.html
```html
{% extends 'base.html' %}

{% block content %}
<div class="card">
    <div class="card-header d-flex justify-content-between align-items-center">
        <h4 class="card-title mb-0">
            <i class="fas fa-list"></i> Lista de Menús
        </h4>
        <a href="{% url 'agregar_Menu' %}" class="btn btn-primary">
            <i class="fas fa-plus"></i> Agregar Menú
        </a>
    </div>
    <div class="card-body">
        <div class="table-responsive">
            <table class="table table-striped table-hover">
                <thead class="table-dark">
                    <tr>
                        <th>ID</th>
                        <th>Nombre</th>
                        <th>Precio</th>
                        <th>Categoría</th>
                        <th>Disponible</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    {% for menu in menus %}
                    <tr>
                        <td>{{ menu.id }}</td>
                        <td>{{ menu.nombre }}</td>
                        <td>${{ menu.precio }}</td>
                        <td>{{ menu.categoria }}</td>
                        <td>
                            {% if menu.disponible %}
                                <span class="badge bg-success">Sí</span>
                            {% else %}
                                <span class="badge bg-danger">No</span>
                            {% endif %}
                        </td>
                        <td>
                            <a href="{% url 'actualizar_Menu' menu.id %}" class="btn btn-warning btn-sm">
                                <i class="fas fa-edit"></i> Editar
                            </a>
                            <a href="{% url 'borrar_Menu' menu.id %}" class="btn btn-danger btn-sm">
                                <i class="fas fa-trash"></i> Borrar
                            </a>
                        </td>
                    </tr>
                    {% empty %}
                    <tr>
                        <td colspan="6" class="text-center">No hay menús registrados</td>
                    </tr>
                    {% endfor %}
                </tbody>
            </table>
        </div>
    </div>
</div>
{% endblock %}
```

### Menu/actualizar_Menu.html
```html
{% extends 'base.html' %}

{% block content %}
<div class="row">
    <div class="col-md-8 mx-auto">
        <div class="card">
            <div class="card-header">
                <h4 class="card-title mb-0">
                    <i class="fas fa-edit"></i> Actualizar Menú: {{ menu.nombre }}
                </h4>
            </div>
            <div class="card-body">
                <form method="POST" action="{% url 'realizar_actualizacion_Menu' menu.id %}">
                    {% csrf_token %}
                    <div class="mb-3">
                        <label class="form-label">Nombre</label>
                        <input type="text" class="form-control" name="nombre" value="{{ menu.nombre }}" required>
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Descripción</label>
                        <textarea class="form-control" name="descripcion" rows="3" required>{{ menu.descripcion }}</textarea>
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Precio</label>
                        <input type="number" step="0.01" class="form-control" name="precio" value="{{ menu.precio }}" required>
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Categoría</label>
                        <input type="text" class="form-control" name="categoria" value="{{ menu.categoria }}" required>
                    </div>
                    <div class="mb-3 form-check">
                        <input type="checkbox" class="form-check-input" name="disponible" value="True" 
                               {% if menu.disponible %}checked{% endif %}>
                        <label class="form-check-label">Disponible</label>
                    </div>
                    <div class="mb-3">
                        <label class="form-label">URL de Imagen</label>
                        <input type="url" class="form-control" name="imagen" value="{{ menu.imagen|default:'' }}">
                    </div>
                    <button type="submit" class="btn btn-primary">Actualizar Menú</button>
                    <a href="{% url 'ver_Menu' %}" class="btn btn-secondary">Cancelar</a>
                </form>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

### Menu/borrar_Menu.html
```html
{% extends 'base.html' %}

{% block content %}
<div class="row">
    <div class="col-md-6 mx-auto">
        <div class="card">
            <div class="card-header">
                <h4 class="card-title mb-0 text-danger">
                    <i class="fas fa-exclamation-triangle"></i> Confirmar Eliminación
                </h4>
            </div>
            <div class="card-body text-center">
                <p class="lead">¿Estás seguro de que deseas eliminar el menú?</p>
                <h5 class="text-primary">{{ menu.nombre }}</h5>
                <p class="text-muted">Precio: ${{ menu.precio }}</p>
                
                <form method="POST">
                    {% csrf_token %}
                    <button type="submit" class="btn btn-danger">
                        <i class="fas fa-trash"></i> Sí, Eliminar
                    </button>
                    <a href="{% url 'ver_Menu' %}" class="btn btn-secondary">Cancelar</a>
                </form>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

## 24. Configurar URLs de la App
En `app_Restaurante/urls.py` (crear archivo):

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_Restaurante, name='inicio_Restaurante'),
    path('agregar-menu/', views.agregar_Menu, name='agregar_Menu'),
    path('ver-menu/', views.ver_Menu, name='ver_Menu'),
    path('actualizar-menu/<int:id>/', views.actualizar_Menu, name='actualizar_Menu'),
    path('realizar-actualizacion-menu/<int:id>/', views.realizar_actualizacion_Menu, name='realizar_actualizacion_Menu'),
    path('borrar-menu/<int:id>/', views.borrar_Menu, name='borrar_Menu'),
]
```

## 25. Agregar App en Settings
En `backend_Restaurante/settings.py`:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app_Restaurante',  # Agregar esta línea
]
```

## 26. Configurar URLs del Proyecto
En `backend_Restaurante/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Restaurante.urls')),
]
```

## 27. Registrar Modelos en Admin
En `app_Restaurante/admin.py`:

```python
from django.contrib import admin
from .models import Menu, Pedido, Reservacion

@admin.register(Menu)
class MenuAdmin(admin.ModelAdmin):
    list_display = ['nombre', 'precio', 'categoria', 'disponible', 'fecha_creacion']
    list_filter = ['categoria', 'disponible']
    search_fields = ['nombre', 'descripcion']

@admin.register(Pedido)
class PedidoAdmin(admin.ModelAdmin):
    list_display = ['id', 'nombre_cliente', 'menu', 'cantidad', 'total', 'entregado']
    list_filter = ['entregado', 'metodo_pago', 'fecha_pedido']

@admin.register(Reservacion)
class ReservacionAdmin(admin.ModelAdmin):
    list_display = ['nombre_cliente', 'correo', 'fecha_reservacion', 'hora', 'numero_personas']
    list_filter = ['fecha_reservacion']
```

## 28. Realizar Migraciones Finales
```bash
python manage.py makemigrations
python manage.py migrate
```

## 29. Crear Superusuario (Opcional)
```bash
python manage.py createsuperuser
```

## 30. Ejecutar Servidor
```bash
python manage.py runserver 8076
```

El proyecto está listo y funcional en `http://localhost:8076`
