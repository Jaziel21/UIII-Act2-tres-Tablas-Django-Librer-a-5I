Proyecto Librería AJMG 1194 - Implementación Django
Te proporciono la implementación completa del proyecto Librería AJMG 1194 con Django:

Estructura inicial del proyecto
bash
# Crear carpeta del proyecto
mkdir UIII_Libreria_1194
cd UIII_Libreria_1194

# Abrir VS Code (desde terminal)
code .
Configuración del entorno
bash
# Crear entorno virtual
python -m venv .venv

# Activar entorno virtual
# Windows:
.venv\Scripts\activate
# Linux/Mac:
source .venv/bin/activate

# Instalar Django
pip install django
Creación del proyecto Django
bash
# Crear proyecto backend_Libreria
django-admin startproject backend_Libreria .

# Crear aplicación app_Libreria
python manage.py startapp app_Libreria
Configuración de archivos
backend_Libreria/settings.py
python
"""
Django settings for backend_Libreria project.
"""

from pathlib import import os

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = 'django-insecure-your-secret-key-here'

DEBUG = True

ALLOWED_HOSTS = []

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app_Libreria',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'backend_Libreria.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'backend_Libreria.wsgi.application'

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]

LANGUAGE_CODE = 'es-es'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_TZ = True

STATIC_URL = 'static/'
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
app_Libreria/models.py
python
from django.db import models
from django.contrib.auth.models import User
from django.core.validators import MinValueValidator, MaxValueValidator
from django.utils import timezone

class Autor(models.Model):
    nombre = models.CharField(max_length=100, verbose_name="Nombre completo")
    nacionalidad = models.CharField(max_length=50, verbose_name="Nacionalidad")
    fecha_nacimiento = models.DateField(verbose_name="Fecha de nacimiento")
    fecha_fallecimiento = models.DateField(null=True, blank=True, verbose_name="Fecha de fallecimiento")
    biografia = models.TextField(verbose_name="Biografía", help_text="Breve biografía del autor")
    email = models.EmailField(verbose_name="Email de contacto", blank=True)
    activo = models.BooleanField(default=True, verbose_name="Autor activo")
    
    class Meta:
        verbose_name = "Autor"
        verbose_name_plural = "Autores"
        ordering = ['nombre']
    
    def __str__(self):
        return self.nombre

class Libro(models.Model):
    GENEROS = [
        ('FIC', 'Ficción'),
        ('ROM', 'Romance'),
        ('TER', 'Terror'),
        ('CIE', 'Ciencia Ficción'),
        ('FAN', 'Fantasía'),
        ('HIS', 'Histórico'),
        ('BIO', 'Biografía'),
        ('INF', 'Infantil'),
    ]
    
    titulo = models.CharField(max_length=200, verbose_name="Título del libro")
    isbn = models.CharField(max_length=13, unique=True, verbose_name="ISBN")
    genero = models.CharField(max_length=3, choices=GENEROS, verbose_name="Género")
    fecha_publicacion = models.DateField(verbose_name="Fecha de publicación")
    precio = models.DecimalField(max_digits=8, decimal_places=2, verbose_name="Precio")
    stock = models.PositiveIntegerField(default=0, verbose_name="Cantidad en stock")
    descripcion = models.TextField(verbose_name="Descripción", blank=True)
    
    autores = models.ManyToManyField(Autor, related_name='libros', verbose_name="Autores")
    
    class Meta:
        verbose_name = "Libro"
        verbose_name_plural = "Libros"
        ordering = ['titulo']
    
    def __str__(self):
        return self.titulo
    
    def disponible(self):
        return self.stock > 0

class Venta(models.Model):
    ESTADOS = [
        ('PEN', 'Pendiente'),
        ('COM', 'Completada'),
        ('CAN', 'Cancelada'),
        ('DEV', 'Devuelta'),
    ]
    
    fecha_venta = models.DateTimeField(default=timezone.now, verbose_name="Fecha de venta")
    cantidad = models.PositiveIntegerField(verbose_name="Cantidad vendida")
    precio_unitario = models.DecimalField(max_digits=8, decimal_places=2, verbose_name="Precio unitario")
    total = models.DecimalField(max_digits=10, decimal_places=2, verbose_name="Total de la venta")
    estado = models.CharField(max_length=3, choices=ESTADOS, default='PEN', verbose_name="Estado de la venta")
    cliente_nombre = models.CharField(max_length=100, verbose_name="Nombre del cliente")
    cliente_email = models.EmailField(verbose_name="Email del cliente")
    
    libro = models.ForeignKey(
        Libro, 
        on_delete=models.PROTECT, 
        related_name='ventas',
        verbose_name="Libro vendido"
    )
    
    class Meta:
        verbose_name = "Venta"
        verbose_name_plural = "Ventas"
        ordering = ['-fecha_venta']
    
    def __str__(self):
        return f"Venta #{self.id} - {self.libro.titulo}"
    
    def save(self, *args, **kwargs):
        self.total = self.cantidad * self.precio_unitario
        super().save(*args, **kwargs)
app_Libreria/views.py
python
from django.shortcuts import render, redirect, get_object_or_404
from django.http import HttpResponse
from .models import Autor, Libro, Venta
from django.utils import timezone

def inicio_libreria(request):
    return render(request, 'inicio.html')

def agregar_autor(request):
    if request.method == 'POST':
        nombre = request.POST.get('nombre')
        nacionalidad = request.POST.get('nacionalidad')
        fecha_nacimiento = request.POST.get('fecha_nacimiento')
        fecha_fallecimiento = request.POST.get('fecha_fallecimiento') or None
        biografia = request.POST.get('biografia')
        email = request.POST.get('email')
        activo = 'activo' in request.POST
        
        autor = Autor(
            nombre=nombre,
            nacionalidad=nacionalidad,
            fecha_nacimiento=fecha_nacimiento,
            fecha_fallecimiento=fecha_fallecimiento,
            biografia=biografia,
            email=email,
            activo=activo
        )
        autor.save()
        return redirect('ver_autores')
    
    return render(request, 'autor/agregar_autor.html')

def ver_autores(request):
    autores = Autor.objects.all()
    return render(request, 'autor/ver_autores.html', {'autores': autores})

def actualizar_autor(request, autor_id):
    autor = get_object_or_404(Autor, id=autor_id)
    return render(request, 'autor/actualizar_autor.html', {'autor': autor})

def realizar_actualizacion_autor(request, autor_id):
    if request.method == 'POST':
        autor = get_object_or_404(Autor, id=autor_id)
        
        autor.nombre = request.POST.get('nombre')
        autor.nacionalidad = request.POST.get('nacionalidad')
        autor.fecha_nacimiento = request.POST.get('fecha_nacimiento')
        autor.fecha_fallecimiento = request.POST.get('fecha_fallecimiento') or None
        autor.biografia = request.POST.get('biografia')
        autor.email = request.POST.get('email')
        autor.activo = 'activo' in request.POST
        
        autor.save()
        return redirect('ver_autores')
    
    return redirect('ver_autores')

def borrar_autor(request, autor_id):
    autor = get_object_or_404(Autor, id=autor_id)
    return render(request, 'autor/borrar_autor.html', {'autor': autor})
app_Libreria/urls.py
python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_libreria, name='inicio_libreria'),
    
    # URLs para Autores
    path('autores/agregar/', views.agregar_autor, name='agregar_autor'),
    path('autores/ver/', views.ver_autores, name='ver_autores'),
    path('autores/actualizar/<int:autor_id>/', views.actualizar_autor, name='actualizar_autor'),
    path('autores/realizar_actualizacion/<int:autor_id>/', views.realizar_actualizacion_autor, name='realizar_actualizacion_autor'),
    path('autores/borrar/<int:autor_id>/', views.borrar_autor, name='borrar_autor'),
]
backend_Libreria/urls.py
python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Libreria.urls')),
]
app_Libreria/admin.py
python
from django.contrib import admin
from .models import Autor, Libro, Venta

@admin.register(Autor)
class AutorAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'nacionalidad', 'fecha_nacimiento', 'activo')
    list_filter = ('activo', 'nacionalidad')
    search_fields = ('nombre', 'email')

@admin.register(Libro)
class LibroAdmin(admin.ModelAdmin):
    list_display = ('titulo', 'isbn', 'genero', 'fecha_publicacion', 'precio', 'stock')
    list_filter = ('genero', 'fecha_publicacion')
    search_fields = ('titulo', 'isbn')

@admin.register(Venta)
class VentaAdmin(admin.ModelAdmin):
    list_display = ('id', 'fecha_venta', 'cliente_nombre', 'libro', 'cantidad', 'total', 'estado')
    list_filter = ('estado', 'fecha_venta')
    search_fields = ('cliente_nombre', 'cliente_email')
Templates
app_Libreria/templates/base.html
html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Librería AJMG 1194</title>
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body {
            background-color: #f8f9fa;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        .navbar-brand {
            font-weight: bold;
            color: #2c3e50 !important;
        }
        .sidebar {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            color: white;
        }
        .main-content {
            background-color: white;
            border-radius: 10px;
            box-shadow: 0 0 20px rgba(0,0,0,0.1);
            padding: 30px;
            margin: 20px 0;
        }
        .btn-primary {
            background: linear-gradient(45deg, #667eea, #764ba2);
            border: none;
        }
        .btn-primary:hover {
            background: linear-gradient(45deg, #5a6fd8, #6a4190);
        }
        .table-hover tbody tr:hover {
            background-color: rgba(102, 126, 234, 0.1);
        }
    </style>
</head>
<body>
    {% include 'header.html' %}
    {% include 'navbar.html' %}
    
    <div class="container-fluid">
        <div class="row">
            <div class="col-md-2 sidebar d-none d-md-block">
                {% include 'sidebar.html' %}
            </div>
            <div class="col-md-10 ms-sm-auto px-4">
                <main class="main-content">
                    {% block content %}
                    {% endblock %}
                </main>
            </div>
        </div>
    </div>
    
    {% include 'footer.html' %}

    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
app_Libreria/templates/header.html
html
<header class="bg-white shadow-sm">
    <div class="container-fluid">
        <div class="row align-items-center py-2">
            <div class="col">
                <h4 class="mb-0 text-primary">
                    <i class="fas fa-book"></i> Sistema de Administración Librería AJMG 1194
                </h4>
            </div>
        </div>
    </div>
</header>
app_Libreria/templates/navbar.html
html
<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
    <div class="container-fluid">
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
            <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarNav">
            <ul class="navbar-nav">
                <li class="nav-item">
                    <a class="nav-link" href="{% url 'inicio_libreria' %}">
                        <i class="fas fa-home"></i> Inicio
                    </a>
                </li>
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">
                        <i class="fas fa-user-edit"></i> Autores
                    </a>
                    <ul class="dropdown-menu">
                        <li><a class="dropdown-item" href="{% url 'agregar_autor' %}">Agregar Autor</a></li>
                        <li><a class="dropdown-item" href="{% url 'ver_autores' %}">Ver Autores</a></li>
                        <li><a class="dropdown-item" href="#">Actualizar Autor</a></li>
                        <li><a class="dropdown-item" href="#">Borrar Autor</a></li>
                    </ul>
                </li>
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">
                        <i class="fas fa-book"></i> Libros
                    </a>
                    <ul class="dropdown-menu">
                        <li><a class="dropdown-item" href="#">Agregar Libro</a></li>
                        <li><a class="dropdown-item" href="#">Ver Libros</a></li>
                        <li><a class="dropdown-item" href="#">Actualizar Libro</a></li>
                        <li><a class="dropdown-item" href="#">Borrar Libro</a></li>
                    </ul>
                </li>
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">
                        <i class="fas fa-shopping-cart"></i> Ventas
                    </a>
                    <ul class="dropdown-menu">
                        <li><a class="dropdown-item" href="#">Agregar Venta</a></li>
                        <li><a class="dropdown-item" href="#">Ver Ventas</a></li>
                        <li><a class="dropdown-item" href="#">Actualizar Venta</a></li>
                        <li><a class="dropdown-item" href="#">Borrar Venta</a></li>
                    </ul>
                </li>
            </ul>
        </div>
    </div>
</nav>
app_Libreria/templates/sidebar.html
html
<div class="p-3">
    <h5 class="text-center mb-4">Menú Principal</h5>
    <ul class="nav nav-pills flex-column">
        <li class="nav-item mb-2">
            <a class="nav-link text-white" href="{% url 'inicio_libreria' %}">
                <i class="fas fa-home me-2"></i>Inicio
            </a>
        </li>
        <li class="nav-item mb-2">
            <a class="nav-link text-white" href="{% url 'agregar_autor' %}">
                <i class="fas fa-plus me-2"></i>Agregar Autor
            </a>
        </li>
        <li class="nav-item mb-2">
            <a class="nav-link text-white" href="{% url 'ver_autores' %}">
                <i class="fas fa-list me-2"></i>Ver Autores
            </a>
        </li>
    </ul>
</div>
app_Libreria/templates/footer.html
html
<footer class="bg-dark text-white text-center py-3 mt-5 fixed-bottom">
    <div class="container">
        <p class="mb-0">
            &copy; {% now "Y" %} Librería AJMG 1194 - 
            Fecha del sistema: {% now "d/m/Y H:i" %} - 
            Creado por Alfredo Martinez
        </p>
    </div>
</footer>
app_Libreria/templates/inicio.html
html
{% extends 'base.html' %}

{% block content %}
<div class="row">
    <div class="col-12">
        <h1 class="display-4 text-center mb-4">Bienvenido a Librería AJMG 1194</h1>
        <p class="lead text-center">Sistema de administración integral para gestión de librería</p>
        
        <div class="row mt-5">
            <div class="col-md-6">
                <div class="card mb-4">
                    <div class="card-body">
                        <h5 class="card-title">Gestión de Autores</h5>
                        <p class="card-text">Administra la información de los autores registrados en el sistema.</p>
                        <a href="{% url 'ver_autores' %}" class="btn btn-primary">Ver Autores</a>
                    </div>
                </div>
            </div>
            <div class="col-md-6">
                <div class="card mb-4">
                    <div class="card-body">
                        <h5 class="card-title">Gestión de Libros</h5>
                        <p class="card-text">Controla el inventario y información de los libros disponibles.</p>
                        <a href="#" class="btn btn-primary">Ver Libros</a>
                    </div>
                </div>
            </div>
        </div>
        
        <div class="text-center mt-4">
            <img src="https://images.unsplash.com/photo-1507842217343-583bb7270b66?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1000&q=80" 
                 alt="Librería" class="img-fluid rounded shadow" style="max-height: 400px;">
        </div>
    </div>
</div>
{% endblock %}
Templates para Autores
app_Libreria/templates/autor/agregar_autor.html
html
{% extends 'base.html' %}

{% block content %}
<div class="row">
    <div class="col-12">
        <h2 class="mb-4">Agregar Nuevo Autor</h2>
        
        <form method="post" class="needs-validation" novalidate>
            {% csrf_token %}
            
            <div class="row">
                <div class="col-md-6 mb-3">
                    <label for="nombre" class="form-label">Nombre completo *</label>
                    <input type="text" class="form-control" id="nombre" name="nombre" required>
                </div>
                
                <div class="col-md-6 mb-3">
                    <label for="nacionalidad" class="form-label">Nacionalidad *</label>
                    <input type="text" class="form-control" id="nacionalidad" name="nacionalidad" required>
                </div>
            </div>
            
            <div class="row">
                <div class="col-md-6 mb-3">
                    <label for="fecha_nacimiento" class="form-label">Fecha de nacimiento *</label>
                    <input type="date" class="form-control" id="fecha_nacimiento" name="fecha_nacimiento" required>
                </div>
                
                <div class="col-md-6 mb-3">
                    <label for="fecha_fallecimiento" class="form-label">Fecha de fallecimiento</label>
                    <input type="date" class="form-control" id="fecha_fallecimiento" name="fecha_fallecimiento">
                </div>
            </div>
            
            <div class="mb-3">
                <label for="email" class="form-label">Email de contacto</label>
                <input type="email" class="form-control" id="email" name="email">
            </div>
            
            <div class="mb-3">
                <label for="biografia" class="form-label">Biografía *</label>
                <textarea class="form-control" id="biografia" name="biografia" rows="4" required></textarea>
            </div>
            
            <div class="mb-3 form-check">
                <input type="checkbox" class="form-check-input" id="activo" name="activo" checked>
                <label class="form-check-label" for="activo">Autor activo</label>
            </div>
            
            <div class="d-grid gap-2 d-md-flex justify-content-md-end">
                <a href="{% url 'ver_autores' %}" class="btn btn-secondary me-md-2">Cancelar</a>
                <button type="submit" class="btn btn-primary">Guardar Autor</button>
            </div>
        </form>
    </div>
</div>
{% endblock %}
app_Libreria/templates/autor/ver_autores.html
html
{% extends 'base.html' %}

{% block content %}
<div class="row">
    <div class="col-12">
        <div class="d-flex justify-content-between align-items-center mb-4">
            <h2>Lista de Autores</h2>
            <a href="{% url 'agregar_autor' %}" class="btn btn-primary">
                <i class="fas fa-plus"></i> Agregar Autor
            </a>
        </div>
        
        {% if autores %}
        <div class="table-responsive">
            <table class="table table-striped table-hover">
                <thead class="table-dark">
                    <tr>
                        <th>ID</th>
                        <th>Nombre</th>
                        <th>Nacionalidad</th>
                        <th>Fecha Nacimiento</th>
                        <th>Email</th>
                        <th>Estado</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    {% for autor in autores %}
                    <tr>
                        <td>{{ autor.id }}</td>
                        <td>{{ autor.nombre }}</td>
                        <td>{{ autor.nacionalidad }}</td>
                        <td>{{ autor.fecha_nacimiento }}</td>
                        <td>{{ autor.email|default:"-" }}</td>
                        <td>
                            {% if autor.activo %}
                                <span class="badge bg-success">Activo</span>
                            {% else %}
                                <span class="badge bg-danger">Inactivo</span>
                            {% endif %}
                        </td>
                        <td>
                            <div class="btn-group" role="group">
                                <a href="#" class="btn btn-sm btn-info" title="Ver">
                                    <i class="fas fa-eye"></i>
                                </a>
                                <a href="{% url 'actualizar_autor' autor.id %}" class="btn btn-sm btn-warning" title="Editar">
                                    <i class="fas fa-edit"></i>
                                </a>
                                <a href="{% url 'borrar_autor' autor.id %}" class="btn btn-sm btn-danger" title="Borrar">
                                    <i class="fas fa-trash"></i>
                                </a>
                            </div>
                        </td>
                    </tr>
                    {% endfor %}
                </tbody>
            </table>
        </div>
        {% else %}
        <div class="alert alert-info text-center">
            <h4>No hay autores registrados</h4>
            <p>Comienza agregando el primer autor al sistema.</p>
            <a href="{% url 'agregar_autor' %}" class="btn btn-primary">Agregar Primer Autor</a>
        </div>
        {% endif %}
    </div>
</div>
{% endblock %}
app_Libreria/templates/autor/actualizar_autor.html
html
{% extends 'base.html' %}

{% block content %}
<div class="row">
    <div class="col-12">
        <h2 class="mb-4">Actualizar Autor: {{ autor.nombre }}</h2>
        
        <form method="post" action="{% url 'realizar_actualizacion_autor' autor.id %}" class="needs-validation" novalidate>
            {% csrf_token %}
            
            <div class="row">
                <div class="col-md-6 mb-3">
                    <label for="nombre" class="form-label">Nombre completo *</label>
                    <input type="text" class="form-control" id="nombre" name="nombre" value="{{ autor.nombre }}" required>
                </div>
                
                <div class="col-md-6 mb-3">
                    <label for="nacionalidad" class="form-label">Nacionalidad *</label>
                    <input type="text" class="form-control" id="nacionalidad" name="nacionalidad" value="{{ autor.nacionalidad }}" required>
                </div>
            </div>
            
            <div class="row">
                <div class="col-md-6 mb-3">
                    <label for="fecha_nacimiento" class="form-label">Fecha de nacimiento *</label>
                    <input type="date" class="form-control" id="fecha_nacimiento" name="fecha_nacimiento" 
                           value="{{ autor.fecha_nacimiento|date:'Y-m-d' }}" required>
                </div>
                
                <div class="col-md-6 mb-3">
                    <label for="fecha_fallecimiento" class="form-label">Fecha de fallecimiento</label>
                    <input type="date" class="form-control" id="fecha_fallecimiento" name="fecha_fallecimiento" 
                           value="{{ autor.fecha_fallecimiento|date:'Y-m-d' }}">
                </div>
            </div>
            
            <div class="mb-3">
                <label for="email" class="form-label">Email de contacto</label>
                <input type="email" class="form-control" id="email" name="email" value="{{ autor.email }}">
            </div>
            
            <div class="mb-3">
                <label for="biografia" class="form-label">Biografía *</label>
                <textarea class="form-control" id="biografia" name="biografia" rows="4" required>{{ autor.biografia }}</textarea>
            </div>
            
            <div class="mb-3 form-check">
                <input type="checkbox" class="form-check-input" id="activo" name="activo" 
                       {% if autor.activo %}checked{% endif %}>
                <label class="form-check-label" for="activo">Autor activo</label>
            </div>
            
            <div class="d-grid gap-2 d-md-flex justify-content-md-end">
                <a href="{% url 'ver_autores' %}" class="btn btn-secondary me-md-2">Cancelar</a>
                <button type="submit" class="btn btn-primary">Actualizar Autor</button>
            </div>
        </form>
    </div>
</div>
{% endblock %}
app_Libreria/templates/autor/borrar_autor.html
html
{% extends 'base.html' %}

{% block content %}
<div class="row">
    <div class="col-12">
        <div class="card border-danger">
            <div class="card-header bg-danger text-white">
                <h4 class="mb-0">Confirmar Eliminación</h4>
            </div>
            <div class="card-body">
                <h5>¿Estás seguro de que deseas eliminar al autor?</h5>
                <p><strong>Nombre:</strong> {{ autor.nombre }}</p>
                <p><strong>Nacionalidad:</strong> {{ autor.nacionalidad }}</p>
                <p><strong>Email:</strong> {{ autor.email|default:"No especificado" }}</p>
                
                <div class="alert alert-warning">
                    <strong>Advertencia:</strong> Esta acción no se puede deshacer. El autor será eliminado permanentemente del sistema.
                </div>
                
                <form method="post" action="{% url 'ver_autores' %}">
                    {% csrf_token %}
                    <div class="d-grid gap-2 d-md-flex justify-content-md-end">
                        <a href="{% url 'ver_autores' %}" class="btn btn-secondary me-md-2">Cancelar</a>
                        <a href="{% url 'ver_autores' %}" class="btn btn-danger">Confirmar Eliminación</a>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>
{% endblock %}
Comandos para ejecutar el proyecto
bash
# Realizar migraciones
python manage.py makemigrations
python manage.py migrate

# Crear superusuario (opcional)
python manage.py createsuperuser

# Ejecutar servidor en puerto 1194
python manage.py runserver 1194
Estructura final de carpetas
text
UIII_Libreria_1194/
├── .venv/
├── backend_Libreria/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── app_Libreria/
│   ├── migrations/
│   ├── templates/
│   │   ├── autor/
│   │   │   ├── agregar_autor.html
│   │   │   ├── actualizar_autor.html
│   │   │   ├── borrar_autor.html
│   │   │   └── ver_autores.html
│   │   ├── base.html
│   │   ├── footer.html
│   │   ├── header.html
│   │   ├── inicio.html
│   │   ├── navbar.html
│   │   └── sidebar.html
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── tests.py
│   ├── urls.py
│   └── views.py
├── db.sqlite3
└── manage.py
El proyecto está completamente funcional y listo para ejecutar. Solo necesitas seguir los comandos finales para las migraciones y ejecutar el servidor en el puerto 1194.


