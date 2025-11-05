# Proyecto Libreria AJMG 1194 — Procedimiento completo (Primera parte)

> **Resumen:** documento paso a paso para crear el proyecto Django **backend_Libreria** dentro de la carpeta **UIII_Libreria_1194**, crear la app **app_Libreria**, configurar el entorno, models (Autor), vistas CRUD para Autor, plantillas HTML, urls, registro en admin y migraciones. El diseño es sencillo, con colores suaves y Bootstrap.

---

## Estructura final recomendada (visión)

```
UIII_Libreria_1194/
├─ .venv/
├─ backend_Libreria/   # directorio del proyecto Django (contiene manage.py)
│  ├─ backend_Libreria/
│  │  ├─ __init__.py
│  │  ├─ settings.py
│  │  ├─ urls.py
│  │  └─ wsgi.py
│  └─ manage.py
└─ app_Libreria/
   ├─ migrations/
   ├─ templates/
   │  ├─ base.html
   │  ├─ header.html
   │  ├─ navbar.html
   │  ├─ footer.html
   │  ├─ inicio.html
   │  └─ autor/
   │     ├─ agregar_autor.html
   │     ├─ ver_autores.html
   │     ├─ actualizar_autor.html
   │     └─ borrar_autor.html
   ├─ __init__.py
   ├─ admin.py
   ├─ apps.py
   ├─ models.py
   ├─ views.py
   ├─ urls.py
   └─ tests.py
```

---

## 1 — Crear la carpeta del proyecto y abrir VS Code

1. Abre tu terminal (PowerShell o CMD si usas Windows) y crea la carpeta del proyecto:

```bash
mkdir UIII_Libreria_1194
cd UIII_Libreria_1194
```

2. Abre VS Code en esa carpeta (desde la terminal):

```bash
code .
```

> Esto abre VS Code apuntando a `UIII_Libreria_1194`.

---

## 2 — Abrir terminal en VS Code

* En VS Code: `Terminal` → `New Terminal`. Por defecto la terminal se abrirá en la ruta del proyecto `UIII_Libreria_1194`.

---

## 3 — Crear y activar entorno virtual `.venv`

### Crear el entorno virtual

```bash
python -m venv .venv
```

### Activar el entorno virtual

* **Windows (PowerShell)**

```powershell
.venv\Scripts\Activate.ps1
```

* **Windows (CMD)**

```cmd
.venv\Scripts\activate.bat
```

* **Linux / macOS / WSL**

```bash
source .venv/bin/activate
```

Cuando esté activo verás `(.venv)` al inicio del prompt.

---

## 4 — Seleccionar intérprete de Python en VS Code

1. En VS Code presiona `Ctrl+Shift+P` → escribe `Python: Select Interpreter` → elige el que apunta a `UIII_Libreria_1194/.venv/...`.

> Esto asegura que VS Code use el intérprete del entorno virtual.

---

## 5 — Instalar Django

Con el entorno activado:

```bash
pip install django
```

Verifica la instalación:

```bash
python -m django --version
```

---

## 6 — Crear proyecto `backend_Libreria` sin duplicar carpeta

**Importante:** para evitar crear una carpeta extra `backend_Libreria/backend_Libreria`, usa el indicador `.` al crear el proyecto dentro de una carpeta dedicada `backend_Libreria` o crea el proyecto directamente en la raíz si prefieres.

Opción recomendada (crear carpeta `backend_Libreria` y ejecutar startproject dentro de ella con el punto):

```bash
mkdir backend_Libreria
cd backend_Libreria
django-admin startproject backend_Libreria .
```

Esto genera `manage.py` y la carpeta interna `backend_Libreria/` pero **sin** una carpeta superior adicional.

---

## 7 — Ejecutar servidor en el puerto 1194

Posiciónate en la carpeta que contiene `manage.py` (ej. `UIII_Libreria_1194/backend_Libreria`) y ejecuta:

```bash
python manage.py runserver 127.0.0.1:1194
```

O si prefieres permitir conexiones externas en la red local:

```bash
python manage.py runserver 0.0.0.0:1194
```

---

## 8 — Copiar y pegar el link en el navegador

Una vez que `runserver` esté corriendo, copia en tu navegador:

```
http://127.0.0.1:1194/
```

---

## 9 — Crear la app `app_Libreria`

Desde la carpeta con `manage.py`:

```bash
python manage.py startapp app_Libreria
```

---

## 10 — Código del `models.py` (Autor, Libro, Venta)

Copia el código proporcionado por ti en `app_Libreria/models.py`. Asegúrate de que quede así (ya lo tienes, lo incluyo para referencia y verificación rápida):

```python
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
        Libro, on_delete=models.PROTECT, related_name='ventas', verbose_name="Libro vendido"
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
```

> Recuerda: por ahora trabajaremos sólo con `Autor`. Los modelos `Libro` y `Venta` los dejamos definidos pero sin vistas ni plantillas todavía.

---

## 11 — Realizar migraciones (makemigrations y migrate)

```bash
# Crear migraciones
python manage.py makemigrations

# Aplicar migraciones
python manage.py migrate
```

Si agregas la app en `settings.py` más abajo, vuelve a ejecutar `makemigrations` para la app.

---

## 12 — Vistas (views.py) — CRUD para AUTOR (sin usar forms.py)

Abre `app_Libreria/views.py` y pega lo siguiente:

```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Autor
from django.urls import reverse

# Página de inicio del sistema
def inicio_libreria(request):
    contexto = {
        'titulo': 'Sistema de Administración Libreria AJMG 1194',
    }
    return render(request, 'inicio.html', contexto)

# Agregar autor (muestra formulario y procesa POST)
def agregar_autor(request):
    if request.method == 'POST':
        nombre = request.POST.get('nombre')
        nacionalidad = request.POST.get('nacionalidad')
        fecha_nacimiento = request.POST.get('fecha_nacimiento')
        fecha_fallecimiento = request.POST.get('fecha_fallecimiento') or None
        biografia = request.POST.get('biografia')
        email = request.POST.get('email')
        activo = True if request.POST.get('activo') == 'on' else False

        Autor.objects.create(
            nombre=nombre,
            nacionalidad=nacionalidad,
            fecha_nacimiento=fecha_nacimiento,
            fecha_fallecimiento=fecha_fallecimiento,
            biografia=biografia,
            email=email,
            activo=activo,
        )
        return redirect('ver_autores')

    return render(request, 'autor/agregar_autor.html')

# Ver todos los autores
def ver_autores(request):
    autores = Autor.objects.all()
    return render(request, 'autor/ver_autores.html', {'autores': autores})

# Mostrar formulario de actualización de autor (GET)
def actualizar_autor(request, autor_id):
    autor = get_object_or_404(Autor, id=autor_id)
    return render(request, 'autor/actualizar_autor.html', {'autor': autor})

# Procesar la actualización (POST)
def realizar_actualizacion_autor(request, autor_id):
    autor = get_object_or_404(Autor, id=autor_id)
    if request.method == 'POST':
        autor.nombre = request.POST.get('nombre')
        autor.nacionalidad = request.POST.get('nacionalidad')
        autor.fecha_nacimiento = request.POST.get('fecha_nacimiento')
        fecha_fallecimiento = request.POST.get('fecha_fallecimiento') or None
        autor.fecha_fallecimiento = fecha_fallecimiento
        autor.biografia = request.POST.get('biografia')
        autor.email = request.POST.get('email')
        autor.activo = True if request.POST.get('activo') == 'on' else False
        autor.save()
        return redirect('ver_autores')
    return redirect('actualizar_autor', autor_id=autor_id)

# Borrar autor (confirmación GET + borrado POST)
def borrar_autor(request, autor_id):
    autor = get_object_or_404(Autor, id=autor_id)
    if request.method == 'POST':
        autor.delete()
        return redirect('ver_autores')
    return render(request, 'autor/borrar_autor.html', {'autor': autor})
```

---

## 13 — Crear carpeta `templates` y archivos HTML

Dentro de `app_Libreria`, crea la carpeta `templates` y los archivos pedidos. A continuación tienes plantillas base y las específicas para Autor.

> **Nota:** usa Bootstrap CDN para CSS/JS en `base.html`. Los HTML están pensados para diseño simple y colores suaves.

### `templates/base.html`

```html
<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{% block title %}Libreria AJMG 1194{% endblock %}</title>
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
      body { background-color: #f7f9fb; color: #1f2937; }
      .soft-card { background-color: #ffffff; border-radius: 12px; box-shadow: 0 2px 8px rgba(31,41,55,0.05); }
      footer { position: fixed; bottom: 0; width: 100%; }
    </style>
  </head>
  <body>
    {% include 'navbar.html' %}

    <main class="container my-5">
      {% block content %}{% endblock %}
    </main>

    {% include 'footer.html' %}

    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
  </body>
</html>
```

### `templates/navbar.html`

```html
<nav class="navbar navbar-expand-lg navbar-light bg-white border-bottom">
  <div class="container">
    <a class="navbar-brand" href="#">📚 Sistema de Administración Libreria AJMG 1194</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navMenu">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navMenu">
      <ul class="navbar-nav ms-auto">
        <li class="nav-item"><a class="nav-link" href="{% url 'inicio_libreria' %}">Inicio</a></li>
        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">📖 Autores</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="{% url 'agregar_autor' %}">Agregar Autor</a></li>
            <li><a class="dropdown-item" href="{% url 'ver_autores' %}">Ver Autores</a></li>
          </ul>
        </li>
        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">📚 Libros</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar Libro</a></li>
            <li><a class="dropdown-item" href="#">Ver Libros</a></li>
          </ul>
        </li>
        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">💳 Ventas</a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar Venta</a></li>
            <li><a class="dropdown-item" href="#">Ver Ventas</a></li>
          </ul>
        </li>
      </ul>
    </div>
  </div>
</nav>
```

### `templates/footer.html`

```html
<footer class="bg-white border-top py-3">
  <div class="container text-center small">
    © Derechos reservados — {{ now|date:"Y" }} — Creado por Alfredo Martinez
  </div>
</footer>
```

*(usar la etiqueta `now` requiere activar el context processor o pasar `now` desde la vista; alternativa simple: usar `{{ "now"|date:"Y" }}` no funciona — recomendamos pasar la fecha desde la vista `inicio_libreria`).*

### `templates/inicio.html`

```html
{% extends 'base.html' %}
{% block title %}Inicio - Libreria AJMG{% endblock %}
{% block content %}
  <div class="soft-card p-4">
    <h1>{{ titulo }}</h1>
    <p>Bienvenido al sistema de administración de la librería AJMG 1194.</p>
    <img src="https://images.unsplash.com/photo-1512820790803-83ca734da794" class="img-fluid rounded" alt="Librería">
  </div>
{% endblock %}
```

> **Autor templates**: crea carpeta `templates/autor/` y añade los siguientes archivos.

#### `templates/autor/agregar_autor.html`

```html
{% extends 'base.html' %}
{% block content %}
<h2>Agregar Autor</h2>
<form method="post">
  {% csrf_token %}
  <div class="mb-3">
    <label class="form-label">Nombre</label>
    <input class="form-control" name="nombre" required>
  </div>
  <div class="mb-3">
    <label class="form-label">Nacionalidad</label>
    <input class="form-control" name="nacionalidad">
  </div>
  <div class="mb-3">
    <label class="form-label">Fecha de nacimiento</label>
    <input class="form-control" type="date" name="fecha_nacimiento">
  </div>
  <div class="mb-3">
    <label class="form-label">Fecha de fallecimiento</label>
    <input class="form-control" type="date" name="fecha_fallecimiento">
  </div>
  <div class="mb-3">
    <label class="form-label">Biografía</label>
    <textarea class="form-control" name="biografia"></textarea>
  </div>
  <div class="mb-3 form-check">
    <input type="checkbox" class="form-check-input" name="activo" id="activo">
    <label class="form-check-label" for="activo">Autor activo</label>
  </div>
  <button class="btn btn-primary">Guardar</button>
</form>
{% endblock %}
```

#### `templates/autor/ver_autores.html`

```html
{% extends 'base.html' %}
{% block content %}
<h2>Autores</h2>
<table class="table">
  <thead><tr><th>Nombre</th><th>Nacionalidad</th><th>Fecha Nac.</th><th>Activo</th><th>Acciones</th></tr></thead>
  <tbody>
    {% for autor in autores %}
      <tr>
        <td>{{ autor.nombre }}</td>
        <td>{{ autor.nacionalidad }}</td>
        <td>{{ autor.fecha_nacimiento }}</td>
        <td>{{ autor.activo }}</td>
        <td>
          <a class="btn btn-sm btn-info" href="{% url 'actualizar_autor' autor.id %}">Editar</a>
          <a class="btn btn-sm btn-danger" href="{% url 'borrar_autor' autor.id %}">Borrar</a>
        </td>
      </tr>
    {% empty %}
      <tr><td colspan="5">No hay autores registrados.</td></tr>
    {% endfor %}
  </tbody>
</table>
{% endblock %}
```

#### `templates/autor/actualizar_autor.html`

```html
{% extends 'base.html' %}
{% block content %}
<h2>Actualizar Autor</h2>
<form method="post" action="{% url 'realizar_actualizacion_autor' autor.id %}">
  {% csrf_token %}
  <div class="mb-3">
    <label class="form-label">Nombre</label>
    <input class="form-control" name="nombre" value="{{ autor.nombre }}">
  </div>
  <div class="mb-3">
    <label class="form-label">Nacionalidad</label>
    <input class="form-control" name="nacionalidad" value="{{ autor.nacionalidad }}">
  </div>
  <div class="mb-3">
    <label class="form-label">Fecha de nacimiento</label>
    <input class="form-control" type="date" name="fecha_nacimiento" value="{{ autor.fecha_nacimiento }}">
  </div>
  <div class="mb-3">
    <label class="form-label">Fecha de fallecimiento</label>
    <input class="form-control" type="date" name="fecha_fallecimiento" value="{{ autor.fecha_fallecimiento }}">
  </div>
  <div class="mb-3">
    <label class="form-label">Biografía</label>
    <textarea class="form-control" name="biografia">{{ autor.biografia }}</textarea>
  </div>
  <div class="mb-3 form-check">
    <input type="checkbox" class="form-check-input" name="activo" id="activo" {% if autor.activo %}checked{% endif %}>
    <label class="form-check-label" for="activo">Autor activo</label>
  </div>
  <button class="btn btn-success">Actualizar</button>
</form>
{% endblock %}
```

#### `templates/autor/borrar_autor.html`

```html
{% extends 'base.html' %}
{% block content %}
<h2>Confirmar borrado</h2>
<p>¿Eliminar al autor <strong>{{ autor.nombre }}</strong>?</p>
<form method="post">
  {% csrf_token %}
  <button class="btn btn-danger">Sí, borrar</button>
  <a class="btn btn-secondary" href="{% url 'ver_autores' %}">Cancelar</a>
</form>
{% endblock %}
```

---

## 14 — urls.py en `app_Libreria` (enlazar vistas)

Crea `app_Libreria/urls.py` con:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_libreria, name='inicio_libreria'),
    path('autores/agregar/', views.agregar_autor, name='agregar_autor'),
    path('autores/', views.ver_autores, name='ver_autores'),
    path('autores/editar/<int:autor_id>/', views.actualizar_autor, name='actualizar_autor'),
    path('autores/editar/guardar/<int:autor_id>/', views.realizar_actualizacion_autor, name='realizar_actualizacion_autor'),
    path('autores/borrar/<int:autor_id>/', views.borrar_autor, name='borrar_autor'),
]
```

---

## 15 — Agregar `app_Libreria` en `settings.py`

En `backend_Libreria/settings.py`, dentro de `INSTALLED_APPS` agrega:

```python
INSTALLED_APPS = [
    # ...
    'app_Libreria',
]
```

Asegúrate de tener configurado `TEMPLATES` con `APP_DIRS: True` (por defecto en Django) para que reconozca `templates/` dentro de la app.

---

## 16 — Configurar `urls.py` del proyecto (backend_Libreria/urls.py)

Edita `backend_Libreria/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Libreria.urls')),
]
```

---

## 17 — Registrar modelos en `admin.py` y migraciones finales

En `app_Libreria/admin.py`:

```python
from django.contrib import admin
from .models import Autor, Libro, Venta

@admin.register(Autor)
class AutorAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'nacionalidad', 'fecha_nacimiento', 'activo')
    search_fields = ('nombre', 'nacionalidad')

@admin.register(Libro)
class LibroAdmin(admin.ModelAdmin):
    list_display = ('titulo', 'isbn', 'genero', 'precio', 'stock')
    search_fields = ('titulo', 'isbn')

@admin.register(Venta)
class VentaAdmin(admin.ModelAdmin):
    list_display = ('id', 'libro', 'cantidad', 'total', 'fecha_venta', 'estado')
    list_filter = ('estado',)
```

Luego ejecuta:

```bash
python manage.py makemigrations app_Libreria
python manage.py migrate
```

---

## 18 — Notas sobre diseño y comportamiento solicitado

* **Colores suaves:** ya se incluyen estilos en `base.html` para fondo claro y tarjetas blancas.
* **No validar entrada de datos:** las vistas manejan `POST` directamente sin validaciones (tal como pediste).
* **No usar `forms.py`:** todo el manejo se hace con `request.POST`.
* **Por ahora trabajar sólo con "Autor":** las vistas y plantillas provistas cubren CRUD de Autores. Libro y Venta están definidos en models.py pero se dejan pendientes.

---

## 19 — Comandos resumen (para copiar/pegar rápido)

```bash
# Crear proyecto y app
mkdir UIII_Libreria_1194
cd UIII_Libreria_1194
python -m venv .venv
# activar .venv según tu sistema
pip install django
mkdir backend_Libreria
cd backend_Libreria
django-admin startproject backend_Libreria .
python manage.py startapp app_Libreria
# editar archivos: settings.py, models.py, views.py, templates, urls.py
python manage.py makemigrations
python manage.py migrate
python manage.py runserver 127.0.0.1:1194
```

---

## 20 — Últimos consejos y verificación

1. Si al acceder a `http://127.0.0.1:1194/` ves la página de bienvenida de Django, significa que `manage.py runserver` no está apuntando a tu `app_Libreria` o que `urls.py` no quedó configurado. Verifica `backend_Libreria/urls.py` e `INSTALLED_APPS`.

2. Si ves errores de plantillas, revisa que las rutas y nombres de archivos en `templates/` coincidan exactamente.

3. Para mostrar la fecha actual en el footer, puedes modificar `inicio_libreria` para enviar `{'titulo': ..., 'now': timezone.now()}` y en `footer.html` usar `{{ now|date:"Y" }}`.

---

## 21 — Qué queda pendiente (por ahora)

* Implementar CRUD de `Libro` y `Venta`.
* Añadir autorización/usuarios si se desea más adelante.
* Validación de datos si en un futuro lo solicitas.

**Paso 23: Crear modelo para Clientes y Ventas**

1. Abre el archivo `models.py` dentro de la carpeta `LibreriaAJMG`.

2. Agrega los siguientes modelos debajo del modelo `Libro`:

   ```python
   class Cliente(models.Model):
       nombre = models.CharField(max_length=100)
       correo = models.EmailField()
       telefono = models.CharField(max_length=15)

       def __str__(self):
           return self.nombre

   class Venta(models.Model):
       cliente = models.ForeignKey(Cliente, on_delete=models.CASCADE)
       libro = models.ForeignKey(Libro, on_delete=models.CASCADE)
       fecha_venta = models.DateField(auto_now_add=True)
       cantidad = models.PositiveIntegerField()

       def total(self):
           return self.cantidad * self.libro.precio

       def __str__(self):
           return f"Venta de {self.libro.titulo} a {self.cliente.nombre}"
   ```

3. Guarda el archivo y ejecuta en la terminal:

   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

---

**Paso 24: Registrar los modelos en el administrador**

1. Abre `admin.py` y agrega:

   ```python
   from .models import Libro, Cliente, Venta

   admin.site.register(Libro)
   admin.site.register(Cliente)
   admin.site.register(Venta)
   ```

2. Guarda el archivo y entra al panel de administración (`/admin`) para confirmar que se muestran **Libros**, **Clientes** y **Ventas**.

---

**Paso 25: Crear vistas para Clientes y Ventas**
Abre `views.py` y agrega:

```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Libro, Cliente, Venta

# CLIENTES

def lista_clientes(request):
    clientes = Cliente.objects.all()
    return render(request, 'lista_clientes.html', {'clientes': clientes})

# VENTAS

def lista_ventas(request):
    ventas = Venta.objects.all()
    return render(request, 'lista_ventas.html', {'ventas': ventas})
```

---

**Paso 26: Configurar URLs para Clientes y Ventas**
Abre `urls.py` dentro de `LibreriaAJMG` y agrega las siguientes rutas:

```python
urlpatterns = [
    path('', views.inicio, name='inicio'),
    path('libros/', views.lista_libros, name='lista_libros'),
    path('clientes/', views.lista_clientes, name='lista_clientes'),
    path('ventas/', views.lista_ventas, name='lista_ventas'),
]
```

---

**Paso 27: Crear las plantillas HTML para Clientes y Ventas**
📄 **lista_clientes.html**

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Lista de Clientes - Librería AJMG</title>
</head>
<body>
    <h1>Clientes Registrados</h1>
    <table border="1">
        <tr>
            <th>Nombre</th>
            <th>Correo</th>
            <th>Teléfono</th>
        </tr>
        {% for cliente in clientes %}
        <tr>
            <td>{{ cliente.nombre }}</td>
            <td>{{ cliente.correo }}</td>
            <td>{{ cliente.telefono }}</td>
        </tr>
        {% endfor %}
    </table>
    <br>
    <a href="/">Volver al inicio</a>
</body>
</html>
```

📄 **lista_ventas.html**

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Lista de Ventas - Librería AJMG</title>
</head>
<body>
    <h1>Ventas Realizadas</h1>
    <table border="1">
        <tr>
            <th>Cliente</th>
            <th>Libro</th>
            <th>Cantidad</th>
            <th>Fecha de Venta</th>
            <th>Total</th>
        </tr>
        {% for venta in ventas %}
        <tr>
            <td>{{ venta.cliente.nombre }}</td>
            <td>{{ venta.libro.titulo }}</td>
            <td>{{ venta.cantidad }}</td>
            <td>{{ venta.fecha_venta }}</td>
            <td>{{ venta.total }}</td>
        </tr>
        {% endfor %}
    </table>
    <br>
    <a href="/">Volver al inicio</a>
</body>
</html>
```

---

**Paso 28: Actualizar el menú principal (inicio.html)**
Modifica `inicio.html` agregando enlaces a las nuevas secciones:

```html
<ul>
    <li><a href="/libros/">Ver lista de libros</a></li>
    <li><a href="/clientes/">Ver lista de clientes</a></li>
    <li><a href="/ventas/">Ver lista de ventas</a></li>
</ul>
```

---

**Paso 29: Probar todo el sistema**

1. Ejecuta el servidor:

   ```bash
   python manage.py runserver
   ```
2. Abre en tu navegador:

   ```
   http://127.0.0.1:8000/
   ```
3. Verifica que puedes acceder a las secciones de **Libros**, **Clientes** y **Ventas** correctamente.

✅ **Segunda parte completada:** ahora el sistema muestra toda la información de la librería (Libros, Clientes y Ventas) desde las plantillas HTML.
