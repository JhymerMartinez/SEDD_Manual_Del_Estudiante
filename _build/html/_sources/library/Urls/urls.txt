.. _url-title:

*************
Mapeo de URLs
*************

.. _url-intro:

Introducción
============
Django provee un sistema avanzado para la manipulación de URLs que convina la sencillez con la potencia y versatilidad que da como resultado un sistema de enrutamiento elegante.

.. note::
	Para más información revisar `Mapear URLs y Vistas <http://www.cristalab.com/tutoriales/python-en-la-web-con-django-iii-mapear-urls-y-vistas-c103678l/>`_ o en la documentación oficial `URL dispatcher <https://docs.djangoproject.com/en/1.5/topics/http/urls/>`_. 

.. _url-imports:

Importaciones Necesarias
========================

El archivo que contiene el enrutemaiento principal se encuentra ubicado en: ``sedd/proyecto/urls.py``.

Se importa el módulo principal para la administración de las urls::

	from django.conf.urls.defaults import patterns, include, url

Y para la administración::

	from django.contrib import admin


.. _url-main:

Enrutamiento Principal
======================

En esta sección se redacta el enrutamiento necesario para un correcto funcionamiento del Sistema::

	admin.autodiscover()

	urlpatterns = patterns('',

Acontinuación una breve descripción de cada ruta:

.. topic:: Método de origen: :ref:`views-controllers-front`.

	URL: ``/``.

	Nombre: ``portada``.

	Código::

		    url(r'^$', 'proyecto.app.controllers.portada', name='portada'),

.. topic:: Método de origen: :ref:`views-controllers-login-logout`.

	URL: ``/login``.

	Nombre: ``login``.

	Código::

		    url(r'^login/$', 'proyecto.app.controllers.login', name='login'),


.. topic:: Método de origen: :ref:`views-controllers-index`.

	URL: ``index/``.

	Nombre: ``index``.

	Código::

		    url(r'^index/$', 'proyecto.app.controllers.index', name='index'),  


.. topic:: Método de origen: :ref:`views-controllers-login-logout`.

	URL: ``logout/``.

	Nombre: ``logout``.

	Código::

		    url(r'^logout/$', 'proyecto.app.controllers.logout', name='logout'),

.. topic:: Método de origen: :ref:`views-controllers-login-logout`.

	URL: ``estudiante/asignaturas/docentes/(\d{1})/``.

	Nombre: ``estudiante_asignaturas_docentes``.

	En ``(\d{1})`` se recibe como parametro el número de carrera.

	Código::

		    url(r'^estudiante/asignaturas/docentes/(\d{1})/$', 
		        'proyecto.app.controllers.estudiante_asignaturas_docentes', name='estudiante_asignaturas_docentes'),

.. note::
	Para entender de mejor manera el uso de expresiones regulares en Python revisar: `GUÍA PYTHON: EXPRESIONES REGULARES <http://www.maestrosdelweb.com/editorial/guia-python-expresiones-regulares/>`_.


.. topic:: Método de origen: :ref:`views-controllers-evaluation_directors_teachers`.

	URL: ``director/docentes/(\d{1})/``.

	Nombre: ``director_docentes``.

	En ``(\d{1})`` se recibe un número de carrera de la sesion.

	Código::

		    url(r'^director/docentes/(\d{1})/$', 
		        'proyecto.app.controllers.director_docentes',
		        name='director_docentes'),


.. topic::	Método de origen: :ref:`views-controllers-evaluation-academic_couple_teachers`.

	URL: ``pares_academicos/docentes/(\d{1})/``.

	Nombre: ``pares_academicos_docentes``.

	En ``(\d{1})`` se recibe un número de carrera de la sesión.

	Código::

		    url(r'^pares_academicos/docentes/(\d{1})/$', 'proyecto.app.controllers.pares_academicos_docentes', 
		        name='pares_academicos_docentes'),


.. topic:: Método de origen: :ref:`views-controllers-poll`.

	URL: ``encuestas/(?P<id_docente>\d{1,5})/(?P<id_asignatura>\d{1,5})/(?P<id_tinformante>\d{1,2})/(?P<id_cuestionario>\d{1,2})/``. 

	Nombre: ``encuestas``.

	En ``(?P<id_docente>\d{1,5})`` se recibe el id del docente.Mínimo 1 y máximo 5 dígitos.

	En ``(?P<id_asignatura>\d{1,5})`` se recibe el id de la asignatura. Mínimo 1 y máximo 5 dígitos.

	En ``(?P<id_tinformante>\d{1,2})`` se recibe el id del tipo de informante. Mínimo 1 y máximo 2 dígitos.

	En ``(?P<id_cuestionario>\d{1,2})`` se recibe el id del cuestionario. Mínimo 1 y máximo 2 dígitos.

	Código::

		    url(r'^encuestas/(?P<id_docente>\d{1,5})/(?P<id_asignatura>\d{1,5})/(?P<id_tinformante>\d{1,2})/(?P<id_cuestionario>\d{1,2})/', 
		        'proyecto.app.controllers.encuestas', name='encuestas'),



.. topic:: Método de origen: :ref:`views-controllers-poll_create`.

	URL: ``encuesta/responder/(\d{1,5})/``.

	Nombre: ``encuestas``.

	En ``(\d{1,5})`` se recibe el id de la encuesta. Mínimo 1 y máximo 5 digitos.

	Código::

		    url(r'^encuesta/responder/(\d{1,5})/$', 'proyecto.app.controllers.encuesta_responder',
		        name='encuesta_responder'),


.. topic:: Método de origen: :ref:`views-controllers-poll_save`.

	URL: ``encuesta/grabar/``.

	Nombre: ``encuesta_grabar``.

	Código::

		    url(r'^encuesta/grabar/$', 'proyecto.app.controllers.encuesta_grabar',name='encuesta_grabar'),


.. topic:: Método de origen: :ref:`views-controllers-calculation_presentation_results-results_career`.

	URL: ``resultados/carrera/(\d{1})/``.

	Nombre: ``resultados_carrera``.

	En ``(\d{1})`` se recibe número de carrera.

	Código::

		    url(r'^resultados/carrera/(\d{1})/$', 'proyecto.app.controllers.resultados_carrera',name='resultados_carrera'),


.. topic:: Método de origen: :ref:`views-controllers-calculation_presentation_results-menu_results_carreer`.

	URL: ``resultados/periodo/(\d{1,2})/``.

	Nombre: ``menu_resultados_carrera``.

	En ``(\d{1,2})`` se recibe el periodo académico. Mínimo1 y máximo 2 digitos.

	Código::

		    url(r'^resultados/periodo/(\d{1,2})/$', 'proyecto.app.controllers.menu_resultados_carrera',name='menu_resultados_carrera'),


.. topic:: Método de origen: :ref:`views-controllers-calculation_presentation_results-results_presentation`.

	URL: ``resultados/mostrar/``.

	Nombre: ``mostrar_resultados``.

	Código::

		    url(r'^resultados/mostrar/$', 'proyecto.app.controllers.mostrar_resultados',name='mostrar_resultados'), 


.. topic:: Método de origen: :ref:`views-controllers-load_deals_sga`.

	URL: ``sga/cargar_ofertas_sga/(\d{1,3})/``.

	Nombre: ``cargar_ofertas_sga``.

	En ``(\d{1,3})`` las ofertas disponibles en el SGA. Mínimo 1 y máximo 3 dígitos.

	Código::

		    url(r'^sga/cargar_ofertas_sga/(\d{1,3})/$', 'proyecto.app.controllers.cargar_ofertas_sga',name='cargar_ofertas_sga'),


.. topic:: Método de origen: :ref:`views-controllers-summary_evaluation_admin-summary_evaluations`.

	URL: ``admin/resumen/evaluaciones/``.

	Nombre: ``resumen_evaluaciones``.

	Código::

		    url(r'^admin/resumen/evaluaciones/$', 'proyecto.app.controllers.resumen_evaluaciones',name='resumen_evaluaciones'),


.. topic:: Método de origen: :ref:`views-controllers-summary_evaluation_admin-summary_calculate`.

	URL: ``admin/resumen/calcular/``.

	Nombre: ``calcular_resumen``.

	Código::

		    url(r'^admin/resumen/calcular/$', 'proyecto.app.controllers.calcular_resumen',name='calcular_resumen'),



.. topic:: Método de origen: :ref:`views-controllers-calculation_presentation_results-results_admin`.

	URL: ``admin/app/resultados/``.

	Nombre: ``resultados``.

	Código::

		    url(r'^admin/app/resultados/$', 'proyecto.app.controllers.resultados',name='resultados'),

.. topic:: URL para ingreso del administrados
	::
	    url(r'^admin/', include(admin.site.urls)),


.. topic:: API Webservices
	::
	    url(r'^api/', include('proyecto.app.api.urls'))
	)



.. 	    # django-chronograph: Aplicacion para comandos de administración
	    # url(r'^admin/chronograph/job/(?P<pk>\d+)/run/$', 'chronograph.views.job_run', name="admin_chronograph_job_run"),


.. _url-handler:

Enrutamiento Handler
====================