# empresaDjango
Desplegando el código en [Render](https://render.com/). Puede accederse a la página web a través de la siguiente url:
https://empresadjango.onrender.com/

## Render

[Service types](https://docs.render.com/service-types)

* [Web Services](https://docs.render.com/web-services): crear un Web Service; dar acceso a nuestro github o especificar un repositorio concreto.
* [Databases (PostgreSQL)](https://docs.render.com/databases)

### Web Service: modificaciones a realizar
Seguir los pasos especificados en quickstarts. Adicionalmente, hacer los siguientes pasos.

1. Especificar un fichero Procfile con lo siguiente:
```
web: gunicorn empresaDjango.wsgi:application
```
Adicionalmente, en la propia página de Render, en settings, cambiar el comando de start commando por el siguiente:
```
gunicorn empresaDjango.wsgi:application
```

2. Crear una base de datos PostgreSQL en la misma página web, y cambiar el contenido de settings.py:
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

a esto otro (hace uso del paquete dj_database_url)

```
import dj_database_url

DATABASES = {
   'default': dj_database_url.config (
       default = 'postgresql://postgres_user:33ZxkwDDzAmtwcRsokevW5XYC2cWgheO@dpg-crhufvbtq21c7383fid0-a/empresadjango', # internal render
       conn_max_age = 600,
   )
}
```


3. Modificar los "ALLOWED_HOSTS" en settings.py:
```
ALLOWED_HOSTS = ['empresadjango.onrender.com', 'localhost', '127.0.0.1']
```


### SQLite to PostgreSQL
Por lo general, Render (y otras plataformas) no recomiendan -- e incluso a veces no permiten-- el uso de SQLite, debido a problemas relacionados con la persistencia de datos, concurrencia y rendimiento. Sin embargo, podemos migrar el contenido actual de nuestro modelo SQLite. 

Para migrar los datos de las 3 tablas de nuestra base de datos a .json:
```
python manage.py dumpdata appEmpresaDjango.Departamento --natural-primary --natural-foreign > departamentos.json
python manage.py dumpdata appEmpresaDjango.Habilidad --natural-primary --natural-foreign > habilidades.json
python manage.py dumpdata appEmpresaDjango.Empleado --natural-primary --natural-foreign > empleados.json
```
--natural-primary y --natural-foreign: Aseguran que las relaciones se exporten correctamente, utilizando las claves naturales en lugar de los IDs numéricos.

Para importarlos en PostgreSQL, tendremos que cambiar nuestro fichero settings.py para que lo utilice en lugar del fichero original que apuntaba a SQLite. Tras ello, cargaremos los datos de forma secuencial (primero departamentos y habilidades, y despues empleados ya que este último tiene relaciones con las anteriores dos tablas):

```
python manage.py loaddata departamentos.json
python manage.py loaddata habilidades.json
python manage.py loaddata empleados.json
```


Podemos verificar que se ha creado todo correctamente mediante shell:
```
python manage.py shell
from appEmpresaDjango.models import Departamento
print(Departamento.objects.count())  # Verifica el número de registros importados
```

#### PostgreSQL load on Render
En el caso de Render, la cuenta de pago no permite utilizar el shell. Para ello, lo que haremos es coger el mismo código que tenemos en github, y cambiaremos la DATABASE_URL para que acceda a la base de datos de forma remota.
* Al crear la base de datos PostgreSQL en Render, nos proveerá de URLs (o datos de login) para poder acceder tanto desde los propios servicios de Render como desde un aplicación externa. Usaremos la última funcionalidad para poder cargar los datos de forma local.

En nuestro ordenador (local), haremos:

```
python3 manage.py migrate
```
para crear la base de datos en funcion de nuestro models.py

Tras ello, ejecutaremos los comandos de load especificados en el apartado anterior, cargando así los datos en la base de datos remota.