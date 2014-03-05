.. _views-controllers-title:

**************************
Las vistas y controladores 
**************************

.. _views-controllers-intro:

Introducción
============

Un función de vista o una vista, como es conocida generalmente, es una función en Python que hace una solicitud Web y devuelve una respuesta Web, esta respuesta puede ser el contenido de una página, un error 404, una imagen, un documento XML, entre muchas cosas más.

Comunmente se hace uso del archivo ``views.py`` para la redacción del código necesario. Pero para el desarrollo del presente sistema se ha modificado el nombre del archivo a ``controllers.py`` ubicado en ``sedd\proyecto\app``.

.. note::
    Para mas información revisar `CURSO DJANGO: LAS VISTAS <http://www.maestrosdelweb.com/editorial/curso-django-las-vistas/>`_. 


.. _views-controllers-imports-django:

Importaciones de los modulos de Django
======================================

Primero antes de todo se agrega el sistema de codificación para que acepte caracteres especiales::

	#encoding:utf-8

Luego agregamos todos los paquetes necesarios para el correcto funcionamiento de las vistas::

	from django.http import HttpResponse
	from django.core.context_processors import csrf
	from django.shortcuts import render_to_response
	from django.shortcuts import redirect
	from django.template import RequestContext
	from django.template.loader import render_to_string

.. _note::
	Para mas información revisar `Writing views <https://docs.djangoproject.com/en/1.5/topics/http/views/>`_.


Para la autenticación::

	from django.contrib import auth

Anotaciones para confirmación del login::

	from django.contrib.auth.decorators import login_required

Formularios y restricciones::

	from django import forms
	from django.forms.forms import NON_FIELD_ERRORS

Consultas avanzadas , mensajes de error y algunas otras adiciones::

	from django.contrib import messages
	from django.db.models import Q
	from django.utils import simplejson

.. _views-controllers-imports-sistem:

Importaciones de las demás clases del Sistema
=============================================

Haciendo uso de los :ref:`models-title` para usar sus metodo y funciones::

	from proyecto.app.models import Configuracion
	from proyecto.app.models import Cuestionario
	from proyecto.app.models import Seccion
	from proyecto.app.models import Pregunta
	from proyecto.app.models import Evaluacion
	from proyecto.app.models import Contestacion
	from proyecto.app.models import AsignaturaDocente
	from proyecto.app.models import EstudianteAsignaturaDocente
	from proyecto.app.models import EstudiantePeriodoAcademico
	from proyecto.app.models import DocentePeriodoAcademico
	from proyecto.app.models import DireccionCarrera
	from proyecto.app.models import Asignatura
	from proyecto.app.models import PeriodoAcademico
	from proyecto.app.models import PeriodoEvaluacion
	from proyecto.app.models import Tabulacion
	from proyecto.app.models import TabulacionSatisfaccion2012
	from proyecto.app.models import TabulacionAdicionales2012
	from proyecto.app.models import TabulacionEvaluacion2013
	from proyecto.app.models import OfertaAcademicaSGA
	from proyecto.app.models import AreaSGA

Importación de todos los archivos ubicados en ``sedd\proyecto\app\forms.py``::

	from proyecto.app.forms import *

Módulos para autenticación haciendo uso de la red institucional para acceso al SGA::

	from proyecto.tools.sgaws.cliente import SGA
	from proyecto.settings import SGAWS_USER, SGAWS_PASS

Importaciones para trabajar con fechas y logs::

	from datetime import datetime
	import logging
	logg = logging.getLogger('logapp')

.. _views-controllers-front:

Portada
=======

Mediante el uso de la Clase configuración obtenemos el periodo de evaluación actual y a su vez lo envía y renderiza la plantilla :ref:`template-user-portada.html` . La información es enviada haciendo uso de la variable ``datos``::

	def portada(request):
	    pea = Configuracion.getPeriodoEvaluacionActual()
	    datos = dict(periodoEvaluacionActual=pea)
	    return render_to_response("app/portada.html",datos)


.. _views-controllers-login-logout:

Ingreso al Sistema
==================

Para el acceso se hace uso de un registro común mediante un usuario y contraseña. Dichos datos son obtenidos de un formulario y comparados con la información de la base de datos para realizar su verificación::

	def login(request):
	    form = forms.Form()

Se implementó mensajes de error personalizados en caso de que se presente algún error en los datos::

	    form.fields['username'] = forms.CharField(label="Cedula", max_length=30, 
	                                              widget=forms.TextInput(attrs={'title': 'Cedula',}),
	                                              error_messages={'required':'Ingrese nombre de usuario'})


	    form.fields['password'] = forms.CharField(label="Clave SGA", 
	                                              widget=forms.PasswordInput(attrs={'title': 'Clave SGA-UNL',}),
	                                              error_messages={'required':'Ingrese el password'})
	    data = dict(form=form)    
	    if request.POST:
	        username = request.POST.get('username')
	        password = request.POST.get('password')
	        user = auth.authenticate(username=username, password=password)       
Controles necesarios en caso de que el usuario este o no activo::

	        if user is not None and user.is_active:
	            auth.login(request, user)
	            if not Configuracion.getPeriodoAcademicoActual() or not Configuracion.getPeriodoEvaluacionActual(): 
	                return redirect('/logout')
	            else:
	                return redirect('/index')
	        else:
	            form.full_clean()
	            form._errors[NON_FIELD_ERRORS] = form.error_class(['Error de usuario o contraseña'])
	    data.update(csrf(request))

Renderiza la plantilla que se presentará :ref:`template-user-login.html`::

	    return render_to_response("app/login.html",data)

El siguiente método se ejecutará para el cierre de sesión del usuario. Su redirección es hacia la raíz (:ref:`template-user-portada.html`) del proyecto::

	def logout(request):
	    auth.logout(request)
	    return redirect('/')


.. _views-controllers-index:

Página Inicial
==============

Una véz realizado el registro, el siguiente paso es el acceso a la página inicial.
La anotación ``@login_required()`` permite dar acceso al metodo ``index()`` solo en caso de que exista un acceso previo al Sistema caso contrario redirecciona a la página de :ref:`views-controllers-login-logout`::

	@login_required(login_url='/login/')
	def index(request):
	    periodoAcademico = Configuracion.getPeriodoAcademicoActual()
	    usuario = request.user
	    noEstudiante = False
	    noDocente = False
	    periodoEvaluacion = Configuracion.getPeriodoEvaluacionActual()
	    datos = dict()

Se obtiene solo las siglas de las areas del periodo de evaluacion actual::

	    areas_periodo = periodoEvaluacion.areasSGA.values_list('siglas', flat=True)

En caso de que se trate de un estudiante::

	    try:
	        estudiante = EstudiantePeriodoAcademico.objects.get(periodoAcademico=periodoAcademico, usuario=usuario)

Aqui se toma en cuenta solo las carreras que estan dentro de las areas asignadas al presente Periodo de Evaluación::

	        carreras_estudiante = estudiante.asignaturasDocentesEstudiante.filter(
	            asignaturaDocente__asignatura__area__in=areas_periodo).values(
	            'asignaturaDocente__asignatura__carrera','asignaturaDocente__asignatura__area').distinct()

Se agrega a sesión un diccionario con las carreras del estudiante registrado::

	        carreras_estudiante = [ dict(num_carrera=i, nombre=c['asignaturaDocente__asignatura__carrera'],
	                                     area=c['asignaturaDocente__asignatura__area'])
	                                for i,c in enumerate(carreras_estudiante) ]
	        request.session['estudiante'] = estudiante
	        request.session['carreras_estudiante'] = carreras_estudiante
	    except EstudiantePeriodoAcademico.DoesNotExist:
	        noEstudiante = True

En caso de tratarse de un docente::

	    try:
	        docente = DocentePeriodoAcademico.objects.get(periodoAcademico=periodoAcademico, usuario=usuario)
	        request.session['docente'] = docente
	        if 'ACE' in docente.get_areas():
	            cuestionarios_docente = periodoEvaluacion.cuestionarios.filter(informante__tipo='DocenteIdiomas')
	        else:
	            cuestionarios_docente = periodoEvaluacion.cuestionarios.filter(informante__tipo='Docente')
	        request.session['cuestionarios_docente'] = cuestionarios_docente
	        docente_autoevaluaciones = list()
	        cuestionarios_evaluados = [e.cuestionario for e in docente.evaluaciones.all()]
	        for c in cuestionarios_docente:
	            if c in cuestionarios_evaluados:
	                docente_autoevaluaciones.append(dict(cuestionario=c, evaluado=True))
	            else:
	                docente_autoevaluaciones.append(dict(cuestionario=c, evaluado=False))
	        request.session['docente_autoevaluaciones'] = docente_autoevaluaciones

En caso de tratarse de un docente DIRECTOR/COORDINADOR DE CARRERA::

	        if docente.direcciones.count() > 0:
	            request.session['director_carrera'] = docente
	            if 'ACE' in docente.get_areas():
	                cuestionarios_directivos = periodoEvaluacion.cuestionarios.filter(informante__tipo='DirectivoIdiomas')
	            else:
	                cuestionarios_directivos = periodoEvaluacion.cuestionarios.filter(informante__tipo='Directivo')
	            request.session['cuestionarios_directivos'] = cuestionarios_directivos


Se agregan las carreras en las que el docente es coordinador o director, incluyendo también el nombre del área a la que pertenece::

	            carreras_director = [ dict(num_carrera=i,
	                                      nombre=dc.carrera.split('|')[0],
	                                      area=dc.carrera.split('|')[1] )
	                                 for i,dc in enumerate(docente.direcciones.all()) ]
	            request.session['carreras_director'] = carreras_director

Si el docente es también PAR ACADEMICO::

	        elif docente.parAcademico:
	            if 'ACE' in docente.get_areas():
	                cuestionarios_pares_academicos = periodoEvaluacion.cuestionarios.filter(
	                    informante__tipo='ParAcademicoIdiomas')
	            else:
	                cuestionarios_pares_academicos = periodoEvaluacion.cuestionarios.filter(
	                    informante__tipo='ParAcademico')
	            request.session['cuestionarios_pares_academicos'] = cuestionarios_pares_academicos
	            areas_periodo = periodoEvaluacion.areasSGA.values_list('siglas', flat=True)
	            carreras_pares_academicos = docente.asignaturasDocente.filter(asignatura__area__in=areas_periodo).values(
	                'asignatura__carrera','asignatura__area').distinct()

De igual manera se almacena un diccionario de carreras del para academico en la session::

	            carreras_pares_academicos = [ dict(num_carrera=i, nombre=c['asignatura__carrera'], 
	                                               area=c['asignatura__area'])
	                                          for i,c in enumerate(carreras_pares_academicos) ]
	            carreras_aux = [ c['nombre'] for c in carreras_pares_academicos ]
	            if docente.carrera and docente.carrera not in carreras_aux:

En docente solo existe el nombre de la carrera::

	                carreras_pares_academicos.append(dict(num_carrera=len(carreras_pares_academicos), 
	                                                      nombre=docente.carrera, 
	                                                      area=''))
	            aux_carreras = [c['nombre'] for c in carreras_pares_academicos]
	            aux_areas = [c['area'] for c in carreras_pares_academicos]

A continuación también se toma en cuenta los Pares Academicos de Frances y Ruso::

	            if 'ACE' in aux_areas and 'Curso de Ingles' in aux_carreras:
	                carreras_pares_academicos.append(dict(num_carrera=len(carreras_pares_academicos), 
	                                                      nombre=u'Curso de Francés', 
	                                                      area='ACE'))
	                carreras_pares_academicos.append(dict(num_carrera=len(carreras_pares_academicos), 
	                                                      nombre=u'Curso de Ruso', 
	                                                      area='ACE'))
	            request.session['carreras_pares_academicos'] = carreras_pares_academicos

	    except DocentePeriodoAcademico.DoesNotExist:
	        noDocente = True

Si por algún motivo intenta acceder un uuario que no es Estudiante ni Docente en el Periodo Academico Actual::

	    if noEstudiante and noDocente:
	        return redirect('/login/')
	    return render_to_response('app/index.html', datos, context_instance=RequestContext(request))


.. _views-controllers-evaluation_students:

Evaluaciones de los Estudiantes
===============================

Contiene la lógica necesaria para llevar a cabo la evaluación de los Estudiantes::

	@login_required(login_url='/login/')
	def estudiante_asignaturas_docentes(request, num_carrera):

Se recupera el estudiante y sus datos almacenados en sesión::

	    estudiante = request.session['estudiante']
	    carrera = [c['nombre'] for c in request.session['carreras_estudiante']
	               if c['num_carrera'] == int(num_carrera) ][0]
	    area = [c['area'] for c in request.session['carreras_estudiante']
	               if c['num_carrera'] == int(num_carrera) ][0]
	    request.session['carrera'] = carrera
	    request.session['area'] = area
	    request.session['num_carrera'] = num_carrera

Obtenemos los id de las AsignaturasDocente::

	    ids = estudiante.asignaturasDocentesEstudiante.filter(
	        asignaturaDocente__asignatura__carrera=carrera, asignaturaDocente__asignatura__area=area).values_list(
	        'asignaturaDocente__id', flat=True).distinct()
	    asignaturasDocentes = AsignaturaDocente.objects.filter(id__in=ids).order_by('asignatura__nombre')

El código siguiente es necesario para obtener solo asignaturas distintas::

	    asignaturas = set([ad.asignatura for ad in asignaturasDocentes])
	    asignaturas_docentes = []

Los Objetos presentes acontinuación serán renderizados en la vista::

	    periodoEvaluacion = Configuracion.getPeriodoEvaluacionActual()
	    for a in asignaturas:
	        diccionario = {}
	        diccionario['asignatura'] = a
	        diccionario['docentes'] = []

Una asignatura puede tener mas de un docente::

	        docentes = [ad.docente for ad in asignaturasDocentes if ad.asignatura == a]
	        for d in docentes:
	            if a.area == 'MED':
	                cuestionarios = [c for c in periodoEvaluacion.cuestionarios.all()
	                                 if c.informante.tipo == 'EstudianteMED']                                 
	            elif a.area == 'ACE':
	                cuestionarios = [c for c in periodoEvaluacion.cuestionarios.all() 
	                                 if c.informante.tipo == 'EstudianteIdiomas']
Aplicable a Modalidad Prescencial::

	            elif a.semestre == u"1":
	                cuestionarios = [c for c in periodoEvaluacion.cuestionarios.all() 
	                                 if c.informante.tipo == 'EstudianteNovel']
Si tienen los mismos cuestionarios que el resto de semestres::

	            else:
	                cuestionarios = [c for c in periodoEvaluacion.cuestionarios.all() 
	                                 if c.informante.tipo == 'Estudiante']

	            estudianteAsignaturaDocente = estudiante.asignaturasDocentesEstudiante.get(
	                asignaturaDocente__asignatura=a, asignaturaDocente__docente=d
	                )

.. TODO: ?
.. if len(cuestionarios) == 0:
..	 cuestionarios = [c for c in periodoEvaluacion.cuestionarios.all() 
.. if c.informante.tipo == 'Estudiante']	 

Si ya ha contestado todos los cuestionario disponibles para el docente 'd'::

	            num_evaluaciones = estudianteAsignaturaDocente.evaluaciones.count()
	            if num_evaluaciones > 0 and num_evaluaciones == len(cuestionarios):
	                diccionario['docentes'].append(dict(docente=d, evaluado=True))
	            else:

Si no ha contestado aun todos los cuestionario disponibles para el docente 'd'::

	                diccionario['docentes'].append(dict(docente=d, evaluado=False))
	        asignaturas_docentes.append(diccionario)

Para regresar posteriormente a esta vista::

	    request.session['num_carrera'] = str(num_carrera)
	    title = u"{0}>>{1}".format(area,carrera)
	    datos = dict(asignaturas_docentes=asignaturas_docentes, title=title)

Renderización de la plantilla :ref:`template-user-asignaturas_docentes.html`::

	    return render_to_response("app/asignaturas_docentes.html", datos, context_instance=RequestContext(request))



.. _views-controllers-evaluation-academic_couple_teachers:

Evaluacion de Pares Académicos a Docentes 
=========================================

El siguiente método contiene la lógica necesaria para realizar la Evaluación del Par Académico al Docente::

	def pares_academicos_docentes(request, num_carrera):

El Par Academico de la Carrera elije el docente a evaluar::

	    par_academico = request.session['docente']
	    carrera = [c['nombre'] for c in request.session['carreras_pares_academicos']
	               if c['num_carrera'] == int(num_carrera)][0]
	    area = [c['area'] for c in request.session['carreras_pares_academicos']
	               if c['num_carrera'] == int(num_carrera)][0]
	    request.session['carrera'] = carrera
	    request.session['area'] = area
	    request.session['num_carrera'] = num_carrera

Obtenemos los id de los Docentes que dictan Asignaturas en la carrera seleccionada::

	    ids_docentes = AsignaturaDocente.objects.filter(
	        docente__periodoAcademico=Configuracion.getPeriodoAcademicoActual(),
	        asignatura__carrera=carrera,
	        asignatura__area=area).values_list(
	        'docente__id', flat=True).distinct()

Se agregan también los docentes que no tengan Asignaturas pero que pertenezcan a la Carrera::

	    docentes = DocentePeriodoAcademico.objects.filter(
	        Q(periodoAcademico=Configuracion.getPeriodoAcademicoActual()) and
	        (Q(id__in=ids_docentes) or Q(carrera=carrera))).order_by(
	        'usuario__last_name', 'usuario__first_name')
	    docentes_evaluaciones = list()
	    cuestionarios_pares_a = request.session['cuestionarios_pares_academicos']

A continuación una comprobación para verificar si los ciertos cuestionarios ya han sido evaluados::

	    for d in docentes:
	        cuestionarios_evaluados = sum([e.cuestionario in cuestionarios_pares_a for e in d.evaluaciones.all()])

Comparación entre cuestionarios evaluados y cuestionarios establecidos::

	        if cuestionarios_evaluados >= len(cuestionarios_pares_a):
	            docentes_evaluaciones.append(dict(docente=d, evaluado=True))
	        else:
	            docentes_evaluaciones.append(dict(docente=d, evaluado=False))
	    docentes_evaluaciones.sort(
	        lambda x,y: cmp(x['docente'].usuario.first_name, y['docente'].usuario.first_name) or
	        cmp(x['docente'].usuario.last_name, y['docente'].usuario.last_name)
	        )
	    title = u"{0}>>{1}".format(area, carrera)
	    datos = dict(docentes_evaluaciones=docentes_evaluaciones, title=title)

Renderización al template :ref:`template-user-pares_academicos_docentes.html`con envio de datos::

	    return render_to_response('app/pares_academicos_docentes.html', datos, context_instance=RequestContext(request))


.. _views-controllers-evaluation_directors_teachers:

Evaluaciones de Directores a Docentes
=====================================

A continuaciónse presenta la lógica necesaria para la evaluación de Directores a Docentes Académicos.
El  Director de Carrera elije los docentes a evaluar::

	def director_docentes(request, num_carrera):
	    director_carrera = request.session['director_carrera']
	    carrera = [c['nombre'] for c in request.session['carreras_director']
	               if c['num_carrera'] == int(num_carrera)][0]
	    area = [c['area'] for c in request.session['carreras_director']
	               if c['num_carrera'] == int(num_carrera)][0]
	    request.session['carrera'] = carrera
	    request.session['area'] = area
	    request.session['num_carrera'] = num_carrera

Obtenemos los id de los Docentes que dictan Asignaturas en la carrera seleccionada::

	    ids_docentes = AsignaturaDocente.objects.filter(
	        docente__periodoAcademico=Configuracion.getPeriodoAcademicoActual(),
	        asignatura__carrera=carrera,
	        asignatura__area=area).values_list(
	        'docente__id', flat=True).distinct()
	    
Se agregan también los docentes que no tengan Asignaturas pero que pertenezcan a la Carrera::

	    docentes = DocentePeriodoAcademico.objects.filter(
	        Q(periodoAcademico=Configuracion.getPeriodoAcademicoActual()) and
	        (Q(id__in=ids_docentes) or Q(carrera=carrera))).order_by(
	        'usuario__last_name', 'usuario__first_name')
	    direccion = DireccionCarrera.objects.get(carrera=u'{0}|{1}'.format(carrera, area))
	    docentes_evaluaciones = list()
	    cuestionarios = request.session['cuestionarios_directivos']
	    for d in docentes:

Cuestionarios disponibles ya han sido evaluados? Forma pythonica de comparar, contar y sumar::

	        cuestionarios_evaluados = sum([e.cuestionario in cuestionarios for e in d.evaluaciones.all()])

Compara cuestionarios evaluados con cuestionarios establecidos::

	        if cuestionarios_evaluados >= len(cuestionarios):
	            docentes_evaluaciones.append(dict(docente=d, evaluado=True))
	        else:
	            docentes_evaluaciones.append(dict(docente=d, evaluado=False))
	    docentes_evaluaciones.sort(
	        lambda x,y: cmp(x['docente'].usuario.first_name, y['docente'].usuario.first_name) or
	        cmp(x['docente'].usuario.last_name, y['docente'].usuario.last_name)
	        )
	    title = u"{0}>>{1}".format(area, carrera)
	    datos = dict(docentes_evaluaciones=docentes_evaluaciones, direccion=direccion,title=title)

Renderización al template :ref:`template-user-director_docentes.html`con envio de datos::

	    return render_to_response('app/director_docentes.html', datos, context_instance=RequestContext(request))

.. _views-controllers-poll:

Listado de Encuestas
====================

Elsiguiente código tiene la finalidad el listar las encuestas para los informantes en el presente periodo de evaluación::

	def encuestas(request, id_docente, id_asignatura=0, id_tinformante=0, id_cuestionario=0):
	    
El parámetro ``id_docente`` representa el Docente a ser evaluado
El parámetro ``id_asignatura`` contiene la asignatura que dicta el docente a evaluar 
El parámetro ``id_informante`` contiene el Id Tipo Informante, por ejemplo: 1 Estudiante, 5 Docente, 6 Directivo 
El parámetro ``id_cuestionario`` Solo se usa en el caso de cuestionarios para docentes.

::

	    datos = dict()
	    title=''
	    periodoEvaluacionActual = Configuracion.getPeriodoEvaluacionActual()
	    cuestionarios = []
	    periodo_finalizado = False
	    periodo_no_iniciado = False

En caso de Periodo no iniciado aun::

	    if periodoEvaluacionActual.noIniciado():
	        periodo_no_iniciado = True

En caso de Periodo Vigente::

	    elif periodoEvaluacionActual.vigente():
	        
Si se ha ingresado como ESTUDIANTE::

	        if id_tinformante == '1':
	            estudiante = request.session['estudiante']

Se maneja las siglas del Area en la session::

	            area = request.session['area']
	            carrera = request.session['carrera']
	            asignaturaDocente = AsignaturaDocente.objects.get(docente__id=id_docente, asignatura__id=id_asignatura)
	            estudianteAsignaturaDocente = EstudianteAsignaturaDocente.objects.get(
	                estudiante=estudiante, asignaturaDocente=asignaturaDocente)
	            request.session['estudianteAsignaturaDocente'] = estudianteAsignaturaDocente
	            title = u"{0}>>{1}>>{2}>>{3}".format(area, carrera, asignaturaDocente.asignatura.nombre,
	                                                 asignaturaDocente.docente) 
	            if asignaturaDocente.asignatura.area == 'MED':
	                cuestionarios = [c for c in periodoEvaluacionActual.cuestionarios.all()
	                                 if c.informante.tipo == 'EstudianteMED']                                 
Estudiante (Asignatura) del Instituto de Idiomas::

	            elif asignaturaDocente.asignatura.area == "ACE":
	                cuestionarios = [c for c in periodoEvaluacionActual.cuestionarios.all() 
	                                 if c.informante.tipo == 'EstudianteIdiomas']
Estudiante del Primer Semestre::

	            elif asignaturaDocente.asignatura.semestre == u"1":
	                cuestionarios = [c for c in periodoEvaluacionActual.cuestionarios.all() 
	                                 if c.informante.tipo == 'EstudianteNovel']
Si los cuestionarios son los mismos para todos los semestres.

	                .. #TODO: ?
	                .. #if len(cuestionarios) == 0:
	                .. #    cuestionarios = [c for c in periodoEvaluacionActual.cuestionarios.all() 
	                .. #                 if c.informante.tipo == 'Estudiante']

Estudiante del segundo semestre en adelante::

	            else:
	                cuestionarios = [c for c in periodoEvaluacionActual.cuestionarios.all() 
	                             if c.informante.tipo == 'Estudiante']

Si ya ha contestado todos los cuestionario disponibles. Para esto se realiza una comprobación para verificar si existen cuestionarios disponibles::

	            evaluados = sum([e.cuestionario in cuestionarios for e in estudianteAsignaturaDocente.evaluaciones.all()])

Comparación entre cuestionarios evaluados y cuestionarios establecidos::

	            if evaluados >= len(cuestionarios):
	                return redirect('/estudiante/asignaturas/docentes/' + request.session['num_carrera'])

La siguiente sección de código es ejecutado en el caso de que el usuario ha ingresado como docente y es DIRECTOR DE CARRERA::

	        elif id_tinformante == '6': #Tambien DirectorIdiomas 

	            # Se maneja las siglas del Area en la sesion

	            director_carrera = request.session['director_carrera']
	            area = request.session.get('area', None)
	            carrera = request.session.get('carrera', None)
	            docente_evaluar = DocentePeriodoAcademico.objects.get(id=id_docente)
	            request.session['docente_evaluar'] = docente_evaluar
	            title = u"{0}>>{1}>>Coordinador: {2}".format(area, carrera, director_carrera) 
	            cuestionarios = request.session['cuestionarios_directivos']
	            datos.update(dict(docente_evaluar=docente_evaluar))

	            # Cuestionarios disponibles ya han sido evaluados? Forma pythonica de comparar, contar y sumar

	            evaluados = sum([e.cuestionario in cuestionarios for e in docente_evaluar.evaluaciones.all()])

Comparación entre cuestionarios evaluados y cuestionarios establecidos::

	            if evaluados >= len(cuestionarios):
	                return redirect('/director/docentes/' + request.session['num_carrera'])
La siguiente sección de código es ejecutado en el caso de que el usuario ha ingresado como docente y es PAR ACADEMICO::

	        elif id_tinformante == '10': #Tambien ParAcademicoIdiomas

	            # Se maneja las siglas del Area en la sesion

	            par_academico = request.session['docente']
	            area = request.session.get('area', None)
	            carrera = request.session.get('carrera', None)
	            docente_evaluar = DocentePeriodoAcademico.objects.get(id=id_docente)
	            request.session['docente_evaluar'] = docente_evaluar
	            title = u"{0}>>{1}>>Par Académico: {2}".format(area, carrera, par_academico) 
	            cuestionarios = request.session['cuestionarios_pares_academicos']
	            datos.update(dict(docente_evaluar=docente_evaluar))

	            # Cuestionarios disponibles ya han sido evaluados? Forma pythonica de comparar, contar y sumar

	            evaluados = sum([e.cuestionario in cuestionarios for e in docente_evaluar.evaluaciones.all()])

Comparación cuestionarios evaluados y cuestionarios establecidos::

	            if evaluados >= len(cuestionarios):
	                return redirect('/pares_academicos/docentes/' + request.session['num_carrera'])

La siguiente sección de código es ejecutado en el caso de que el usuario es DOCENTE::

	        elif id_tinformante == '5' and id_cuestionario: # Tambien DocenteIdiomas

Se filtra la petición de encuestas de autoevaluacion de docentes desde el index por este controlador para reutilizar la validación de vigencia de PeridoEvaluacion::

	            return redirect('/encuesta/responder/' + id_cuestionario)

Si ha expirado el periodo de Evaluacion::

	    elif periodoEvaluacionActual.finalizado():
	        periodo_finalizado = True
	    datos.update(dict(cuestionarios=cuestionarios, title=title, 
	                 periodo_no_iniciado=periodo_no_iniciado, periodo_finalizado=periodo_finalizado))

Renderización al template :ref:`template-user-encuestas.html`con envio de datos::

	    return render_to_response("app/encuestas.html", datos, context_instance=RequestContext(request))


.. _views-controllers-poll_create:

Generación de la Encuesta
=========================

La finalidad de este código es generar y presentar la encuesta que será comletada por los estudiantes, Directores de carrera, pares académicos y Docentes en general para autoevaluacion::

	def encuesta_responder(request, id_cuestionario):
	    datos = dict()
	    cuestionario = Cuestionario.objects.get(id=id_cuestionario)
	    evaluacion = Evaluacion()

	    if cuestionario.informante.tipo.startswith('Estudiante'):
	        area = request.session['area']
	        carrera = request.session['carrera']
	        estudianteAsignaturaDocente = request.session['estudianteAsignaturaDocente']
	        evaluacion.estudianteAsignaturaDocente = estudianteAsignaturaDocente
	        title = u"{0}>>{1}>>{2}>>{3}".format(area, carrera, 
	                                             estudianteAsignaturaDocente.asignaturaDocente.asignatura.nombre,
	                                             estudianteAsignaturaDocente.asignaturaDocente.docente
	                                             )
	        datos.update(dict(asignaturaDocente=estudianteAsignaturaDocente.asignaturaDocente))


.. Parámetro ``info_estudiante = ('EstudianteNovel', 'Estudiante', 'EstudianteMED', 'EstudianteIdiomas')``.


Encuesta para DIRECTORES DE CARRERA::

	    elif cuestionario.informante.tipo.startswith('Directivo'): 
	        area = request.session['area']
	        carrera = request.session['carrera']
	        director_carrera = request.session['director_carrera']
	        docente_evaluar = request.session['docente_evaluar']
	        title = u"{0}>>{1}>>{2}".format(area, carrera, docente_evaluar)
	        evaluacion.directorCarrera = director_carrera
	        evaluacion.carreraDirector = u'{0}|{1}'.format(carrera, area)
	        evaluacion.docentePeriodoAcademico = docente_evaluar
	    
Encuesta para PARES ACADEMICOS::

	    elif cuestionario.informante.tipo.startswith('ParAcademico'): 
	        area = request.session['area']
	        carrera = request.session['carrera']
	        par_academico = request.session['docente']
	        docente_evaluar = request.session['docente_evaluar']
	        title = u"{0}>>{1}>>{2}".format(area, carrera, docente_evaluar)
	        evaluacion.parAcademico = par_academico
	        evaluacion.docentePeriodoAcademico = docente_evaluar
	    
Encuesta para Autoevaluacion de DOCENTES::

	    elif cuestionario.informante.tipo.startswith('Docente'): 

En caso de que el docente sea a la vez director de carrera::

	        docente = None
	        if request.session.get('director_carrera', None):
	            docente = request.session['docente']
	        elif request.session.get('docente', None):
	            docente = request.session['docente']
	        evaluacion.docentePeriodoAcademico = docente

Areas en las que dicta clases el docente::

	        areas_docente = AsignaturaDocente.objects.filter(docente=docente).values_list('asignatura__area', flat=True).distinct()
	        areas_docente = ','.join(areas_docente)

Carreras en las que dicta clases el docente::

	        carreras_docente = AsignaturaDocente.objects.filter(docente=docente).values_list(
	            'asignatura__carrera', flat=True).distinct()
	        carreras_docente = ','.join(carreras_docente)
	        request.session['areas_docente'] = areas_docente
	        request.session['carreras_docente'] = carreras_docente
	        title = u'{0}>>{1}>>{2}'.format(areas_docente, carreras_docente, docente)

	    evaluacion.fechaInicio = datetime.now().date()
	    evaluacion.horaInicio = datetime.now().time()
	    evaluacion.cuestionario = cuestionario
	    request.session['evaluacion'] = evaluacion
	    fecha = datetime.now()
	    datos.update(dict(cuestionario=cuestionario, title=title, fecha=fecha))

Parametro adicional por cuestion del formulario::

	    datos.update(csrf(request))   

Renderización al template :ref:`template-user-encuesta_responder.html`con envio de datos::

	    return render_to_response("app/encuesta_responder.html", datos, context_instance=RequestContext(request))



.. _views-controllers-poll_save:

Grabación de la Encuesta
========================

El presente código tiene la finalidad de grabar la encuesta una vez que ha sido desarrollada por los estudiantes o docentes adecuados::

	def encuesta_grabar(request):
	    datos = dict(num_carrera=request.session.get('num_carrera', None))
	    evaluacion = request.session.get('evaluacion', None)

En caso de intentar dar nuevamente la encuesta terminada se redirecciona a :ref:`template-user-encuesta_finalizada.html` ::

	    if not evaluacion:
	        return render_to_response('app/encuesta_finalizada.html', datos, context_instance=RequestContext(request))     
	    
Finalización de la encuesta dirigida a ESTUDIANTES:

.. ###info_estudiante = ('EstudianteNovel', 'Estudiante', 'EstudianteMED', 'EstudianteIdiomas')
::

	    if evaluacion.cuestionario.informante.tipo.startswith('Estudiante'):
..	    ###if request.session.get('estudianteAsignaturaDocente', None):
	        estudianteAsignaturaDocente = request.session['estudianteAsignaturaDocente']
::
Si se regresa a grabar otra vez la misma encuesta::

	        if Evaluacion.objects.filter(estudianteAsignaturaDocente = estudianteAsignaturaDocente
	                                     ).filter(cuestionario = evaluacion.cuestionario).count() > 0:
	            request.session['evaluacion'] = None
	            request.session['estudianteAsignaturaDocente'] = None
	            return render_to_response('app/encuesta_finalizada.html',
	                                  datos, context_instance=RequestContext(request))     

Finalización de la Encuesta dirigida a DIRECTORES DE CARRERA::

	    elif evaluacion.cuestionario.informante.tipo.startswith('Directivo'):
	        docente_evaluar = request.session['docente_evaluar']
	        director_carrera = request.session['director_carrera']

Si ya ha contestado este cuestionario y se intenta grabar otra vez::

	        if docente_evaluar.evaluaciones.filter(cuestionario=evaluacion.cuestionario,
	                                               directorCarrera=director_carrera).count() > 0:
	            request.session['evaluacion'] = None
	            return render_to_response('app/encuesta_finalizada.html', datos, 
	                                      context_instance=RequestContext(request))     

Finalización de la encuesta dirigida a PARES ACADEMICOS::


	    elif evaluacion.cuestionario.informante.tipo.startswith('ParAcademico'):
	        docente_evaluar = request.session['docente_evaluar']
	        par_academico = request.session['docente']

Si ya ha contestado este cuestionario y se intenta grabar otra vez::

	        if docente_evaluar.evaluaciones.filter(cuestionario=evaluacion.cuestionario,
	                                               parAcademico=par_academico).count() > 0:
	            request.session['evaluacion'] = None
	            return render_to_response('app/encuesta_finalizada.html', datos, 
	                                      context_instance=RequestContext(request))     

Finalización de la Encuesta dirigida a DOCENTES::

	    elif evaluacion.cuestionario.informante.tipo.startswith('Docente'):
	        docente = request.session.get('docente', None)

Si ya ha contestado este cuestionario y se intenta grabar otra vez::

	        if docente.evaluaciones.filter(cuestionario=evaluacion.cuestionario).count() > 0:
	            request.session['evaluacion'] = None
	            return render_to_response('app/encuesta_finalizada.html', datos, 
	                                      context_instance=RequestContext(request))  

Grabacion luego de las validaciones::

	    evaluacion.fechaFin = datetime.now().date()
	    evaluacion.horaFin = datetime.now().time()
	    evaluacion.save()
	    for k,v in request.POST.items():
	        if k.startswith('csrf'):
	            continue
	        if k.startswith('pregunta'):
	            id_pregunta = int(k.split('-')[1])
	            observaciones = None

Solo se graban las observaciones de las preguntas respondidas::

	            if Pregunta.objects.get(id=id_pregunta).observaciones:
	                observaciones = request.POST['observaciones-pregunta-' + str(id_pregunta)]
	            contestacion = Contestacion(pregunta=id_pregunta, respuesta=v, observaciones=observaciones)
	            contestacion.evaluacion = evaluacion
	            contestacion.save()
	    logg.info("Nueva Evaluacion realizada: {0}".format(evaluacion))

Una vez terminada la encuesta se redirecciona a :ref:`template-user-encuesta_finalizada.html` ::

	    return render_to_response('app/encuesta_finalizada.html', datos, context_instance=RequestContext(request))


.. _views-controllers-load_deals_sga:

Ofertas SGA
===========

El método descrito a continuación se invoca a traves de AJAX conla finalidad de recargar todas las ofertas presentes en en SGA::

	def cargar_ofertas_sga(request, periodoAcademicoId):

En caso de no existir un periodo académico::

	    if not periodoAcademicoId:
	        return HttpResponse("Falta Periodo Academico")

Excepción::

	    try:
	        proxy = SGA(SGAWS_USER, SGAWS_PASS)
	        periodoAcademico = PeriodoAcademico.objects.get(id=periodoAcademicoId)
	        ofertas_dict = proxy.ofertas_academicas(periodoAcademico.inicio, periodoAcademico.fin)
	        ofertas = [OfertaAcademicaSGA(idSGA=oa['id'], descripcion=oa['descripcion'])  for oa in ofertas_dict]
	        for oa in ofertas:
	            try:
	                OfertaAcademicaSGA.objects.get(idSGA=oa.idSGA)
	            except OfertaAcademicaSGA.DoesNotExist:
	                oa.save()
	        return HttpResponse("OK")

En caso de algún error se presenta un mensaje::

	    except Exception, e:
	        log.error("Error recargando ofertas SGA: " + str(e))
	        return HttpResponse("error: "+str(e))


.. _views-controllers-summary_evaluation_admin:

Resumen de Evaluaciones en modo Administrador
=============================================

Los métodos descritos a continuación tienen la lógica necesaria para permitirle, al Administrador del Sistema, la obtención de los resúmenes detallados de las distintas evaluaciones. 


.. _views-controllers-summary_evaluation_admin-summary_evaluations:

Resumen de Evaluaciones
-----------------------

Método invocado a través de AJAX cueado hay un cambio en el menú del Resumen ::

	def resumen_evaluaciones(request):

Se dectecta si es petición mediante AJAX::

	    if request.is_ajax():
	        respuesta = menu_academico_ajax(request)
	        return HttpResponse(respuesta, mimetype='application/json')

Creación del formulario con los campos necesarios::

	    else:
	        form = forms.Form()
	        form.fields['periodo_academico'] = forms.ModelChoiceField(queryset=PeriodoAcademico.objects.all())
	        form.fields['periodo_academico'].label = 'Periodos Academicos'
	        form.fields['periodo_evaluacion'] = forms.ModelChoiceField(PeriodoEvaluacion.objects.none())
	        form.fields['periodo_evaluacion'].label = 'Periodos de Evaluacion'
	        form.fields['area'] = forms.ModelChoiceField(AreaSGA.objects.none())
	        form.fields['carrera'] = forms.ModelChoiceField(Asignatura.objects.none())
	        form.fields['semestre'] = forms.ModelChoiceField(Asignatura.objects.none())
	        form.fields['paralelo'] = forms.ModelChoiceField(Asignatura.objects.none())
	        periodoEvaluacion = Configuracion.getPeriodoEvaluacionActual()
	        datos = dict(form=form, evaluadores=periodoEvaluacion.contabilizar_evaluadores(), periodoEvaluacion=periodoEvaluacion)

Renderización del template :ref:`template-admin-resumen_evaluaciones.html` ubicado en la carpeta ``admin/app`` que a su vez contine los templates usados en la parte de administración::

	        return render_to_response("admin/app/resumen_evaluaciones.html", datos)




.. _views-controllers-summary_evaluation_admin-summary_calculate:

Calculos de Resumen
-------------------

Se incluye dentro de una Excepción por si se presenta algún error durante el llamado::

	def calcular_resumen(request):
		try:
		        if request.is_ajax():
		            id_periodo_evaluacion = int(request.GET['id_periodo_evaluacion'])
		            area = request.GET['area']
		            carrera = request.GET['carrera']
		            semestre = request.GET['semestre']
		            paralelo = request.GET['paralelo']
		            periodoEvaluacion = PeriodoEvaluacion.objects.get(id=id_periodo_evaluacion)
		            resumen = periodoEvaluacion.contabilizar_evaluaciones_estudiantes(area, carrera, semestre, paralelo)

Contiene: estudiantes, completados, faltantes::

		            return HttpResponse(simplejson.dumps(resumen), mimetype='application/json')
		    except Exception, ex:
		        logg.error("Error calculando resumen de evaluaciones {0}".format(ex))




.. _views-controllers-summary_evaluation_admin-menu_academy_ajax:

Menú Académico
--------------

Método usado para la creación del menú que contiene las funcionalidades para los resúmenes de las evaluaciones. Se caracteriza por su funcionalidad reutilizada cuando se necesita informacion académica jerárquica estructurada en (PeriodoAcademico, PeriodoEvaluacion, AreaSGA, carrera, semestre, paralelo). Utilizado generalmente en menus de reportes::

	def menu_academico_ajax(request):
	    try:
	        id_campo = request.GET['id']
	        valor_campo = request.GET['valor']
	        campo_siguiente = request.GET['siguiente']
	        if valor_campo == '':
	            return HttpResponse('{"id": "", "valores": []}', mimetype="JSON")
	        id = ""
	        valores = []

Según el tipo de valor que contenga ``id_campor``, se realizará una operación distinta::

	        if id_campo == 'id_periodo_academico':
	            request.session['periodoAcademico'] = PeriodoAcademico.objects.get(id=int(valor_campo))                
	            id = 'id_periodo_evaluacion'
	            objetos = PeriodoEvaluacion.objects.filter(periodoAcademico__id=int(valor_campo)).all()
	            valores = [dict(id=o.id, valor=o.nombre) for o in objetos]
	        elif id_campo == 'id_periodo_evaluacion':
	            request.session['periodoEvaluacion'] = PeriodoEvaluacion.objects.get(id=int(valor_campo))
	            id = 'id_area'
	            objetos = PeriodoEvaluacion.objects.get(id=int(valor_campo)).areasSGA.all()
	            valores = [dict(id=o.siglas, valor=o.nombre) for o in objetos]
	        elif id_campo == 'id_area':
	            # request.session['area'] = AreaSGA.objects.get(siglas=valor_campo)
	            request.session['area'] = valor_campo
	            id = 'id_carrera'
	            objetos = EstudianteAsignaturaDocente.objects.filter(
	                estudiante__periodoAcademico=request.session['periodoAcademico']).filter(
	                asignaturaDocente__asignatura__area=valor_campo).values_list(
	                'asignaturaDocente__asignatura__carrera', flat=True).distinct()
	            for o in objetos:
	                carrera = o.encode('utf-8') if isinstance(o, unicode) else o
	                valores.append(dict(id=carrera, valor=carrera))
	        
Se presentan dos bifuraciones dependiendo del menú::

	        elif id_campo == 'id_carrera':
	            request.session['carrera'] = valor_campo
	            id = campo_siguiente
	            if campo_siguiente == 'id_semestre':
	                objetos = EstudianteAsignaturaDocente.objects.filter(
	                    estudiante__periodoAcademico=request.session['periodoAcademico']).filter(
	                    asignaturaDocente__asignatura__area=request.session['area']).filter(
	                    asignaturaDocente__asignatura__carrera=valor_campo).values_list(
	                    'asignaturaDocente__asignatura__semestre', flat=True).distinct()
	                for o in objetos:
	                    semestre = o.encode('utf-8') if isinstance(o, unicode) else o
	                    valores.append(dict(id=semestre, valor=semestre))
	            elif campo_siguiente == 'id_docente':
	                objetos = AsignaturaDocente.objects.filter(
	                    docente__periodoAcademico=request.session['periodoAcademico']).filter(
	                    asignatura__area=request.session['area']).filter(
	                    asignatura__carrera=valor_campo)
	                docentes = set([o.docente for o in objetos])
	                valores = [dict(id=d.id, valor=d.__unicode__()) for d in docentes]
	        elif id_campo == 'id_semestre':
	            request.session['semestre'] = valor_campo
	            id = 'id_paralelo'
	            objetos = EstudianteAsignaturaDocente.objects.filter(
	                estudiante__periodoAcademico=request.session['periodoAcademico']).filter(
	                asignaturaDocente__asignatura__area=request.session['area']).filter(
	                asignaturaDocente__asignatura__carrera=request.session['carrera']).filter(
	                asignaturaDocente__asignatura__semestre=valor_campo).values_list(
	                'asignaturaDocente__asignatura__paralelo', flat=True).distinct()
	            for o in objetos:
	                paralelo = o.encode('utf-8') if isinstance(o, unicode) else o
	                valores.append(dict(id=paralelo, valor=paralelo))

	        resultado = {'id':id, 'valores':valores}
	        return simplejson.dumps(resultado)

Se genera un logg de error en caso de presentarse una excepción::

	    except Exception, ex:
	        logg.error("Error en menu_academico_ajax: " + str(ex))
	        return ""

.. _views-controllers-calculation_presentation_results:

Cálculo y Presentación de resultados
=====================================

.. _views-controllers-calculation_presentation_results-results_career:

Resultados por Carrera
----------------------

El presente método se encarga de obtener los resultados de las evaluaciones tomando según la carrera. para su funcionamiento uno de sus parámetros en el número de la carrera(``num_carrera``) que lo identifica.
Se trabaja con las carreras cuya direccion esta a cargo del docente almacenadas en la vista previa ``index``::

	def resultados_carrera(request, num_carrera):
	    datos = dict()

	    try:
	        carreras_director = request.session['carreras_director']
	        for cd in carreras_director:
	            if cd['num_carrera'] == int(num_carrera):
	                carrera = cd['nombre']
	                area_siglas = cd['area']
	                break
	        else:
	            logg.error('No hay carreras para este docente director')
	        request.session['carrera'] = carrera
	        request.session['area'] = area_siglas

Se hace uso del objeto AreaSGA::

	        area = AreaSGA.objects.get(siglas=area_siglas)
	        periodoAcademico = Configuracion.getPeriodoAcademicoActual()
	        # Periodos de Evaluacion del Periodo Academico Actual 
	        periodosEvaluacion = area.periodosEvaluacion.filter(periodoAcademico=periodoAcademico)
	        form = forms.Form()

Se selecciona solo los peridos de evaluación en los que se encuentra el area del docente director::

	        form.fields['periodo_evaluacion'] = forms.ModelChoiceField(
	            queryset=area.periodosEvaluacion.filter(periodoAcademico=periodoAcademico)
	            )
	        form.fields['periodo_evaluacion'].label = 'Periodo de Evaluación'
	        datos = dict(form=form, title='>> Coordinador Carrera ' + carrera )
Excepción en caso de error::

	    except Exception, ex:
	        logg.error("Error :" + str(ex))

Se renderiza el template :ref:`template-user-menu_resultados_carrera.html`::

	    return render_to_response("app/menu_resultados_carrera.html", datos, context_instance=RequestContext(request))



.. _views-controllers-calculation_presentation_results-menu_results_carreer:

Menú de resultados por Carrera
------------------------------

Su funcionamiento consiste en generar el menu de opciones para reportes de acuerdo al periodo de evaluación y su tipo de tabulacion especificamente. Llamado con AJAX::

	def menu_resultados_carrera(request, id_periodo_evaluacion):
	    try:
	        periodoEvaluacion=PeriodoEvaluacion.objects.get(id=id_periodo_evaluacion)
	        tabulacion = Tabulacion.objects.get(periodoEvaluacion=periodoEvaluacion)

Para los Docentes Coordinadores de Carrera::

	        if request.session.has_key('carreras_director'):
	            carrera = request.session['carrera']
	            area = request.session['area']

Para la Comision de Evaluacion (Administracion)::

	        else:
	            area = request.GET['area']
	            carrera = request.GET['carrera']

Especifico para el Tipo de Tabulacion - Periodo de Evaluacion. Según el tipo de evaluación se formatean los formularios de administración:  :ref:`template-admin-formulario_ese2012.html`, :ref:`template-admin-formulario_eaad2012.html` y :ref:`template-admin-formulario_edd2013.html`::

	        if tabulacion.tipo == 'ESE2012':
	            tabulacion = TabulacionSatisfaccion2012(periodoEvaluacion)
	            form = ResultadosESE2012Form(tabulacion, area, carrera)
	            formulario_formateado = render_to_string("admin/app/formulario_ese2012.html", dict(form=form))
	        elif tabulacion.tipo == 'EAAD2012': 
	            tabulacion = TabulacionAdicionales2012(periodoEvaluacion)
	            form = ResultadosEAAD2012Form(tabulacion, area, carrera)            
	            formulario_formateado = render_to_string("admin/app/formulario_eaad2012.html", dict(form=form))
	        elif tabulacion.tipo == 'EDD2013':
	            tabulacion = TabulacionEvaluacion2013(periodoEvaluacion)
	            form = ResultadosEDD2013Form(tabulacion, area, carrera)
	            formulario_formateado = render_to_string("admin/app/formulario_edd2013.html", dict(form=form))
	        return HttpResponse(formulario_formateado)

Excepciones::

	    except PeriodoEvaluacion.DoesNotExist:
	        logg.error(u"No Existe el Periodo de Evaluación: {0}".format(id_periodo_evaluacion))
	    except Exception, ex:
	        logg.error('Error en vista menu_resultados_carrera: ' + str(ex))
	        


.. _views-controllers-calculation_presentation_results-results_presentation:

Presentación de Resultados
--------------------------

Método orientado a realizar la presentación de los resultados obtenidos::

	def mostrar_resultados(request):

Se hace uso de unas cuantas etiquetas HTML para mejor presentación::

	    if not (request.POST.has_key('periodo_evaluacion') and request.POST.has_key('opciones')):
	        return HttpResponse("<h2> Tiene que elegir las Opciones de Resultados </h2>")
	    id_periodo = request.POST['periodo_evaluacion']
	    if id_periodo == '':
	        return HttpResponse("<h2> Tiene que elegir el Periodo de Evaluación </h2>")

Se obtiene la opcion generica para cualquier tipo de evaluacion::

	    opcion = request.POST['opciones']
	    periodoEvaluacion=PeriodoEvaluacion.objects.get(id=int(id_periodo))
	    tabulacion = periodoEvaluacion.tabulacion

**Encuesta de Satisfaccion Estudiantil 2012**::

	    if tabulacion.tipo == 'ESE2012':
	        tabulacion = TabulacionSatisfaccion2012(periodoEvaluacion)
	        metodo =  [c[2] for c in tabulacion.calculos if c[0] == opcion][0]
	        titulo = request.session['area'] + '<br/> <b>' + request.session['carrera'] + '</b><br/>'
	        titulo += [c[3] for c in tabulacion.calculos if c[0] == opcion][0]
	        resultados = {}

Por docente::

	        if opcion == 'a':
	            id_docente = request.POST['docentes']
	            if id_docente != '':
	                titulo += u': <b>{0}</b>'.format(DocentePeriodoAcademico.objects.get(id=int(id_docente)))
	                resultados = metodo(request.session['area'], request.session['carrera'], int(id_docente))
Por campos::

	        elif opcion == 'c':
	            id_seccion = request.POST['campos']
	            if id_seccion != '':
	                seccion = Seccion.objects.get(id=int(id_seccion))
	                titulo += u': <b>{0}</b>'.format(seccion.titulo)
	                resultados = metodo(request.session['area'], request.session['carrera'], int(id_seccion))
	                if seccion.orden == 4:
	                    datos = dict(resultados=resultados, titulo=titulo)
	                    return render_to_response('app/imprimir_otros_ese2012.html', datos,
	                                              context_instance=RequestContext(request));
Por indicadores::

	        elif opcion == 'd':
	            id_pregunta = request.POST['indicadores']
	            if id_pregunta != '':
	                titulo += u': <b>{0}</b>'.format(Pregunta.objects.get(id=int(id_pregunta)).texto)
	                resultados = metodo(request.session['area'], request.session['carrera'], int(id_pregunta))

Para el resto de casos::

	        else:
	            resultados = metodo(request.session['area'], request.session['carrera'])
	        if resultados:
	            resultados['titulo'] = titulo
	        plantilla = 'app/imprimir_resultados_ese2012.html'

**Evaluacion de Actividades Adiconales a la Docencia 2011 - 2012**::

	    if tabulacion.tipo == 'EAAD2012':

Nombre completo del Area para su presentacion en el reporte::

	        area = AreaSGA.objects.get(siglas=request.session['area']).nombre
	        carrera = request.session['carrera']
	        tabulacion = TabulacionAdicionales2012(periodoEvaluacion)
	        metodo =  [c[2] for c in tabulacion.calculos if c[0] == opcion][0]

Por docente::

	        resultados = {}
	        if opcion == 'a':
	            id_docente = request.POST['docentes']
	            if id_docente != '':
	                docente = DocentePeriodoAcademico.objects.get(id=int(id_docente))
	                # Referencia a lo que devuelve el metodo especifico invocado sobre la instancia de Tabulacion 
	                resultados = metodo(request.session['area'], request.session['carrera'], int(id_docente))
	        elif opcion == 'z':
	            pass

Para el resto de casos::

	        else:
	            resultados = metodo(request.session['area'], request.session['carrera'])
	        if resultados:
	            resultados['docente'] = docente
	            resultados['carrera'] = carrera
	            resultados['area'] = area
	        plantilla = 'app/imprimir_resultados_eaad2012.html'

**Evaluacion del Desempenio Docente 2012 - 2013**::

	    if tabulacion.tipo == 'EDD2013':
	        if opcion  == 'c' and not request.user.is_staff:
	            return HttpResponse("<h2> Ud no tiene permisos para revisar este reporte </h2>")
	        codigos_filtro = {'a' : '', 'b' : 'CPF', 'c' : 'CPG', 'd' : 'PV', 'e' : 'sugerencias'}
	        objeto_area = AreaSGA.objects.get(siglas=request.session['area'])

Nombre completo del Area para su presentacion en el reporte::

	        area = objeto_area.nombre
	        area_siglas = request.session['area']
	        carrera = request.session['carrera']
	        tabulacion = TabulacionEvaluacion2013(periodoEvaluacion)
	        metodo =  [c[2] for c in tabulacion.calculos if c[0] == opcion][0]
	        filtro = request.POST['filtros']
	        filtro = codigos_filtro[filtro]

Por docente::

	        resultados = {}
	        if opcion == 'a':
	            id_docente = request.POST['docentes']
	            if id_docente != '':
	                docente = DocentePeriodoAcademico.objects.get(id=int(id_docente))

Referencia a lo que devuelve el metodo especifico invocado sobre la instancia de Tabulacion::

	                resultados = metodo(request.session['area'], request.session['carrera'], int(id_docente), filtro)
	                resultados['docente'] = docente
	                resultados['carrera'] = carrera
	                resultados['area'] = area

Por Carrera::

	        elif opcion == 'b':
	            resultados = metodo(request.session['area'], request.session['carrera'], filtro)
	            resultados['carrera'] = carrera
	            resultados['area'] = area

Por Area::

	        elif opcion == 'c':
	            resultados = metodo(request.session['area'], filtro)
	            resultados['area'] = area

Para el resto de casos::

	        else:
	            resultados = metodo(request.session['area'], request.session['carrera'], filtro)

Posicion para ubicar el promedio por componente en la plantilla.

Si se trata del Instituto de Idiomas::

	        if resultados.get('promedios_componentes', None) and objeto_area.id == 6:

	            resultados['promedios_componentes']['CPF'].update({'fila' : 8})
	            resultados['promedios_componentes']['CPG'].update({'fila' : 23})
	            resultados['promedios_componentes']['PV'].update({'fila' : 29})

Para el resto de Areas::

	        elif resultados.get('promedios_componentes', None):

	            resultados['promedios_componentes']['CPF'].update({'fila' : 10})
	            resultados['promedios_componentes']['CPG'].update({'fila' : 27})
	            resultados['promedios_componentes']['PV'].update({'fila' : 33})

Se trata de reporte de sugerencias se escoge entre las plantillas :ref:`template-user-imprimir_sugerencias_edd2013.html`, :ref:`template-user-imprimir_resultados_edd2013.html`::

	        if filtro == 'sugerencias':
	            plantilla = 'app/imprimir_sugerencias_edd2013.html'
	        else:
	            plantilla = 'app/imprimir_resultados_edd2013.html'

Se formatea la vista::

	    return render_to_response(plantilla, resultados, context_instance=RequestContext(request));


.. _views-controllers-calculation_presentation_results-results_admin:

Resultados para los Administradores
-----------------------------------

Manejo de resultados para los administradores::

	def resultados(request):
	    datos = dict(form=ResultadosForm())

Se formatea la vista de administración :ref:`template-admin-menu_resultados.html`::

	    return render_to_response('admin/app/menu_resultados.html', datos, context_instance=RequestContext(request))
	    
