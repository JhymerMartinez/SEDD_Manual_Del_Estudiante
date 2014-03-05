.. _admin-title:

************************
Manejo de Administración
************************

.. _admin-intro:

Introducción
============

El Sitio de administración de Django constituye una Web, limitada a administradores, que permite agregar, editar y eliminar el contenido del sitio. También nos ayuda con la gestión de usuarios si usamos los modelos propios de django


.. note::
	Para ayuda revisar `Curso Django: web de administración <http://www.cristalab.com/tutoriales/python-en-la-web-con-django-vi-web-de-administracion-c104286l/>`_ o en la docuemntación oficial: `The Django admin site <https://docs.djangoproject.com/en/1.5/ref/contrib/admin/>`_.

.. _admin-imports:

Importaciones Necesarias
========================

Importación del módulo principal para la administración::

	from django.contrib import admin

Para el uso de mensajes personalizados::

	from django.contrib import messages

Uso de formularios::

	from django import forms

Importación del paquete completo de :ref:`models-title`::

	from proyecto.app import models

Importación del :ref:`form-EstudianteAsignaturaDocenteAdminForm`::

	from proyecto.app.forms import EstudianteAsignaturaDocenteAdminForm

Importación del :ref:`form-AsignaturaDocenteAdminForm`::

	from proyecto.app.forms import AsignaturaDocenteAdminForm

Para uso de logs::

	import logging
	logg = logging.getLogger('logapp')



.. _admin-PreguntaAdmin:

Clase PreguntaAdmin
===================

Derivada de la :ref:`class-Pregunta` con el fín de agregar nuevas funcionalidades específicas de administración::

	class PreguntaAdmin(admin.ModelAdmin):
	    inlines = (ItemPreguntaEnLinea,)
	    fields = ('seccion','orden', 'codigo', 'texto', 'descripcion', 'tipo')
	    search_fields = ('texto', 'descripcion')
	    list_filter = ('seccion__cuestionario__periodoEvaluacion__periodoAcademico', 
	                   'seccion__cuestionario__periodoEvaluacion', 'seccion__cuestionario')
	    list_per_page = 30

Método sobreescrito.
Se crean y se agregan los items por defecto solo cuando se trata de la creación de una nueva pregunta::

	    def save_model(self, request, obj, form, change):
	        if change:
	            obj.save()
	            return

``request`` es un objeto de tipo WSGIRequest::

	        tipo = request.POST['tipo']
	        
Solo sí : **Tipo de pregunta:  SeleccionUnica**::

	        if tipo=='2':
	            longitud = request.POST['longitud']
	            numeracion = request.POST['numeracion']
	            if longitud == '' or numeracion == '':
	                return
	            # Si se trata de una pregunta de Seleccion por predeterminada
	            obj.save()
	            longitud = int(longitud)
	            if numeracion == '1':
	                for n in range(1, longitud+1):
	                    item = models.ItemPregunta(texto=str(n), pregunta=obj, orden=n)
	                    item.save()
	            elif numeracion.lower() == 'a':
	                for i,n in enumerate(letters):
	                    item = models.ItemPregunta(texto=n, pregunta=obj, orden=i)
	                    item.save()
	        # Cualquier otro tipo de pregunta
	        else:
	            obj.save()
	          
	    class Media:
	        js = ['js/tiny_mce/tiny_mce.js', 'js/tmce_config.js',]



.. _admin-SeccionAdmin:

Clase SeccionAdmin
==================

::

	class SeccionAdmin(admin.ModelAdmin):
	    inlines = (SubSeccionEnLinea,)
	    fields = ('cuestionario', 'codigo', 'nombre', 'titulo', 'descripcion', 'orden', 'ponderacion')
	    search_fields = ('nombre',)
	    list_filter = ('cuestionario__periodoEvaluacion__periodoAcademico', 'cuestionario__periodoEvaluacion', 
	                   'cuestionario', 'superseccion')

.. _admin-CuestionarioAdmin:

Clase CuestionarioAdmin
=======================

::

	class CuestionarioAdmin(admin.ModelAdmin):
	    actions = ['clonar_cuestionario']
	    ###inlines = ('SeccionEnLinea',)
	    save_as = True
	    list_filter = ('periodoEvaluacion__periodoAcademico', 'periodoEvaluacion')

	    # Accion para copiar un cuestionario en el Admin
	    def clonar_cuestionario(self, request, queryset):
	        cantidad = queryset.count()
	        if cantidad == 1:
	            objeto = queryset.all()[0]
	            objeto.clonar()
	            mensaje = u'Se ha clonado satisfactoriamente el Cuestinario'
	            self.message_user(request, mensaje)
	        else:
	            mensaje = u'Debe seleccionar uno solo Cuestionario! Ha seleccionado %s' % (str(cantidad))
	            messages.error(request, mensaje)

	    clonar_cuestionario.short_description = "Crear Copia de Cuestionario"
	    
	    class Media:
	        js = ['js/tiny_mce/tiny_mce.js', 'js/tmce_config.js',]



.. _admin-PeriodoEvaluacionAdmin:

Clase PeriodoEvaluacionAdmin
============================
::

	class PeriodoEvaluacionAdmin(admin.ModelAdmin):
	    filter_horizontal = ('areasSGA',)
	    list_filter = ('periodoAcademico',)



.. _admin-PeriodoAcademicoAdmin:

Clase PeriodoAcademicoAdmin
===========================

::

	class PeriodoAcademicoAdmin(admin.ModelAdmin):
	    filter_horizontal = ('ofertasAcademicasSGA',)



.. _admin-EstudiantePeriodoAcademicoAdmin:

Clase EstudiantePeriodoAcademicoAdmin
=====================================

::

	class EstudiantePeriodoAcademicoAdmin(admin.ModelAdmin):
	    list_display = ('cedula', '__unicode__', 'periodoAcademico')
	    list_per_page = 20
	    search_fields = ('usuario__username','usuario__cedula','usuario__last_name','usuario__first_name')
	    inlines = (EstudianteAsignaturaDocenteEnLinea,)
	    raw_id_fields = ('usuario',)
	    list_filter = ('periodoAcademico',)

.. _admin-EstudianteAsignaturaDocenteAdmin:

Clase EstudianteAsignaturaDocenteAdmin
======================================

::

	class EstudianteAsignaturaDocenteAdmin(admin.ModelAdmin):
	    raw_id_fields = ('asignaturaDocente','estudiante')
	    search_fields = ('estudiante__usuario__cedula', 'estudiante__usuario__first_name',
	                     'estudiante__usuario__last_name', 'asignaturaDocente__asignatura__carrera',
	                     'asignaturaDocente__asignatura__semestre' )
	    list_per_page = 30
	    list_display = ('estudiante', 'get_asignatura', 'get_area', 'get_carrera', 'get_semestre', 'get_paralelo')
	    list_filter = ('estudiante__periodoAcademico',)

	    # Permitir filtros
	    def lookup_allowed(self, key, value):
	        if key in ('asignaturaDocente__asignatura__semestre',):
	            return True
	        return super(EstudianteAsignaturaDocenteAdmin, self).lookup_allowed(key, value)

.. _admin-AsignaturaDocenteAdmin:

Clase AsignaturaDocenteAdmin
============================

::

	class AsignaturaDocenteAdmin(admin.ModelAdmin):
	    actions = ['clonar_asignaturadocente']
	    raw_id_fields = ('asignatura','docente')
	    search_fields = ('docente__usuario__cedula', 'docente__usuario__first_name',
	                     'docente__usuario__last_name','asignatura__carrera', 'asignatura__nombre',
	                     'asignatura__idSGA' )
	    list_per_page = 30
	    list_display = ( 'get_nombre_corto', 'get_carrera', 'get_semestre', 'get_paralelo')
	    fields = ('docente','asignatura')
	    list_filter = ('docente__periodoAcademico',)

	    def validar_clonar(self, request, queryset):
	        paralelo = request.POST['paralelo']
	        mensaje = ''
	        if paralelo == '':
	            mensaje = 'Debe especificar un Paralelo'
	            messages.error(request, mensaje)
	            return False
	        cantidad = queryset.count()
	        if cantidad > 1:
	            mensaje = u'Debe seleccionar uno solo Objeto, ha seleccionado %s' % (str(cantidad))
	            messages.error(request, mensaje)
	            return False
	        asignaturaDocente = queryset.all()[0]
	        if paralelo == asignaturaDocente.asignatura.paralelo:
	            mensaje = "Se trata del mismo Paralelo, debe clonar la AsignaturaDocente en uno diferente"
	            messages.error(request, mensaje)
	            return False
	        asignatura = asignaturaDocente.asignatura
	        # Paralelo que tiene el semestre al cual pertenece AsignaturaDocente
	        consulta = models.Asignatura.objects.filter(area=asignatura.area, carrera=asignatura.carrera).filter(semestre=asignatura.semestre)
	        paralelos = consulta.values_list('paralelo', flat=True).distinct()
	        if paralelo not in paralelos:
	            mensaje = "No existe este paralelo en el Semestre {0} de la Carrera {1}".format(asignatura.semestre, asignatura.carrera)
	            messages.error(request, mensaje)
	            return False
	        return True
	        
	    def clonar_asignaturadocente(self, request, queryset):
	        """
	        Permite crear una Asignatura con el Docente respectivo pero en otro paralelo
	        de la misma carrera y el mismo semestre. Funcionalidad creada especialmente para
	        crear aquellas unidades que no existen en el SGA en el caso de las carreras del
	        Area de la Salud Humana.
	        """
	        if self.validar_clonar(request, queryset):
	            paralelo = request.POST['paralelo']
	            asignaturaDocente = queryset.all()[0]
	            asignatura = asignaturaDocente.asignatura
	            docente = asignaturaDocente.docente
	            """ Obtener los estudiantes del paralelo que no tiene esta asignatura """
	            consulta = models.EstudianteAsignaturaDocente.objects.filter(
	                asignaturaDocente__asignatura__area=asignatura.area).filter(
	                asignaturaDocente__asignatura__carrera=asignatura.carrera).filter(
	                asignaturaDocente__asignatura__semestre=asignatura.semestre).filter(
	                asignaturaDocente__asignatura__paralelo=paralelo # El paralelo en el cual se clonará 
	            )
	            estudiantes = set([ead.estudiante for ead in consulta.all()])
	            """ Creación de la Nueva Asignatura y AsignaturaDocente """
	            # Se clona el objeto Asignatura , será una nueva entidad
	            asignatura.pk = None
	            id_unidad = asignatura.idSGA.split(':')[0]
	            asignatura.idSGA = u"{0}:SEDD".format(id_unidad)
	            asignatura.paralelo = paralelo # El Paralelo en el cual se clonará
	            asignatura.nombre = u"{0} (Clonado)".format(asignatura.nombre)
	            # Grabado en la base de datos
	            asignatura.save()
	            # Se graba el nuevo Asignatura-Docente
	            models.AsignaturaDocente.objects.create(docente=docente, asignatura=asignatura)
	            """ Se asigna la nueva Asignatura-Docente a todos los estudiantes del Paralelo """
	            for e in estudiantes:
	                models.EstudianteAsignaturaDocente.objects.create(estudiante=e, asignaturaDocente=asignaturaDocente)
	            mensaje = "Se agregó la asignatura {0} al paralelo {1} (con {2} estudiantes)".format(asignatura,paralelo,len(estudiantes))
	            self.message_user(request, mensaje)        
	                        
	    clonar_asignaturadocente.short_description = u'Clonar Asignatura-Docente en otro Paralelo'




.. _admin-DocentePeriodoAcademicoAdmin:

Clase DocentePeriodoAcademicoAdmin
==================================

::

	class DocentePeriodoAcademicoAdmin(admin.ModelAdmin):
	    list_display = ('cedula', '__unicode__', 'periodoAcademico')
	    list_per_page = 20
	    search_fields = ('usuario__cedula','usuario__last_name','usuario__first_name') 
	    inlines = (AsignaturaDocenteEnLinea,)
	    raw_id_fields = ('usuario',)
	    list_filter = ('periodoAcademico',)


.. _admin-DireccionCarreraAdmin:

Clase DireccionCarreraAdmin
===========================

::

	class DireccionCarreraAdmin(admin.ModelAdmin):
	    actions = ['actualizar_periodo_academico']
	    list_filter = ('director__periodoAcademico',)
	    raw_id_fields = ('director',)

	    def actualizar_periodo_academico(self, request, queryset):
	        """
	        Promueve los directores de carrera al presente Periodo Academico
	        seleccionando el mismo docente pero del periodo actual.
	        """
	        periodoAcademico = models.Configuracion.getPeriodoAcademicoActual()
	        cont = 0
	        for dc in queryset.all():
	            try:
	                docente_actual = models.DocentePeriodoAcademico.objects.get(
	                    usuario=dc.director.usuario, periodoAcademico=periodoAcademico)
	                if docente_actual:
	                    dc.director = docente_actual
	                    cont = cont + 1
	                    dc.save()
	            except models.DocentePeriodoAcademico.DoesNotExist:
	                logg.error('Docente {0} no encontrado en periodo Actual {1}'.format(
	                    dc.director, periodoAcademico))
	        self.message_user(request,'Actualizadas {0} Direcciones de Carrera'.format(cont))

	    actualizar_periodo_academico.short_description = u'Actualizar Coordinadores al Periodo Academico Actual'


.. _admin-AsignaturaAdmin:

Clase AsignaturaAdmin
=====================

::

	class AsignaturaAdmin(admin.ModelAdmin):
	    list_display = ('carrera','semestre','paralelo', '__unicode__')
	    list_display_links = ('__unicode__',)
	    search_fields = ('idSGA','area','carrera','semestre','paralelo','nombre',)
	    list_filter = ('periodoAcademico', 'area','carrera','semestre','tipo',)
	    list_per_page = 20
	    # TODO: combobox
	    #readonly_fields = ('area','carrera','semestre','paralelo','tipo',)
	    fieldsets = (
	        ('Datos Académicos', {
	            'fields': ('area', 'carrera', 'semestre', 'paralelo')
	            }),
	        ('Asignatura', {
	            'fields': ('nombre', 'tipo', 'creditos')
	            }),
	        ('Duracion', {
	            'fields': ('duracion', 'inicio', 'fin', 'periodoAcademico')
	            }),        
	        
	        )
	    

.. _admin-UsuarioAdmin:

Clase UsuarioAdmin
==================

::

	class UsuarioAdmin(admin.ModelAdmin):
	    search_fields = ('username','cedula')
	    list_display = ('username','cedula','get_full_name',)
	    list_display_links = ('cedula','get_full_name',)
	    list_per_page = 20


.. _admin-ConfiguracionAdmin:

Clase ConfiguracionAdmin
========================

::

	class ConfiguracionAdmin(admin.ModelAdmin):
	    actions = None
	    
	    def has_add_permission(self, request):
	        return False

	    def has_delete_permission(self, request, obj=None):
	        return False



.. _admin-EvaluacionAdmin:

Clase EvaluacionAdmin
=====================

::

	class EvaluacionAdmin(admin.ModelAdmin):
	    # Si no se especifica fields no se muestra la vista de modificacion
	    search_fields = ('estudianteAsignaturaDocente__estudiante__usuario__cedula',
	                     'estudianteAsignaturaDocente__asignaturaDocente__docente__usuario__cedula',
	                     'docentePeriodoAcademico__usuario__cedula',
	                     'directorCarrera__usuario__cedula')
	    list_per_page = 30
	    list_filter = ('cuestionario__periodoEvaluacion__periodoAcademico', 'cuestionario__periodoEvaluacion', 
	                   'cuestionario__informante',  'fechaInicio', 'fechaFin',)
	    date_hierarchy = 'fechaFin'
	    fields = ('cuestionario', 'fechaFin', 'horaFin')
	    readonly_fields = ('cuestionario', 'fechaFin', 'horaFin')

	    def has_add_permission(self, request):
	        return False

.. _admin-ResultadosAdmin:

Clase ResultadosAdmin
=====================

::

	class ResultadosAdmin(admin.ModelAdmin):
	    pass

.. _admin-TabulacionAdmin:

Clase TabulacionAdmin
=====================

::

	class TabulacionAdmin(admin.ModelAdmin):
	    list_filter = ('periodoEvaluacion__periodoAcademico', 'periodoEvaluacion')

.. _admin-TipoInformanteAdmin:
Clase TipoInformanteAdmin
=========================

::

	class TipoInformanteAdmin(admin.ModelAdmin):
	    pass



.. _admin-online:

Clases y Ofertas en EnLinea
===========================

A continuación una lista de clases que se caracterizan por adquirir :ref:`models-title` del paquete ``models`` con el fin de adicionar ciertos atributos 

.. _admin-online-ItemPreguntaEnLinea:

Clase ItemPreguntaEnLinea
-------------------------

Esta clase obtiene el modelo de la :ref:`class-ItemPregunta` y se aumenta ciertos atributos::

	class ItemPreguntaEnLinea(admin.StackedInline):
	    model = models.ItemPregunta
	    extra = 1


.. _admin-online-PreguntaEnLinea:

Clase PreguntaEnLinea
---------------------

Esta clase obtiene el modelo de la :ref:`class-Pregunta` y se aumenta ciertos atributos::

	class PreguntaEnLinea(admin.TabularInline):
	    model = models.Pregunta
	    extra = 1



.. _admin-online-SubSeccionEnLinea:

Clase SubSeccionEnLinea
-----------------------

Esta clase obtiene el modelo de la :ref:`class-Seccion` y se aumenta ciertos atributos::

	class SubSeccionEnLinea(admin.TabularInline):
	    model = models.Seccion
	    extra = 1
	    verbose_name = u'Subsección'
	    verbose_name_plural = 'Subsecciones'
	    exclude = ('cuestionario',)



.. _admin-online-OfertaAcademicaSGAEnLinea:

Clase OfertaAcademicaSGAEnLinea
-------------------------------

Esta clase obtiene el modelo de la :ref:`class-OfertaAcademicaSGA` y se aumenta ciertos atributos::

	class OfertaAcademicaSGAEnLinea(admin.TabularInline):
	    model = models.OfertaAcademicaSGA
	    extra = 1



.. _admin-online-EstudianteAsignaturaDocenteEnLinea:

Clase EstudianteAsignaturaDocenteEnLinea
----------------------------------------

Esta clase obtiene el modelo de la :ref:`class-EstudianteAsignaturaDocente` y se aumenta ciertos atributos::

	class EstudianteAsignaturaDocenteEnLinea(admin.StackedInline):
	    model = models.EstudianteAsignaturaDocente
	    extra = 1
	    verbose_name = 'Asignaturas de Estudiante'
	    raw_id_fields = ('asignaturaDocente',)
	    # fields = ('carrera','semestre','paralelo','asignaturaDocente')



.. _admin-online-AsignaturaDocenteEnLinea:

Clase AsignaturaDocenteEnLinea
------------------------------

Esta clase obtiene el modelo de la :ref:`class-AsignaturaDocente` y se aumenta ciertos atributos::

	class AsignaturaDocenteEnLinea(admin.TabularInline):
	    model = models.AsignaturaDocente
	    form = AsignaturaDocenteAdminForm
	    extra = 1
	    verbose_name = 'Asignaturas de Docente'
	    raw_id_fields = ('asignatura',)



.. _admin-online-ContestacionEnLinea:

Clase ContestacionEnLinea
-------------------------

Esta clase obtiene el modelo de la :ref:`class-Contestacion` y se aumenta ciertos atributos::

	class ContestacionEnLinea(admin.TabularInline):
	    model = models.Contestacion
	    extra = 15
	    verbose_name = 'Respuesta'



.. _admin-register:

Agregación de Clases a la Administración
========================================

En esta sección se asignan las distintas clases utilizadas para realizar operaciones de administración::

	admin.site.register(models.Cuestionario,CuestionarioAdmin)
	admin.site.register(models.Seccion,SeccionAdmin)
	admin.site.register(models.Pregunta,PreguntaAdmin)
	admin.site.register(models.PeriodoAcademico,PeriodoAcademicoAdmin)
	admin.site.register(models.PeriodoEvaluacion,PeriodoEvaluacionAdmin)
	admin.site.register(models.EstudiantePeriodoAcademico, EstudiantePeriodoAcademicoAdmin)
	admin.site.register(models.DocentePeriodoAcademico, DocentePeriodoAcademicoAdmin)
	admin.site.register(models.DireccionCarrera, DireccionCarreraAdmin)
	admin.site.register(models.EstudianteAsignaturaDocente, EstudianteAsignaturaDocenteAdmin)
	admin.site.register(models.AsignaturaDocente, AsignaturaDocenteAdmin)
	admin.site.register(models.Asignatura, AsignaturaAdmin)
	admin.site.register(models.Usuario, UsuarioAdmin)
	admin.site.register(models.Configuracion, ConfiguracionAdmin)
	admin.site.register(models.Evaluacion, EvaluacionAdmin)
	admin.site.register(models.Tabulacion, TabulacionAdmin)
	admin.site.register(models.TipoInformante, TipoInformanteAdmin)
	admin.site.register(models.Resultados, ResultadosAdmin)