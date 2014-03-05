.. _settings-title:

*************************
Configuraciones Generales
*************************


.. _settings-intro:

Introducción
============

Una parte muy importante del proyecto es saber configurar adecuadamente el archivo ``settings.py``, encargado de la conexión a la base de datos, la zona horaria, el idioma, los directorios principales del proyecto, las aplicaciones del proyecto, entre otras.

.. note::
    Para mas información revisar `Primera aplicación en Django <http://www.maestrosdelweb.com/editorial/curso-django-instalacion-y-primera-aplicacion/>`_ o en la documentación oficial: `Settings <https://docs.djangoproject.com/en/1.5/ref/settings/>`_ y `Django settings <https://docs.djangoproject.com/en/1.5/topics/settings/>`_. 

.. _settings-imports:

Importaciones Necesarias
========================

Para el presente proyecto se hará uso del paquete ``os`` de python que será utilizado para la generación automática de las rutas necerarias para el proyecto::

	import os


.. _settings-content:

Contenido de **settings.py**
============================



.. _settings-content-debug:

Activando la depuración
-----------------------

Se activa la depuración con el fín de mantener contantemente informado sobre algún error que se presente durante el desarrollo de la aplicación::

	DEBUG = True
	TEMPLATE_DEBUG = DEBUG



.. _settings-content-login:

Ruta del login
--------------

Se establece la ruta para el registro::

	LOGIN_URL = '/login/'


.. _settings-content-admin:

Administradores
---------------

Información sobre los administradores::

	ADMINS = (
	    # ('Your Name', 'your_email@example.com'),
	)

	MANAGERS = ADMINS


.. _settings-content-bd_conection:

Conexión con la Base de Datos
-----------------------------

Para este ejemplo usaremos una conexión con una base de datos postgres::

	DATABASES = {
	    'default': {
	        'ENGINE': 'django.db.backends.postgresql_psycopg2',
	        'NAME': 'nombre_de_la_BD',
	        'USER': 'nombre_de_usuario',			
	        'PASSWORD': 'mi_password',              	
	        'HOST': 'localhost',                 	
	        'PORT': '5432',                     
	    },
	}



.. _settings-content-time_zone:

Zona horaria, lenguaje y mas
----------------------------

Configuraciones relacionadas con el lenguaje y la zona horaria::

	TIME_ZONE = 'America/Guayaquil'

	LANGUAGE_CODE = 'es-EC'
	SITE_ID = 1

	USE_I18N = True

	USE_L10N = True	


.. _settings-content-static_files:

Configuración de la ruta para archivos estáticos
------------------------------------------------

Rutas usadas por la aplicación para encontrar los archivos estáticos::

	MEDIA_ROOT = ''

	MEDIA_URL = ''

	STATIC_ROOT = ''

	STATIC_URL = '/static/'

	ADMIN_MEDIA_PREFIX = '/static/admin/'
	
	STATICFILES_DIRS = (
	    '%s%s' % (os.path.dirname(__file__), '/static/'),
	)

	STATICFILES_FINDERS = (
	    'django.contrib.staticfiles.finders.FileSystemFinder',
	    'django.contrib.staticfiles.finders.AppDirectoriesFinder',
	)

	#clave encriptada

	SECRET_KEY = 'cpj4-75y1$65e$ysn9m49d0oo6(fr4!p4b%uy_%*m3#3=!h6c+'



.. _settings-content-templates_urls:

Configuración de templates y urls
---------------------------------

Configuraciones para los templates y urls usados en la aplicación::

	TEMPLATE_LOADERS = (
	    'django.template.loaders.filesystem.Loader',
	    'django.template.loaders.app_directories.Loader',
	)

	TEMPLATE_CONTEXT_PROCESSORS = (
	    "django.contrib.auth.context_processors.auth",
	    "django.core.context_processors.debug",
	    "django.core.context_processors.i18n",
	    "django.core.context_processors.media",
	    "django.core.context_processors.static",
	    "django.core.context_processors.request",
	    "django.contrib.messages.context_processors.messages",
	)

	MIDDLEWARE_CLASSES = (
	    'django.middleware.common.CommonMiddleware',
	    'django.contrib.sessions.middleware.SessionMiddleware',
	    'django.middleware.csrf.CsrfViewMiddleware',
	    'django.contrib.auth.middleware.AuthenticationMiddleware',
	    'django.contrib.messages.middleware.MessageMiddleware',
	)

Rutas para las urls::

	ROOT_URLCONF = 'proyecto.urls'

Ruta de los templates::

	TEMPLATE_DIRS = (
	    '%s%s' % (os.path.dirname(__file__), '/templates')
	)



.. _settings-content-installed_apps:

Instalación de aplicaciones
---------------------------

Aplicaciones adicionales para el correcto funcionamiento de la aplicación::

	INSTALLED_APPS = (
	    'django.contrib.auth',
	    'django.contrib.contenttypes',
	    'django.contrib.sessions',
	    'django.contrib.sites',
	    'django.contrib.messages',
	    'django.contrib.staticfiles',

	    # Activando la administración:

	    'django.contrib.admin',

	    # Instalando la aplicación principal

	    'proyecto.app',

	    # se instalo la libreria Werkzeug para mejorar el DEBUG
	    #que a su vez es llamada mediante django_extensions

	    'django_extensions',

	    # para migraciones a la BD

	    'south',

	)



.. _settings-content-logs_backends:

Logs, Backends y demás Configuraciones
--------------------------------------

Configuraciones adicionales para los logs, backends y correos::

	LOGGING = {
	    'version': 1,
	    'disable_existing_loggers': True, #False
	    'formatters': {
	        'verbose': {
	            'format': '[%(levelname)s %(asctime)s %(module)s]:  %(message)s'
	        },
	        'simple': {
	            'format': '%(levelname)s %(message)s'
	        },
	    },    
	    'handlers': {
	        'mail_admins': {
	            'level': 'ERROR',
	            'class': 'django.utils.log.AdminEmailHandler'
	        },
	        'filelogapp': {
	            'level': 'DEBUG',
	            'class': 'logging.FileHandler', 
	            'formatter': 'verbose',
	            'filename': '%s%s' % (os.path.dirname(__file__), '/tmp/sedd.log') 
	        },

	    },
	    'loggers': {
	        'django.request': {
	            'handlers': ['mail_admins'],
	            'level': 'ERROR',
	            'propagate': True,
	        },
	        'logapp': {
	            'handlers': ['filelogapp'],
	            'level': 'DEBUG',
	            'propogate': True,
	        },
	    }
	}

	AUTHENTICATION_BACKENDS = (
	    #'proyecto.auth_backends.SGAAuthBackend',
	    'django.contrib.auth.backends.ModelBackend',
	    'proyecto.auth_backends.DNIAuthBackend',
	)

Adicionales::

	###################################################
	# Cuenta web del SGA 
	#==================================================
	SGAWS_USER = ''
	SGAWS_PASS = ''
	#==================================================

	###################################################
	# Antigua SEDD MySQL database
	#==================================================
	OLD_HOST = ''
	OLD_DBNAME = ''
	OLD_USER = ''
	OLD_PASS = ''
	#==================================================

	###################################################
	# Cuenta de correo
	#==================================================
	EMAIL_USE_TLS = True
	EMAIL_HOST = ''
	EMAIL_HOST_USER = ''
	EMAIL_HOST_PASSWORD = ''
	EMAIL_PORT = 587
	#==================================================
