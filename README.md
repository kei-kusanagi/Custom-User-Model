En  esta occasion, revisaremos la función de Django para administrar usuarios y su propio modelo de base de datos para usuarios  en Django, para poder remplazar mas fácilmente el nombre de usuarios con correo electrónico y agregar campos personalizados y similares.

empezamos como siempre, creando nuestro entorno virtual, instalando Django y creando un nuevo proyecto y app

![image](IMG%20README/Pasted%20image%2020220725130537.png)


iniciamos declarando nuestra app dentro de settings.py 
![image](IMG%20README/Pasted%20image%2020220725132926.png)

y justo abajo declaramos el modelo que usaremos para nuestra base de datos, esto apuntara al modelo de base de datos llamado User que crearemos ahorita mismo

```
	...
	
    'authuser',

]

  

AUTH_USER_MODEL = 'authuser.User'

  

MIDDLEWARE = [

...
```

Asi que vamos a authuser/models.py , aqui podemos comenzar configurando un administrador de usuarios personalizado

primero que nada importamos el UserManager para anular el que viene por default de Django

``from django.contrib.auth.models import UserManager``

Primero haremos una función para crear a los usuarios
```
class CustomUserManager(UserManager):

    def _create_user(self, email, password, **extra_fields):
```

aquí checaremos si el usuario proporciono una dirección de correo electrónico, así que pondremos este if
```

        if not email:

            raise ValueError("You have not provided a valid e-mail address")

```

Aquí limpiaremos ahora un poco mas los datos
```

        email = self.normalize_email(email)

        user = self.model(email=email, **extra_fields)

        user.set_password(password)

        user.save(using=self._db)

  

        return user
```

Después configuraremos unos campos adicionales por si es un usuario normal o un superusuario
```
 def create_user(self, email=None, password=None, **extra_fields):

        extra_fields.setdefault('is_staff', False)

        extra_fields.setdefault('is_superuser', False)

        return self._create_user(email, password, **extra_fields)
```

hora solo copiamos la misma estructura solo cambiamos el 'is_staff' e 'is_superuser' a True
```

    def create_superuser(self, email=None, password=None, **extra_fields):

        extra_fields.setdefault('is_staff', True)

        extra_fields.setdefault('is_superuser', True)

        return self._create_user(email, password, **extra_fields)
```

ahora crearemos la clase User ya que tenemos los modelos de usuario normal y superusuario, ojo nos saldrá un error porque tenemos que importar el AbstractBaseUser y PermissionMixin
```
from django.contrib.auth.models import AbstractBaseUser, PermissionsMixin,

...

class User(AbstractBaseUser, PermissionsMixin):
````
ahora podremos definir que campos quiero tener aquí
```


    email = models.EmailField(blank=True, default='', unique=True)

    name = models.CharField(max_length=255, blank=True, default='')

```

ahora ponemos las tres propiedades para el personal activo y el super usuario
```

    is_active = models.BooleanField(default=True)

    is_superuser = models.BooleanField(default=False)

    is_staff = models.BooleanField(default=False)
```

Tambien queremos el campo para saber cuando el usuario inicio la sesión y cuando el usuario se registro
```
...

from django.utils import timezone

...

    date_joined = models.DateTimeField(default=timezone.now)

    last_login = models.DateTimeField(blank=True, null=True)

```

ahora decimos que los objetos son iguales al administrador de usuarios personalizado, asi que ahora  esto apunta a administrador de usuarios que creamos arriba, asi que ahora en ves de usar el que viene de default con Django usaremos nuestro propio modelo
```

    objects = CustomUserManager()
```
luego queremos especificar que campo es el nombre de usuario para que el campo de nombre de usuario sea igual al correo, y que el campo de correo electrónico sea el de email y que campos deben ser obligatorias, lo pondremos como una lista vacía.
```
  

    USERNAME_FIELD = 'email'

    EMAIL_FIELD = 'email'

    REQUIRED_FIELDS = []
```

ahora necesitamos especificar la clase "Metta"
```
    class Meta:

        verbose_name = 'User'

        verbose_name_plural = 'Users'
```
le ponemos lo siguiente por si queremos obtener el nombre de Django
```

    def get_full_name(self):

        return self.name

    def get_short_name(self):
```
Y por ultimo este return que con el split partirá el correo electrónico lo que nos regresara el nombre sin el @blablabla.com
```
        return self.name or self.email.split('@')[0]
```

![image](IMG%20README/Pasted%20image%2020220725163625.png)

ahora toca hacer las migraciones para dejar listo todo para actualizar nuestra base de datos
![image](IMG%20README/Pasted%20image%2020220725145122.png)

ahora podemos crear un super usuario también
![image](IMG%20README/Pasted%20image%2020220725154150.png)

esto esta usando la función que creamos mas arriba nos preguntara directamente el email en ves de unnombre de usuario como lo hace la forma tradicional de Django
![image](IMG%20README/Pasted%20image%2020220725154223.png)

podemos correr el servidor e ir al sitio /admin y logearnos con este nuevo usuario que creamos

![image](IMG%20README/Pasted%20image%2020220725154410.png)

Pero como podemos observar no aparece nuestra lista de usuarios ya que no están formando parte del Django asi que ahora tenemos que ir a admin.py y lo registramos allí
```
from django.contrib import admin

from .models import User
  

admin.site.register(User)
```

actualizamos y ahora si aparece aqui
![image](IMG%20README/Pasted%20image%2020220725164455.png)

si entramos y vemos el usuario que hemos creado
![image](IMG%20README/Pasted%20image%2020220725164521.png)

podemos ver una version codificada de nuestra contraseña, los permisos que tenemos conectados y similares
![image](IMG%20README/Pasted%20image%2020220725164614.png)

Una ultima cosa mas, en la parte de arriba nos aparece el nombre "split"eado que configuramos, pero si queremos podemos guardar un numbre en el campo Name

![image](IMG%20README/Pasted%20image%2020220725164744.png)
![image](IMG%20README/Pasted%20image%2020220725164918.png)

le damos en SAVE
![image](IMG%20README/Pasted%20image%2020220725164934.png)

Nice, ahora con esto ya sabemos crear modelos personalizados, esto normalmente  se hace antes de comenzar  a hacer cualquier otra cosa cuando crea proyectos con Django para no estropear esto con otras funciones.

Proyecto sacado del video

![image](IMG%20README/Pasted%20image%2020220725165710.png)
[Custom User Model | Explore Django](https://youtu.be/mndLkCEiflg)
https://youtu.be/mndLkCEiflg
