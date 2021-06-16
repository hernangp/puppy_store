# puppy_store

> se siguen los lineamientos del tutorial en: 

https://realpython.com/test-driven-development-of-a-django-restful-api/#why-django-rest-framework


> una vez instalado pipenv:

```python
pip install pipenv
```

> Tenes que instalar los paquetes para eso usas:

```
pipenv install django_rest_framework
pipenv instal django
```

El comando anterior, si no existe un entorno virtual para el proyecto va a crear uno nuevo. 
Si en cambio se quiere crear un entorno virtual con alguna version de python en especial antes de iniciar el entorno virtual:
> ej:

```python
pipenv install --python 3.8.7
```

El comando anterior crea el entorno virtual con el interprete deseado, despues para trabajar correctamente debemos setear el interprete en vs code par aque se ejecute bien. 

En el ejemplo no puse otra version python por lo que en pipfile va a aparecer la version 3.9

> acordarse de tener seleccionado el interprete para la carpeta que vamos a trabajar.

## Ahora arrancamos con el proyecto de la puppie store. 

> en linea de comandos:

```
django-admin startproject puppy_store
```

> cambiamos a la carpeta que creo el proyecto:

en este caso 

```
cd puppy_store
``` 
> en linea de comandos:

```
python manage.py startapp puppies
``` 

se crea la carpeta puppies adentro el proyecto

> tenemos que agregar en settings.py la configuracion para rest_framework:
```
REST_FRAMEWORK = {
    # Use Django's standard `django.contrib.auth` permissions,
    # or allow read-only access for unauthenticated users.
    'DEFAULT_PERMISSION_CLASSES': [],
    'TEST_REQUEST_DEFAULT_FORMAT': 'json'
}
```

> da acceso libre a la api y setea el formato por default en JSON para los request. El acceso no restringido abria que modificarlo para el caso de produccion. 

>tenes que instalar la siguiente libreria para manejar postgres desde python:

```python
pipenv install psycopg2
```

## instalar postgres!

> lo instalas desde la pagina hay que tener cuidado de no darle todo que si, si no te instalar programas basura. 

tenes que agregar a path el bin para poder operar desde el cmd.

>crear basedates
en el cmd:

```
psql
CREATE DATABASE puppy_store_drf;
CREATE DATABASE
\q
``` 


> en settings.py hay que agregar la configuracion de la base de datos: 

## seguimos con el proyecto:
> el siguiente paso es definir el modelo, en models.py:

```python
from django.db import models

class Puppy(models.Model):
    """
    Puppy Model
    Defines the attributes of a puppy
    """
    name = models.CharField(max_length=255)
    age = models.IntegerField()
    breed = models.CharField(max_length=255)
    color = models.CharField(max_length=255)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def get_breed(self):
        return self.name + ' belongs to ' + self.breed + ' breed.'

    def __repr__(self):
        return self.name + ' is added.'
``` 

> siguiente paso es hacer el migrations: 

en linea de comandos:

```
python manage.py makemigrations
python manage.py migrate
```

los comandos anteriores crean en la carpeta de la app donde esta el models.py /puppies un directorio llamado migrations, donde almacena todos los migrations que vas haciendo. Se pueden volver para atras tambien indicando el nombre del migration al que queres volver. (tener cuidado porque si se borra alguna columna se pierde la info)

> Ahora tendriamos que revisar con psql o pgadmin4 que las tablas se crearon correctamente. 

> testeamos que los modelos esten bien creados: 

adentro de la carpeta de la app /puppy_store/puppy_store/puppies creamos una carpeta tests, y adentro de la misma el archivo test_models.py

con el siguiente contenido:

```python
from django.test import TestCase
from ..models import Puppy


class PuppyTest(TestCase):
    """ Test module for Puppy model """

    def setUp(self):
        Puppy.objects.create(
            name='Casper', age=3, breed='Bull Dog', color='Black')
        Puppy.objects.create(
            name='Muffin', age=1, breed='Gradane', color='Brown')

    def test_puppy_breed(self):
        puppy_casper = Puppy.objects.get(name='Casper')
        puppy_muffin = Puppy.objects.get(name='Muffin')
        self.assertEqual(
            puppy_casper.get_breed(), "Casper belongs to Bull Dog breed.")
        self.assertEqual(
            puppy_muffin.get_breed(), "Muffin belongs to Gradane breed.")
```

en el testeo se agregan valores dummy a la tabla puppy mediante el metodo setUp() de django.test.TestCase. Tambien se testea que el metodo get_breed creado en models.py adentro de la clase Puppy devuelva valores correctos. 

Tenemos que agregar un archivo __init__.py a la carpeta tests, y tambien el eliminar el archivos test.py de la carpeta de la app puppies. 

> para testear corremos el comnado en consola:

```
python manage.py test
```
```
deberia indicar lo siguiente:

Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.005s

OK
```

## primer unit test superado!

> ahora tenemos que agregar los serializers:

En la carpeta puppies de la app agregamos el archivo serializers.py con el siguiente codigo: 

```python
from rest_framework import serializers
from .models import Puppy


class PuppySerializer(serializers.ModelSerializer):
    class Meta:
        model = Puppy
        fields = ('name', 'age', 'breed', 'color', 'created_at', 'updated_at')
``` 

el serializer valida el modelo de querysets y entrega la data en un formato bonito para trabajar. 
el ModelSerializer sirve para este caso en que tenemos una relacion one-to-one entre el modelo y los endpoint de la API

## ahora hay que contruir la API

el tutorial encara una metodologia en la que primero genera codigo como para que al testear falle y vas actualizando hasta que pase el test. Una vez que pasa el testeo arrancas con el mismo proceso para un nuevo test. 

> hay que crear un nuevo archivo para testar y eso olo hacemos como antes en la carpeta puppies. Generamos el archivo test_views.py:

```python
import json
from rest_framework import status
from django.test import TestCase, Client
from django.urls import reverse
from ..models import Puppy
from ..serializers import PuppySerializer

# initialize the APIClient app
client = Client()
```

> ahora creamos el esqueleto de las vistas en views.py todas con respuestas vacias

``` python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status
from .models import Puppy
from .serializers import PuppySerializer


@api_view(['GET', 'DELETE', 'PUT'])
def get_delete_update_puppy(request, pk):
    try:
        puppy = Puppy.objects.get(pk=pk)
    except Puppy.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    # get details of a single puppy
    if request.method == 'GET':
        return Response({})
    # delete a single puppy
    elif request.method == 'DELETE':
        return Response({})
    # update details of a single puppy
    elif request.method == 'PUT':
        return Response({})


@api_view(['GET', 'POST'])
def get_post_puppies(request):
    # get all puppies
    if request.method == 'GET':
        return Response({})
    # insert a new record for a puppy
    elif request.method == 'POST':
        return Response({})
```

> creamos las urls en puppies, para eso tenemos que crear un nuevo archivos urls en la carpeta puppies:

```python
from django.conf.urls import url
from . import views


urlpatterns = [
    url(
        r'^api/v1/puppies/(?P<pk>[0-9]+)$',
        views.get_delete_update_puppy,
        name='get_delete_update_puppy'
    ),
    url(
        r'^api/v1/puppies/$',
        views.get_post_puppies,
        name='get_post_puppies'
    )
]
```

> referenciamos las urls creadas en el archivo /puppy_store/urls.py:

```python
from django.contrib import admin
from django.urls import path
from django.conf.urls import include, url

urlpatterns = [
    url(r'^', include('puppies.urls')),
    url(
        r'^api-auth/',
        include(rest_framework.urls, namespace = 'rest_framework')
    ),
    url(r'^admin/', admin.site.urls),
]
``` 

## Arrancamos con la parte de Testeos. 

> en test_views.py agregamos los siguientes testos:

```python
class GetAllPuppiesTest(TestCase):
    """ Test module for GET all puppies API """

    def setUp(self):
        Puppy.objects.create(
            name='Casper', age=3, breed='Bull Dog', color='Black')
        Puppy.objects.create(
            name='Muffin', age=1, breed='Gradane', color='Brown')
        Puppy.objects.create(
            name='Rambo', age=2, breed='Labrador', color='Black')
        Puppy.objects.create(
            name='Ricky', age=6, breed='Labrador', color='Brown')

    def test_get_all_puppies(self):
        # get API response
        response = client.get(reverse('get_post_puppies'))
        # get data from db
        puppies = Puppy.objects.all()
        serializer = PuppySerializer(puppies, many=True)
        self.assertEqual(response.data, serializer.data)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
```

> corremos el test por linea de comandos:

```
python manage.py test
```

En este caso el testo deberia dar error ya que la salidas en views.py las habiamos dejado vacias

deberiamos tener el siguiente mensaje de error:

```
self.assertEqual(response.data, serializer.data)
AssertionError: {} != [OrderedDict([('name', 'Casper'), ('age',[687 chars])])]
```

Si nos da este error se debe solo a que dejamos las respuestas de views.py vacias, entonces podemos ir y completar la salida:

> en views.py modificamos la vista para get_post_puppies siguiente linea:

```python
@api_view(['GET', 'POST'])
def get_post_puppies(request):
    # get all puppies
    if request.method == 'GET':
        puppies = Puppy.objects.all()
        serializer = PuppySerializer(puppies, many='True')
        return Response(serializer.data)
    # insert a new record for a puppy
    elif request.method == 'POST':
        return Response({})
```

> ahora deberiamos correr el test y deberia funcionar:

```
python manage.py test
```

> devuelve el siguiente mensaje:

```
Ran 2 tests in 0.071s

FAILED (errors=1)
Destroying test database for alias 'default'...
(puppy_store) PS D:\mainque_labs\repos mainque\puppy_store\puppy_store> python manage.py test
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..
----------------------------------------------------------------------
Ran 2 tests in 0.054s

OK
Destroying test database for alias 'default'...
```

Podemos seguir adelante!

> ahora vamos a testear el GET SINGLE

para este caso se tiene que contemplar dos casos:

- get valid puppy - e.g. cuando el puppy existe

- get invalid puppy - e.g. cuando el puppy no existe

> agregamos el siguiente test para hacerlo: 

```python
class GetSinglePuppyTest(TestCase):
    """ Test module for GET single puppy API """

    def setUp(self):
        self.casper = Puppy.objects.create(
            name='Casper', age=3, breed='Bull Dog', color='Black')
        self.muffin = Puppy.objects.create(
            name='Muffin', age=1, breed='Gradane', color='Brown')
        self.rambo = Puppy.objects.create(
            name='Rambo', age=2, breed='Labrador', color='Black')
        self.ricky = Puppy.objects.create(
            name='Ricky', age=6, breed='Labrador', color='Brown')

    def test_get_valid_single_puppy(self):
        response = client.get(
            reverse('get_delete_update_puppy', kwargs={'pk': self.rambo.pk}))
        puppy = Puppy.objects.get(pk=self.rambo.pk)
        serializer = PuppySerializer(puppy)
        self.assertEqual(response.data, serializer.data)
        self.assertEqual(response.status_code, status.HTTP_200_OK)

    def test_get_invalid_single_puppy(self):
        response = client.get(
            reverse('get_delete_update_puppy', kwargs={'pk': 30}))
        self.assertEqual(response.status_code, status.HTTP_404_NOT_FOUND)
```

En primer lugar deberia dar error porque todavia no actualizamos el views.py: 

```
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..
----------------------------------------------------------------------
Ran 2 tests in 0.054s

OK
Destroying test database for alias 'default'...
(puppy_store) PS D:\mainque_labs\repos mainque\puppy_store\puppy_store> python manage.py test
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
...F
======================================================================
FAIL: test_get_valid_single_puppy (puppies.tests.test_views.GetSinglePuppyTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "D:\mainque_labs\repos mainque\puppy_store\puppy_store\puppies\tests\test_views.py", line 56, in test_get_valid_single_puppy
    self.assertEqual(response.data, serializer.data)
AssertionError: {} != {'name': 'Rambo', 'age': 2, 'breed': 'Labr[109 chars]05Z'}

----------------------------------------------------------------------
Ran 4 tests in 0.129s
```

Todo OK, podemos seguir a completar el views.py!

> modificamos la view get_delete_update_puppy:

```python
@api_view(['GET', 'UPDATE', 'DELETE'])
def get_delete_update_puppy(request, pk):
    try:
        puppy = Puppy.objects.get(pk=pk)
    except Puppy.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    # get details of a single puppy
    if request.method == 'GET':
        serializer = PuppySerializer(puppy)
        return Response(serializer.data)
´´´

> volvemos a correr el test:

´´´
python manage.py test
´´´

> tenemos el siguiente mensaje: 

```
Ran 4 tests in 0.107s

OK
Destroying test database for alias 'default'...
```

SIGUE TODO OK!


> Ahora testeamos el POST:

los casos de testeo son dos:

> Insertar un puppy valido
> insertar un puppy in{valido

> creamos el test:

```python
class CreateNewPuppyTest(TestCase):
    """ Test module for inserting a new puppy """

    def setUp(self):
        self.valid_payload = {
            'name': 'Muffin',
            'age': 4,
            'breed': 'Pamerion',
            'color': 'White'
        }
        self.invalid_payload = {
            'name': '',
            'age': 4,
            'breed': 'Pamerion',
            'color': 'White'
        }

    def test_create_valid_puppy(self):
        response = client.post(
            reverse('get_post_puppies'),
            data=json.dumps(self.valid_payload),
            content_type='application/json'
        )
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)

    def test_create_invalid_puppy(self):
        response = client.post(
            reverse('get_post_puppies'),
            data=json.dumps(self.invalid_payload),
            content_type='application/json'
        )
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
```

> al correr el test nuevamente deberiamos tener dos errores ya que no tenemos actualizado el views.py

```
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.FF...
======================================================================
FAIL: test_create_invalid_puppy (puppies.tests.test_views.CreateNewPuppyTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "D:\mainque_labs\repos mainque\puppy_store\puppy_store\puppies\tests\test_views.py", line 96, in test_create_invalid_puppy
    self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
AssertionError: 200 != 400

======================================================================
FAIL: test_create_valid_puppy (puppies.tests.test_views.CreateNewPuppyTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "D:\mainque_labs\repos mainque\puppy_store\puppy_store\puppies\tests\test_views.py", line 88, in test_create_valid_puppy
    self.assertEqual(response.status_code, status.HTTP_201_CREATED)
AssertionError: 200 != 201

----------------------------------------------------------------------
Ran 6 tests in 0.127s

FAILED (failures=2)
Destroying test database for alias 'default'...
```

Todo OK segun lo esperado:

> actualizamos el views.py:

```python
@api_view(['GET', 'POST'])
def get_post_puppies(request):
    # get all puppies
    if request.method == 'GET':
        puppies = Puppy.objects.all()
        serializer = PuppySerializer(puppies, many=True)
        return Response(serializer.data)
    # insert a new record for a puppy
    if request.method == 'POST':
        data = {
            'name': request.data.get('name'),
            'age': int(request.data.get('age')),
            'breed': request.data.get('breed'),
            'color': request.data.get('color')
        }
        serializer = PuppySerializer(data=data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
``` 

> corremos nuevamente el test: 

```
Ran 6 tests in 0.138s

OK
Destroying test database for alias 'default'...
```

## otro testeo:

> en este punto tambien podemos testar usando postman o la api web:

en postman: 

con el metodo post: http://localhost:8000/api/v1/puppies/ e ingresando la data como body en form-data:

deberia ingresar los valores dados a la base de datos!

y funciona perfecto, tanto POST como GET ALL and GET SINGLE!


> vamos con el metodo PUT:

> creamos la clase para testear en text_views.py:

Nuevamente hay que testear los dos casos posibles, ingreso válido e inválido

```python
class UpdateSinglePuppyTest(TestCase):
    """ Test module for updating an existing puppy record """

    def setUp(self):
        self.casper = Puppy.objects.create(
            name='Casper', age=3, breed='Bull Dog', color='Black')
        self.muffin = Puppy.objects.create(
            name='Muffy', age=1, breed='Gradane', color='Brown')
        self.valid_payload = {
            'name': 'Muffy',
            'age': 2,
            'breed': 'Labrador',
            'color': 'Black'
        }
        self.invalid_payload = {
            'name': '',
            'age': 4,
            'breed': 'Pamerion',
            'color': 'White'
        }

    def test_valid_update_puppy(self):
        response = client.put(
            reverse('get_delete_update_puppy', kwargs={'pk': self.muffin.pk}),
            data=json.dumps(self.valid_payload),
            content_type='application/json'
        )
        self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)

    def test_invalid_update_puppy(self):
        response = client.put(
            reverse('get_delete_update_puppy', kwargs={'pk': self.muffin.pk}),
            data=json.dumps(self.invalid_payload),
            content_type='application/json')
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
```

tenemos los dos errores esperados:

```
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
......FF
======================================================================
FAIL: test_invalid_update_puppy (puppies.tests.test_views.UpdateSinglePuppyTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "D:\mainque_labs\repos mainque\puppy_store\puppy_store\puppies\tests\test_views.py", line 132, in test_invalid_update_puppy
    self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
AssertionError: 200 != 400

======================================================================
FAIL: test_valid_update_puppy (puppies.tests.test_views.UpdateSinglePuppyTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "D:\mainque_labs\repos mainque\puppy_store\puppy_store\puppies\tests\test_views.py", line 125, in test_valid_update_puppy
    self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)
AssertionError: 200 != 204

----------------------------------------------------------------------
Ran 8 tests in 0.604s

FAILED (failures=2)
Destroying test database for alias 'default'...
```

> actualizamos el views.py y volvemos a testar:

```python
@api_view(['GET', 'DELETE', 'PUT'])
def get_delete_update_puppy(request, pk):
    try:
        puppy = Puppy.objects.get(pk=pk)
    except Puppy.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    # get details of a single puppy
    if request.method == 'GET':
        serializer = PuppySerializer(puppy)
        return Response(serializer.data)

    # update details of a single puppy
    if request.method == 'PUT':
        serializer = PuppySerializer(puppy, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_204_NO_CONTENT)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    # delete a single puppy
    elif request.method == 'DELETE':
        return Response({})
```

y paso los test correctamente!

# ultimo test el delete:

> creamos el test en test_views.py:

```python
class DeleteSinglePuppyTest(TestCase):
    """ Test module for deleting an existing puppy record """

    def setUp(self):
        self.casper = Puppy.objects.create(
            name='Casper', age=3, breed='Bull Dog', color='Black')
        self.muffin = Puppy.objects.create(
            name='Muffy', age=1, breed='Gradane', color='Brown')

    def test_valid_delete_puppy(self):
        response = client.delete(
            reverse('get_delete_update_puppy', kwargs={'pk': self.muffin.pk}))
        self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)

    def test_invalid_delete_puppy(self):
        response = client.delete(
            reverse('get_delete_update_puppy', kwargs={'pk': 30}))
        self.assertEqual(response.status_code, status.HTTP_404_NOT_FOUND)
```

el test arroja los errores debido a que no estan actualizadas las vistas, procedemos como antes:

> modificamos el views.py:

```python
@api_view(['GET', 'DELETE', 'PUT'])
def get_delete_update_puppy(request, pk):
    try:
        puppy = Puppy.objects.get(pk=pk)
    except Puppy.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    # get details of a single puppy
    if request.method == 'GET':
        serializer = PuppySerializer(puppy)
        return Response(serializer.data)

    # delete a single puppy
    elif request.method == 'DELETE':
        puppy.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)

    # update details of a single puppy
    elif request.method == 'PUT':
        serializer = PuppySerializer(puppy, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status= status.HTTP_204_NO_CONTENT)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

> volvemos a correr el test, y todo OK!

> solo queda testear todo con el postman o con la webAPI

## Las tareas siguientes son:

> verificar todo
> ver de agregar algun tipo de autenticacion
> asegurarse que si se sube a produccion no permitir el acceso a la browsable API!!

LISTO POR AHORA




