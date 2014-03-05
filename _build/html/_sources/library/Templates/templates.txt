.. _templates-title:

**************
Las plantillas
**************


.. _templates-intro:

Introducción
============

Una plantilla de Django es una cadena de texto pensada para separar la presentación de un documento de sus datos. Una plantilla define variables y varios trozos de lógica básica (tags) que regulan la manera en que debería mostrarse el documento. Normalmente, las plantillas se usan para producir HTML, pero las plantillas de Django son igualmente capaces de generar cualquier formato basado en texto.

Para el presente sistema se uso las plantillas que por defecto nos facilita Django, usadas en la parte de administración de cada proyecto. Por lo general, una vez intalado Python y Django, dichas plantillas se encuentran ubicadas en ``C:\Python27\Lib\site-packages\django-1.5.2-py2.7.egg\django\contrib\admin\templates\admin``.

.. note::
	Para mas información visitar: `curso de Django en español <http://www.maestrosdelweb.com/editorial/curso-django-las-plantillas/>`_.


.. _template-base:

Plantilla Base
==============
Contiene el diseño básico de las pantallas que se presentarán al usuario:

.. code-block:: html

	{% load admin_static %}<!DOCTYPE html>
	<html lang="{{ LANGUAGE_CODE|default:"en-us" }}" {% if LANGUAGE_BIDI %}dir="rtl"{% endif %}>

Cabecera:

.. code-block:: html

	<head>
	<title>{% block title %}{% endblock %}</title>
	<link rel="stylesheet" type="text/css" href="{% block stylesheet %}{% static "admin/css/base.css" %}{% endblock %}" />
	{% block extrastyle %}{% endblock %}
	<!--[if lte IE 7]><link rel="stylesheet" type="text/css" href="{% block stylesheet_ie %}{% static "admin/css/ie.css" %}{% endblock %}" /><![endif]-->
	{% if LANGUAGE_BIDI %}<link rel="stylesheet" type="text/css" href="{% block stylesheet_rtl %}{% static "admin/css/rtl.css" %}{% endblock %}" />{% endif %}
	<script type="text/javascript">window.__admin_media_prefix__ = "{% filter escapejs %}{% static "admin/" %}{% endfilter %}";</script>
	{% block extrahead %}{% endblock %}
	{% block blockbots %}<meta name="robots" content="NONE,NOARCHIVE" />{% endblock %}
	</head>

Carga de `internationalization`::

	{% load i18n %}

Inicio del body:

.. code-block:: html
    :linenos:

	<body class="{% if is_popup %}popup {% endif %}{% block bodyclass %}{% endblock %}">
	<!-- Container -->
	<div id="container">
	    {% if not is_popup %}
	    <!-- Header -->
	    <div id="header">
	        <div id="branding">
	        {% block branding %}{% endblock %}
	        </div>
	        {% if user.is_active and user.is_staff %}
	        <div id="user-tools">
	            {% trans 'Welcome,' %}
	            <strong>{% filter force_escape %}{% firstof user.get_short_name user.get_username %}{% endfilter %}</strong>.
	            {% block userlinks %}
	                {% url 'django-admindocs-docroot' as docsroot %}
	                {% if docsroot %}
	                    <a href="{{ docsroot }}">{% trans 'Documentation' %}</a> /
	                {% endif %}
	                {% if user.has_usable_password %}
	                <a href="{% url 'admin:password_change' %}">{% trans 'Change password' %}</a> /
	                {% endif %}
	                <a href="{% url 'admin:logout' %}">{% trans 'Log out' %}</a>
	            {% endblock %}
	        </div>
	        {% endif %}
	        {% block nav-global %}{% endblock %}
	    </div>
	    <!-- END Header -->
	    {% block breadcrumbs %}
	    <div class="breadcrumbs">
	    <a href="{% url 'admin:index' %}">{% trans 'Home' %}</a>
	    {% if title %} &rsaquo; {{ title }}{% endif %}
	    </div>
	    {% endblock %}
	    {% endif %}
	    {% block messages %}
	        {% if messages %}
	        <ul class="messagelist">{% for message in messages %}
	          <li{% if message.tags %} class="{{ message.tags }}"{% endif %}>{{ message }}</li>
	        {% endfor %}</ul>
	        {% endif %}
	    {% endblock messages %}
	    <!-- Content -->
	    <div id="content" class="{% block coltype %}colM{% endblock %}">
	        {% block pretitle %}{% endblock %}
	        {% block content_title %}{% if title %}<h1>{{ title }}</h1>{% endif %}{% endblock %}
	        {% block content %}
	        {% block object-tools %}{% endblock %}
	        {{ content }}
	        {% endblock %}
	        {% block sidebar %}{% endblock %}
	        <br class="clear" />
	    </div>
	    <!-- END Content -->
	    {% block footer %}<div id="footer"></div>{% endblock %}
	</div>
	<!-- END Container -->
	</body>

Fin del html:

.. code-block:: html

	</html>


.. _template-admin:

Plantillas para Administración
==============================

.. _template-admin-signatures:

Asignaturas y docentes
----------------------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/admin/app/asignaturadocente/change_list.html``::

	{% extends "admin/change_list.html" %}
	{% load i18n %}

	{% block extrahead %}
	{{ block.super }}
	<script type="text/javascript">
	  (function($){
	      $(document).ready(function(){
		  ///$("#paralelo").hide();
		  $('#changelist-form .actions').bind('click', function(){
		      //var seleccion=$('#action option:selected').val();
		      alert(seleccion);
		  });
	      });
	  })(django.jQuery);
	</script>
	{% endblock %}

	{% block result_list %}
	<!-- Para clonar una Asginatura Docente en otro Paralelo -->
	<span id="paralelo"> <b> Paralelo: </b> <input type="text" name="paralelo"/>  </span>

	{{ block.super }}

	{% endblock %}



.. _template-admin-evaluation:

Evaluaciones
------------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/admin/app/evaluacion/change_form.html``::

	{% extends "admin/change_form.html" %}
	{% load i18n %}

	{% block extrahead %}
	{{ block.super }}
	<style type="text/css">

	</style>
	{% endblock %}

	{% block content_title %}
	<!-- Se anula el tituto por defecto de la pagina -->
	{% endblock %}

	{% block field_sets %}
	{{ original.cuestionario.informante }}: <b>{{ original.evaluador }}
	</b> Docente evaluado: <b>{{ original.evaluado }}</b>
	<p/>
	{{ block.super }}
	{{ formsets }}

	<table> 
	  {% for contestacion in original.contestaciones.all %}
	  <tr>
		<td rowspan="2" > {{ contestacion.get_pregunta.get_codigo }} </td> 
		<td> {{ contestacion.get_pregunta }} </td> 
	  </tr>
	  <tr>
		<td style="border-bottom: 1px dashed;"> Respuesta: <b> {{ contestacion.respuesta }} </b> </td>
	  </tr>
	  {% endfor %}
	</table>

	{% endblock %}

	{% block submit_buttons_bottom %}
	<!-- Se elimina los botones de grabacion (submit) -->
	{% endblock %}

.. ###########################################################

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/admin/app/evaluacion/change_list.html``::

	{% extends "admin/change_list.html" %}
	{% load i18n %}

	{% block extrahead %}
	{{ block.super }}
	<script language="javascript">
	  function resumen(){
	      var win = window.open("/admin/resumen/evaluaciones", "resumen", 
	  'height=500,width=800,resizable=yes,scrollbars=yes');
	      win.focus();
	  }
	</script>
	{% endblock%}

	{% block object-tools %}
	{{ block.super }}
	  <ul class="object-tools">
	    <li>
	      <a href="javascript:void(0)" onclick="resumen();">
	        Resumen de Evaluaciones
	      </a>
	    </li>
	  </ul>
	{% endblock %}



.. _template-admin-period:

Periodo académico
-----------------
Ubicación: dentro de ``{{TEMPLATE_DIRS}}/admin/app/periodoacademico/change_form.html``::

	{% extends "admin/change_form.html" %}
	{% load i18n %}

	{% block extrahead %}
	{{ block.super }}
	<script type="text/javascript">
	    (function($) {
	        $(document).ready(function($) {
		    var logjs = "";
		    /*
		      Funcionalidad muy 'pesada', se resolvió hacerla por scripts
		    $("#link_load_sga").click(function() {
			var r=confirm("Se va a cargar la información Académica del SGA");
			if (r == true){
			    $.ajax({
				url: "/sga/cargar_info_sga/{{ object_id }}",
				method: "GET",
				//data: "periodo_academico_id={{ object_id }}",
				success: function (data){
				    alert(data);
				},
				error: function(xhr,status,error){
				    alert("error: " + status + "-" + error);
				}
			    });
			}
		    });*/ 

		    $("#link_load_ofertas").click(function() {
			var r=confirm("Va a recargar ofertas académicas del SGA");
			if (r == true){
			    $.ajax({
				url: "/sga/cargar_ofertas_sga/{{ object_id }}",
				method: "GET",
				//data: "periodo_academico_id={{ object_id }}",
				success: function (data){
				    // Regresamos a la lista de Periodos Académicos
				    $(window.location).attr('href',"{% url 'admin:app_periodoacademico_changelist' %}")
				},
				error: function(xhr,status,error){
				    alert("error: " + status + "-" + error);
				}
			    });
			}
		    });
	         });
	    })(django.jQuery);
	</script>
	{% endblock %}

	{% block after_field_sets %}
	<br/>
	<br/>
	<ul class="object-tools">
	  <li> <a id="link_load_ofertas"  class="xxhistorylink"> Recargar Ofertas </a> </li>
	</ul>
	{% endblock %}

	{% block before_related_objects %}

	{% endblock %}



.. _template-admin-question:

Preguntas en general
--------------------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/admin/app/periodoacademico/change_form.html``::

	{% extends "admin/change_form.html" %}
	{% load i18n %}

	{% block extrahead %}

	{{ block.super }}

	<script type="text/javascript">
	  (function($){
	      $(document).ready(function($){
		  $('#id_tipo').change(function(){
		      var seleccion=$('#id_tipo option:selected').val();
		      // Tipo de pregunta: SeleccionUnica
		      if (seleccion == '2'){
			  $('#id_tipo').parent().append(
			      '<span id="activador"><input type="checkbox" id="tipo_predefinido"' + 
				  'name="tipo_predefinido" value="Valor">' + 
				  'Predeterminado </input></span>'
			  );
			  $('#tipo_predefinido').change(function(){
			      if ($('#tipo_predefinido').is(':checked')) {
				  $('#items-group').hide();
				  var contenido = '<span id="opciones_grupo">';
				  contenido += '<b> Tamaño: </b> <select name="longitud">' + 
				      '<option value="" selected > ----- </option> ';
				  for (var i=2; i<10; i++){
				      var selected = '';
				      if ( i==4 ){
					  selected = 'selected';
				      }
				      contenido += '<option value="' + i + '" ' + selected + '>' + i + '</option>';
				  }
				  contenido += '</select> <b> Numeración:</b>' + 
				      '<input type="radio" name="numeracion" value="a"> Letras </input>' +
				      '<input type="radio" name="numeracion" value="1" checked> Números </input>';
	 			  contenido += "</span>";
				  $('#id_tipo').parent().append(contenido);
			      } else {
				  $('#opciones_grupo').remove();
				  $('#items-group').show();
			      }
			  });
		      }else{
			  // Ocultar activador de SeleccionUnica por predeterminada
			  $('#activador').remove();
			  // Tipo de pregunta: Ensayo, no tiene items
			  if (seleccion == 1){
			      $('#items-group').hide();
			  } else{
			      // Cualquier otro tipo de pregunta
			      $('#items-group').show();
			  }
		      }
		  });
	      });
	  })(django.jQuery);
	</script>

	{% endblock%}


.. _template-admin-formulario_eaad2012.html:

Formularios eaad2012
--------------------
Ubicación: dentro de ``{{TEMPLATE_DIRS}}/admin/app/formulario_eaad2012.html``::

	<style type="text/css">
	input[type=radio] {
	    margin-top: 7px;

	}
	fieldset{
	    margin: 15px 0px 15px 0px;
	    width: 75%;
	    padding: 10px 20px 10px 20px;
	}
	#id_campos, #id_indicadores{
	    width: 75%;
	}

	</style>
	<fieldset>
	  <legend> Opciones de Resultados </legend>
	  <ol type="a">
	    <li>
	      <label for="id_opciones_0">
		<input type="radio" id="id_opciones_0" value="a" name="opciones" />  
		Resultados Evaluación Actividades Adicionales por Docente 
	      </label>
	    </li>
	    <ul>
	      <li> {{ form.docentes }} </li>
	    </ul>
	  
	    <!--li>
	      <label for="id_opciones_1">
		<input type="radio" id="id_opciones_1" value="b" name="opciones" />  
		Resultados Evaluación Actividades Adicionales por Carrera 
	      </label>
	    </li-->
	  </ol>
	</fieldset>

.. _template-admin-formulario_edd2013.html:

Formularios edd2013
--------------------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/admin/app/formulario_edd2013.html``::

	<style type="text/css">
	input[type=radio] {
	    margin-top: 7px;

	}
	fieldset{
	    margin: 15px 0px 15px 0px;
	    width: 75%;
	    padding: 10px 20px 10px 20px;
	}
	#id_campos, #id_indicadores{
	    width: 75%;
	}

	</style>
	<fieldset>
	  <legend> Opciones de Resultados </legend>

	  <table>
		<tr>
		  <!-- Columna 1 --> 
		  <td style="border-right: 1px solid;">
			<b>Seleccione el tipo de Reporte:</b>
			<ol type="a">
			  <li>
				<label for="id_opciones_0">
				  <input type="radio" id="id_opciones_0" value="a" name="opciones" checked="checked"/>  
				  Resultados POR DOCENTE 
				</label>
			  </li>
			  <ul>
				<li> {{ form.docentes }} </li>
			  </ul>
			  
			  <li>
				<label for="id_opciones_1">
				  <input type="radio" id="id_opciones_1" value="b" name="opciones" />  
				  Resultados POR CARRERA
				</label>
			  </li>

			  <li>
				<label for="id_opciones_2">
				  <input type="radio" id="id_opciones_2" value="c" name="opciones" />  
				  Resultados POR AREA
				</label>
			  </li>

			</ol>
		  </td>

		  <!-- Columna 2 -->
		  <td>
			<b> Seleccione el Campo Específico: </b>
			<ol type="a">
			  <li>
				<label for="id_filtros_0">
				  <input type="radio" id="id_filtros_0" value="a" name="filtros" checked="checked"/> 
				  Todos los COMPONENTES
				</label>
			  </li>

			  <li>
				<label for="id_filtros_1">
				  <input type="radio" id="id_filtros_1" value="b" name="filtros" /> 
				  Componente CAPACIDAD PROFESIONAL (CPF)
				</label>
			  </li>

			  <li>
				<label for="id_filtros_2">
				  <input type="radio" id="id_filtros_2" value="c" name="filtros" /> 
				  Componente CAPACIDAD PEDAGÓGICA (CPG)
				</label>
			  </li>

			  <li>
				<label for="id_filtros_3">
				  <input type="radio" id="id_filtros_3" value="d" name="filtros" /> 
				  Componente PRÁCTICA DE VALORES (PV)
				</label>
			  </li>

			  <li>
				<label for="id_filtros_4">
				  <input type="radio" id="id_filtros_3" value="e" name="filtros" /> 
				  Reporte de SUGERENCIAS
				</label>
			  </li>

			</ol>
		  </td>
		</tr>
	  </table>
	</fieldset>


.. _template-admin-formulario_ese2012.html:

Formularios ese2012
--------------------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/admin/app/formulario_ese2012.html``::


	<style type="text/css">
	input[type=radio] {
	    margin-top: 7px;

	}
	fieldset{
	    margin: 15px 0px 15px 0px;
	    width: 75%;
	    padding: 10px 20px 10px 20px;
	}
	#id_campos, #id_indicadores{
	    width: 75%;
	}

	</style>
	<fieldset>
	<legend> Opciones de Resultados </legend>
	<ol type="a">
	<li><label for="id_opciones_0"><input type="radio" id="id_opciones_0" value="a" name="opciones" /> 
	La valoración global de la Satisfacción Estudiantil por DOCENTE</label></li>
	<ul>
	<li> {{ form.docentes }}</li>
	</ul>
	</li>
	<li><label for="id_opciones_1"><input type="radio" id="id_opciones_1" value="b" name="opciones" /> 
	La valoracion global de la Satisfacción Estudiantil por CARRERA</label></li>
	<li><label for="id_opciones_2"><input type="radio" id="id_opciones_2" value="c" name="opciones" /> 
	La valoracion  de la Satisfacción Estudiantil en cada uno de los CAMPOS, por carrera</label> 
	<ul>
	<li> {{ form.campos }}</li>
	</ul>
	</li>

	<li><label for="id_opciones_3"><input type="radio" id="id_opciones_3" value="d" name="opciones" /> La va
	loración estudiantil en cada uno de los INDICADORES por carrera</label>
	<ul>
	<li>{{ form.indicadores }} </li>
	</ul>
	</li>
	<li><label for="id_opciones_4"><input type="radio" id="id_opciones_4" value="e" name="opciones" /> Los 1
	0 indicadores de mayor SATISFACCIÓN en la Carrera</label></li>
	<li><label for="id_opciones_5"><input type="radio" id="id_opciones_5" value="f" name="opciones" /> Los 1
	0 indicadores de mayor INSATISFACCIÓN en la Carrera</label></li>
	</ol>
	</fieldset>


.. _template-admin-menu_resultados.html:

Menú resultados
---------------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/admin/app/menu_resultados.html``::


	{% extends "admin/base_site.html" %}
	{% block extrahead %}
	<script type="text/javascript" src="/static/js/jquery-1.6.2.min.js"></script>
	<script type="text/javascript" src="/static/js/sedd.js"></script>
	<script type="text/javascript">
	    $(document).ready(function(){
		$('.boton').hide();
		menu_academico_ajax();
		// Para actualización de los dos campos Docente
		$('#id_carrera').change(function(){
		    var id_periodoe = $("#id_periodo_evaluacion option:selected").val();
		    /* Si se trata de la Encuesta de Satisfacción Estudiantil */
		    if (id_periodoe == 1){
			// Se oculta el primer campo Docente del menú general
			$("label[for=id_docente]").hide();
			$("#id_docente").hide();
		    }

		    $("#ver-opciones").click();
		});
		$('#ver-opciones').click(function(){
		    /* NOTA: JavaScript no soporta nombres de variables largos! */
		    var id_periodoe = $("#id_periodo_evaluacion option:selected").val();
		    var area = $("#id_area option:selected").val();
		    var carrera = $("#id_carrera option:selected").val();
		    if (id_periodoe !='' && area !='' && carrera !='' ){
			$.ajax({
			    url: "/resultados/periodo/" + id_periodoe + "/",
			    method: "GET",
			    data: {area:area, carrera:carrera},
			    // El servicio devuelve un formulario HTML formateado
			    dataType: 'html',
			    success: function (data){
				$("#menu-opciones").html(data);
				$('.boton').show();
			    },
			    error: function(xhr,status,error){
				alert("error: " + status + "-" + error);
			    }
			});
		    }else{
			$("#menu-opciones").html("");
			$('.boton').hide();
		    }	    
		});
	    });
	</script>
	<style type="text/css">
	body{
	    font-size: 12px;
	}
	.boton {
	    align: right;
	    width: 150px;
	    height: 30px;
	    font-size: 15px;
	}
	/*select nth:eq(1) {
	    width: 600px;
	    font-size: 15px;
	    font-weight: bold;
	    color: #666666;
	}*/
	td {
	    vertical-align: center;
	}
	</style>
	{% endblock %}


	{% block content_title %}
	  <ul class="object-tools">
	    <li>
	      <a href="javascript:void(0)" id="ver-opciones">
	        Ver Opciones de Resultados
	      </a>
	    </li>
	  </ul>
	  <h1 style="margin-bottom: 20px;"> Seleccione Datos Académicos </h1>
	{% endblock %}


	{% block breadcrumbs %}
	  <div class="breadcrumbs">
	      <a href="../../">
	        Inicio
	      </a>
	       &rsaquo;
	       <a href="../">
	         App
	      </a>
	      &rsaquo;
	      Evaluaciones
	    </div>
	{% endblock %}

	{% block content %}
	{{ block.super }}
	<form action="/resultados/mostrar/" method="POST" target="_blank">

	  {% csrf_token %}

	  <table style="margin-bottom: 20px;">
	    {{ form.as_table }} 
	  </table>
	  <div id="menu-opciones">
	  </div>
	  <div id="acciones">
	    <input class="boton" id="borrar" type="reset" value="Borrar"/>
	    <input class="boton" id="procesar" type="submit" value="Aceptar"/>
	  </div>
	</form>
	{% endblock %}



.. _template-admin-resumen_evaluaciones.html:

Resumen de Evaluaciones
-----------------------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/admin/app/resumen_evaluaciones.html``::

	{% extends "admin/base_site.html" %}


	{% load i18n %}

	{% block extrahead %}

	{{ block.super }}
	<style>
	input {
	    font-size: 14px;
	    font-weight: bold;
	    text-align: right;
	    margin-right: 20px;
	}
	img {
	    vertical-align: middle;
	}
	</style>

	<script type="text/javascript" src="/static/js/jquery-1.6.2.min.js"></script>
	<script type="text/javascript" src="/static/js/sedd.js"></script>
	<script type="text/javascript">
	    $(document).ready(function(){
		menu_academico_ajax();
		/* Calculos de los datos de Resumen */
		$('#calcular').click(function(event){
		    var pa = $('#id_periodo_academico option:selected').val();
		    var c = $('#id_carrera option:selected').val(); 
		    if ( pa != '' &&  c != '') {
			$('#cargando').show();
			$.ajax({
			    method: 'GET',
			    url: '/admin/resumen/calcular',
			    data: {'id_periodo_evaluacion': $('#id_periodo_evaluacion option:selected').val(),
				   'area': $('#id_area option:selected').val(),
				   'carrera': $('#id_carrera option:selected').val(),
				   'semestre': $('#id_semestre option:selected').val(),
				   'paralelo': $('#id_paralelo option:selected').val()
				  },
			    dataType: 'JSON',
			    success: function(response){
				$('#estudiantes').val(response['estudiantes']);
				$('#completados').val(response['completados']);
				$('#faltantes').val(response['faltantes']);
				$('#cargando').hide();
			    },
			    error: function(xhr,status,error){
				alert("error: " + status + "-" + error);
			    }
			});
			event.preventDefault();
		    } // end if != ''
		});
	    });
	</script>
	{% endblock %}

	{% block content %}
	<h2> Total de Evaluadores del Periodo {{ periodoEvaluacion }} </h2>
	<table width="100%" style="margin-bottom: 40px;"> 
	  <tr> 
		  <td> <h3> Estudiantes: {{ evaluadores.estudiantes }} </h3></td>
		  <td> <h3> Docentes: {{ evaluadores.docentes }} </h3> </td>
		  <td> <h3> Pares Academico: {{ evaluadores.pares }} </h3></td>
		  <td> <h3> Coordinadores: {{ evaluadores.directores }} </h3></td>
		</h3>
	  </tr>
	</table>
	<h2> Detalle de Evaluaciones de Estudiantes </h2>
	<table>
	{{ form.as_table }}
	</table>
	<hr/>
	<div style="padding-top: 10px;" >
	    <button id="calcular" style="margin-right: 40px;"> Calcular </button>
	  Total: <input type='text' id='estudiantes' size='10' readonly='readonly'/>
	  Han Evaluado: <input type='text' id='completados' size='10' readonly='readonly'/>
	  NO Evaluan: <input type='text' id='faltantes' size='10' readonly='readonly'/>
	  <br/>
	  <img id="cargando" src="/static/images/loading-grey.gif"/>
	</div>

	{% endblock %}


.. ########### PLANTILLAS DE USUARIOS ##########

.. _template-user:

Plantillas para usuarios y tareas
=================================


.. _template-user-portada.html:

Portada
-------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/app/portada.html``::

	<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"> 
	<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en"> 
	<head> 
	<meta http-equiv="content-type" content="text/html; charset=utf-8" /> 

	<!-- Title --> 
	<title>SEED-UNL</title>

	<!-- Link to the stylesheet -->
	<link href="/static/style.css" type="text/css" rel="stylesheet" /> 
	<!-- Scripts in js -->
	<!--script type="text/javascript" src="/static/js/jquery-1.7.2.min.js"></script-->
	<script type="text/javascript" src="/static/js/jquery-1.6.2.min.js"></script>
	<script type="text/javascript" src="/static/js/efectos.js"></script>
		
	</head>
	<body>

	<!-- Main Block -->
	<div id="wrapper">

		<!-- Header -->
		<div id="header">
			<div id="logo">
				<a href="#"><img src="/static/images/logo.jpg" /></a>
			</div>
			<div id="slogan">
				<img src="/static/images/slogan.jpg" />
			</div>
		</div>
		<!-- End header -->

		<!-- Content -->
		<div id="content">
		
			<!-- Menu -->
			<div id="menu">
				<ul>
					<li><a href="#">Inicio</a></li>
					<li><a href="/admin/">Acceso al sistema</a></li>
					<li><a href="http://www.unl.edu.ec">Acerca de...</a></li>	
				</ul>
			</div>
			<!-- End menu -->

			<!-- Left column -->
			<div id="left">
				<h1 style="color: #8EAD45;"> Bienvenidos al sistema de evaluación de docentes de la UNL</h1>
				<p> 
				  {% if periodoEvaluacionActual %}
				  <a href="{% url 'login' %}">
				    >>> <u> <b style="color: black; font-weight: 250%; font-size: 14px;"> 
					{{ periodoEvaluacionActual.titulo|upper }} </b>  </u>
				  </a>
				  {% else %}
				  >>> <b> No existen Procesos de Evaluacion actualmente </b>
				  {% endif %}

				</p>
				<p>
				  {{ periodoEvaluacionActual.descripcion }} <br/>
				  <b> 
				    (Finaliza el {{ periodoEvaluacionActual.fin|date:"l"|title }}
				    {{ periodoEvaluacionActual.fin }})
				  </b>
				</p>

			</div>
			<!-- End left column -->
			
			<!-- Right column -->
			<div id="right">
				<img src="/static/images/coffe.jpg" />
				<div id="caption">
						<p>&quot;En los tesoros de la sabiduría está la glorificación de la vida&quot;.</p>
				</div>
			</div>
			<!-- End right column -->
			<div class="clear"></div>
		</div>
		<!-- End content -->
		
		<!-- Footer -->
		<div id="footer">
			<p>2012 &copy; Design by Iceman.</p>
		</div>
		<!-- End footer -->
		
	</div>
	<!-- End Main Block -->

	</body>
	</html>



.. _template-user-index.html:

Pagina de Inicio
----------------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/app/index.html``::


	{%  extends "admin/base_site.html" %}

	{% block content_title %}

	{% endblock %}

	{% block content %}

	<p/>

	<!-- Estudiante -->
	{% if request.session.estudiante %}
	{% if request.session.carreras_estudiante|length > 0 %}
	<h1> Carreras del Estudiante </h1>
	<ul style="margin-top: 20px;">
	  {% for carrera in request.session.carreras_estudiante %}
	  <h1> 
	    <li> 
	      <a href="{% url 'estudiante_asignaturas_docentes' carrera.num_carrera %}"> 
		{{ carrera.nombre|upper }} 
	      </a> 
	    </li> 
	    <h1>
	  {% endfor %}
	</ul>
	<hr/>
	{% endif %}
	{% endif %} 

	<!-- End Estudiante -->


	<!-- Docente -->

	{% if request.session.docente %}
	<h1> Encuestas para Autoevaluación del Docente </h1>
	<!--ul style="margin-top: 20px;"-->
	  <!--h3--> 
	      {#% for cuestionario in request.session.cuestionarios_docente %#}
	      <!-- Se recorren los cuestionarios para autoevaluacion con su estado --> 
	      {% for da in request.session.docente_autoevaluaciones %}
	      <ul class="{% if da.evaluado %} messagelist {% endif %}" 
		  {% if forloop.first %} style="margin-top: 20px;" {% endif %}>
		<h3>
		<li>
		  {% if da.evaluado %}
		  {{ da.cuestionario.titulo|upper }}
		  {% else %}
		  <a href="{% url 'encuestas' id_docente=request.session.docente.id id_asignatura=0 id_tinformante=5 id_cuestionario=da.cuestionario.id %}">
		  {{ da.cuestionario.titulo|upper }}
		  </a>
		  {% endif %}
		</li>
		</h3>
	      </ul>
	      {% empty %}
	      <ul>
		<h3> No existen encuestas para el Docente el presente Periodo.</h3>
	      </ul>
	      {% endfor %}
	  <!--/h3-->
	<!--/ul-->  
	<hr/>
	{% endif %} 

	<!-- Fin Docente -->


	<!-- Par Academico -->

	{% if request.session.docente.parAcademico %}

	<h1 style="margin-top: 20px;"> Encuestas para Pares Académicos de la Carrera </h1>
	<ul style="margin-top: 20px;">
	  {% if not request.session.cuestionarios_pares_academicos %} 
	  <h3> No existen encuestas para Pares Académicos en el presente Periodo.</h3>
	  {% else %}
	  {% for carrera in request.session.carreras_pares_academicos %}
	  <h3> 
	    <li>
	      <a href="{% url 'pares_academicos_docentes' carrera.num_carrera %} ">
		SELECCIONAR DOCENTES DE LA CARRERA {{ carrera.nombre|upper }} [{{ carrera.area|upper }}]
	      </a> 
	    </li> 
	  </h3>
	  {% endfor %}
	  {% endif %}
	</ul>
	<hr/>
	{% endif %}
	<!-- Fin Par Academico -->


	<!-- Evaluciones Director de Carrera  --> 

	{% if request.session.docente.direcciones.count > 0 %}

	<h1 style="margin-top: 20px;"> Encuestas para Coordinadores Carrera </h1>
	<ul style="margin-top: 20px;">
	  {% if not request.session.cuestionarios_directivos %} 
	  <h3> No existen encuestas para los Coordinadores en el presente Periodo.</h3>
	  {% else %}
	  {% for carrera in request.session.carreras_director %}
	  <h3> 
	    <li>
	      <a href="{% url 'director_docentes' carrera.num_carrera %} ">
		SELECCIONAR DOCENTES DE LA CARRERA {{ carrera.nombre|upper }} [{{ carrera.area|upper }}]
	      </a> 
	    </li> 
	  </h3>
	  {% endfor %}
	  {% endif %}
	</ul>
	<hr/>

	<!-- Fin Evaluaciones Director de Carrera -->

	<!-- Resultados -->  

	<h1 style="margin-top: 20px;"> Resultados de Evaluaciones de la Carrera </h1>
	<ul style="margin-top: 20px;">
	  {% for carrera in request.session.carreras_director %}
	  <h3> 
	    <li>
	      <a href="/resultados/carrera/{{ carrera.num_carrera }}/">
		RESULTADOS DE EVALUACIONES CARRERA {{ carrera.nombre|upper }} [{{ carrera.area|upper }}]</a> 
	    </li> 
	  </h3>
	  {% endfor %}
	</ul>
	<hr/>

	{% endif %} 

	<!-- Fin Resultados -->


	{% endblock %}




.. _template-user-login.html:


Login
-----

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/app/login.html``::

	{% extends "admin/login.html" %}

	{% block title %} Evaluación Docente  {% endblock %}

	{% csrf_token %}
	{% if next %}
	    <input type="hidden" name="next" value="{{ next }}" />
	{% endif %}


	{% block branding %}
	   <h1 id="site-name">  Sistema de Evaluación de Docentes UNL  </h1>
	   
	{% endblock %}

	{% block content %}
	   {{ form.non_field_errors }}   
	   {{ block.super }}
	{% endblock %}






.. _template-user-carreras.html:

Carreras
--------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/app/carreras.html``::

	{%  extends "admin/base_site.html" %}
	{#  extends "base.html" #}

	<!--link rel="stylesheet" type="text/css" href="/static/admin/css/base.css" /-->

	{% block content_title %}
	 <h3>  Carreras </h3>
	{% endblock %}

	{% block content %}

	<p/>
	<ul>
	{% for carrera in request.session.carreras %}

	<h1> <li> <a href="/carrera/asignaturas/{{carrera.num_carrera}}"> {{ carrera.nombre }} </a> </li> <h1>

	{% endfor %}
	</ul>

	{% endblock %}


.. _template-user-asignaturas_docentes.html:


Asignaturas y Docentes
----------------------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/app/asignaturas_docentes.html``::

	{%  extends "admin/base_site.html" %}
	{#% extends "base.html" %#}

	{% block content_title %}
	<h1>Seleccione el Docente a Evaluar </h1>
	{% endblock %}

	{% block content %}
	<table style="margin-top: 20px;">
	  <thead> <th> ASIGNATURA </th> <th> DOCENTES <th/> </thead>
	  <tbody>
	  {% for ad in asignaturas_docentes %}
	  <tr style="border-bottom: 1px solid dotted;">
	    <td>
	      <b>
		{{ ad.asignatura.nombre }}
	      </b>
	    </td>
	    <td>
	      {% for d in ad.docentes %}
	      <ul class="{% if d.evaluado %} messagelist {% endif %}">
		<li>
		  {% if d.evaluado %}
		     {{ d.docente }} <br/>
		  {% else %}
		  <b>
		    {# Se envia ids de asignatura y de docente #}
		    {# Modificado en encuestas de actividades adicionales 2012, revisar???#}
		    <a href="{% url 'encuestas' id_docente=d.docente.id id_asignatura=ad.asignatura.id id_tinformante=1 id_cuestionario=0 %}"> 
		      {{ d.docente }} 
		    </a> 
		    <br/>
		  </b>
		  {% endif %}
		</li>
	      </ul>
	      {% endfor %}
	    </td>
	  </tr>
	  {% endfor %}
	  <tr> 
	    <td> 
	    </td> 
	    <td align="right">
	      <a href="{% url 'proyecto.app.controllers.index' %}"> <u><<< Carreras >>> </u></a>  
	    </td> 
	  </tr>
	  </tbody>
	</table>



	{% endblock %}



.. _template-user-director_docentes.html:

Directores y Docentes
---------------------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/app/director_docentes.html``::

	{% extends "admin/base_site.html" %}
	{#% extends "base.html" %#}

	{% block content_title %}
	<h1>Seleccione el Docente a Evaluar </h1>
	{% endblock %}

	{% block content %}
	<table style="margin-top: 20px; width: 100%;">
	  <tbody>
	    <tr style="border-bottom: 1px dotted #B4CBE5;">
	      <td>
		{% for de in docentes_evaluaciones %}
		<b>
		  <ul class="{% if de.evaluado %} messagelist {% endif %}">
		    <li>
		      {% if de.evaluado %}
		      {{ de.docente }}
		      {% else %}
		      <a href="{% url 'encuestas' id_docente=de.docente.id id_asignatura=0 id_tinformante=6 id_cuestionario=0%}">
			{{ de.docente }}
		      </a>
		      {% endif %}
		    </li>
		  </ul>
		</b>
		<!-- Paginador de docentes -->
		{% if forloop.counter|divisibleby:"10" %}
	        </td>
	        <td>
		{% endif %}
		{% endfor %}
	    </td>
	  </tr>

	  <tr> 
	    <td> 
	    </td> 
	    <td align="right">
	      <a href="{% url 'proyecto.app.controllers.index' %}"> <u><<< Carreras >>> </u></a>  
	    </td> 
	  </tr>
	  </tbody>
	</table>

	{% endblock %}


.. _template-user-pares_academicos_docentes.html:

Pares Académicos y Docentes
---------------------------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/app/pares_academicos_docentes.html``::

	{% extends "admin/base_site.html" %}

	{% block content_title %}
	<h1>Seleccione el Docente a Evaluar </h1>
	{% endblock %}

	{% block content %}
	<table style="margin-top: 20px; width: 100%;">
	  <tbody>
	    <tr style="border-bottom: 1px dotted #B4CBE5;">
	      <td>
		{% for de in docentes_evaluaciones %}
		<b>
		  <ul class="{% if de.evaluado %} messagelist {% endif %}">
		    <li>
		      {% if de.evaluado %}
		      {{ de.docente }}
		      {% else %}
		      <a href="{% url 'encuestas' id_docente=de.docente.id id_asignatura=0 id_tinformante=10 id_cuestionario=0%}">
			{{ de.docente }}
		      </a>
		      {% endif %}
		    </li>
		  </ul>
		</b>
		<!-- Paginador de docentes -->
		{% if forloop.counter|divisibleby:"10" %}
	        </td>
	        <td>
		{% endif %}
		{% endfor %}
	    </td>
	  </tr>

	  <tr> 
	    <td> 
	    </td> 
	    <td align="right">
	      <a href="{% url 'proyecto.app.controllers.index' %}"> <u><<< Carreras >>> </u></a>  
	    </td> 
	  </tr>
	  </tbody>
	</table>

	{% endblock %}



.. _template-user-en_construccion.html:

en contruccion
--------------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/app/en_construccion.html``::

	<html>
	  <head>  
	    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
	    <title> En construcción </title>
	    <style type="text/css">

	    body {
		font-family: Helvetica, Arial, Sans;
	    }

	    </style>
	  </head>
	  <body>
	    <center>
	      <center><h2> En Construcción ...  </h2> </center>
	      <div id="pie" style="font-size: 9px; color: #666666">
			<hr style="margin-bottom: 6px;"/>
			Unidad de Telecomunicaciones e Infomacion - Departamento de Desarrollo de Software <br>
			Correo Sistema de Evaluacion: 
			<a href="mailto:evaluacion.docente@unl.edu.ec"> <u>evaluacion.docente@unl.edu.ec</u> </a> <br/>
			Loja - Ecuador
	      </div>

	    </center>
	  </body>
	</html>




.. _template-user-encuestas.html:

Encuestas
---------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/app/encuestas.html``::

	{% extends "admin/base_site.html" %}

	{% block content_title %}
	<h1> Encuestas Disponibles </h1>
	{% endblock %}

	{% block content %}

	{% if periodo_no_iniciado %}

	<ul class="messagelist" style="margin-top: 20px;">
	  <b><li class="warning"> El Periodo de Evaluacion aun no inicia </li></b>
	</ul>

	{% else %}
	{% if periodo_finalizado %}

	<ul class="messagelist" style="margin-top: 20px;">
	  <b><li class="error"> Lo sentimos, el Periodo de Evaluacion ha Finalizado </li></b>
	</ul>

	{% else %}

	<ul>
	  {% for cuestionario in cuestionarios %}
	  <h3>
	    <li>
	      <a href="/encuesta/responder/{{ cuestionario.id }} " > {{ cuestionario.titulo }} </a> 
	    </li>
	  </h3>
	  {% empty %}
	  <h3> No existen Encuestas Disponibles </h3>
	  {% endfor %}
	</ul>

	{% endif %}
	{% endif %}

	{% endblock %}


.. _template-user-encuesta_responder.html:

Encuesta para Responder
-----------------------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/app/encuesta_responder.html``::

	{%  extends "admin/base_site.html" %}

	{% load i18n %}


	{% block extrahead %}

	   {{ block.super }}

	    <style type="text/css">
	    #encabezado {
		margin-top: 20px;
		margin-bottom: 20px;
		align: center;
	    }

	    .seccion{
		/*background-color: #EDF3FE;*/
	        background-color: #B4CBE5;
	    }

	    .td-item-seleccion{
		border: 2px white solid;
		background-color: #B4CBE5;
		text-align: center;
	        display: table-cell;
		/* Para cortar las palabras al ancho de la celda */
	        word-break: break-all;
	        
	    }

	    .td-tabla-contenedor {
	        border-bottom: 1px dotted  #B4CBE5;
	        border-left: 1px dashed  #B4CBE5;
	    }

	    #tabla-items{
		width: 100%;
	    }
	    </style>

	    <script type="text/javascript" src="/static/js/jquery-1.6.2.min.js"> </script>

	    <script type="text/javascript">
	    
	function validarSeleccionUnica(){
	    var preguntas = new Array();
	    // Se validan como obligatorias todas las preguntas de Selección Unica
	    $.each($('#formulario input[name^="pregunta"]:radio'), function(){
		var nombre = $(this).attr("name");
		preguntas[nombre] = nombre;
	    });
	    for (var pregunta in preguntas){
		if (! $('#formulario input[name=' + pregunta +']:radio').is(':checked') ){
		    alert('ERROR: Faltan preguntas por contestar');
		    return false;
		}
		
	    }
	    return true;
	};

	function validarEnsayos(){
	    var preguntas = new Array();
	    // Se validan como obligatorias todas las preguntas de Ensayo
	    $.each($('#formulario textarea[name^="pregunta"]'), function(){
		var nombre = $(this).attr("name");
		preguntas[nombre] = nombre;
	    });
	    for (var pregunta in preguntas){
		if ( $('#formulario textarea[name=' + pregunta +']').val().length < 4 ){
		    alert('ERROR: Faltan preguntas por completar');
		    return false;
		}
	    }
	    return true;
	};

	function validarObservaciones(){
	    /* Validación en Evaluacion de Actividades Adicionales a la Docencia 2011-2012 */
	    var flag = true;
	    $.each($('#formulario input[name^="pregunta"]:radio:checked'), function(){
		var p = $(this).attr("name");
		var r = $(this).attr("value");
		var np = p.split('-')[1]; //Numero de la pregunta
		var o = 'observaciones-pregunta-' + np;
		var observaciones = $('textarea[name='+o+']').val();
		if (r != '1' && observaciones == ''){
		    alert('Faltan ingresar fuentes de verificación')
		    flag=false;
		    return flag;
		}
	    });
	    return flag;
	}

	function grabarEncuesta(){
	    /* Se trata de la Evaluacion de Actividades Adicionales a la Docencia 2011-2012 */
	    {% ifequal cuestionario.periodoEvaluacion.id 2 %}
	    if (validarObservaciones()){
		$('#formulario').submit();
	    } else {
		return false;
	    }
	    {% endifequal %}
	    /* Para el resto de casos de Periodos de Evaluacion */
	    $('#formulario').submit();


	    /*$.ajax({
		url: "/encuesta/grabar/",
		method: "POST",
		//data: "periodo_academico_id={{ object_id }}",
		success: function (data){
		    alert(data);
		},
		error: function(xhr,status,error){
		    alert("error: " + status + "-" + error);
		}
	    });*/
	}

	$(document).ready(function(){
	    $('#grabar').click(function(){
		{% if cuestionario.preguntas_obligatorias %}
		if (validarSeleccionUnica() == true && validarEnsayos() == true){
		    grabarEncuesta();
		}
		{% else %}
		    grabarEncuesta();	
		{% endif %}
	    });
	});
	    </script>
	    
	{% endblock %}


	{% block content_title %}

	<h3> {{ cuestionario.titulo }} </h3>

	{% endblock %}


	{% block content %}


	<form action="/encuesta/grabar/" method="post" id="formulario" name="formulario" >

	  {% csrf_token %}

	  <table id="tabla-contenedor">

		<td colspan="2">
		  <div id="encabezado-cuestionario" width="25%">
		    {{ cuestionario.encabezado|safe }} 
		  </div>

		  <!-- Encuesta dirigida a los ESTUDIANTES -->
		  {#% if cuestionario.informante.tipo.startswith('Estudiante') %#}
		  {% if asignaturaDocente %}
		  <div id="datos_academicos" style="line-height: 18px;">
		    Area: <b> {{ asignaturaDocente.asignatura.area }} </b> <br/>
		    Carrera: <b> {{ asignaturaDocente.asignatura.carrera }} </b> <br/>
		    Modulo ( {% ifequal asignaturaDocente.asignatura.tipo "modulo" %} <b>X</b> {% endifequal %}  );
		    Unidad ( {% ifequal asignaturaDocente.asignatura.tipo "unidad" %} <b>X</b> {% endifequal %}  )
		    Curso ( {% ifequal asignaturaDocente.asignatura.tipo "curso" %} <b>X</b> {% endifequal %}  )
		    Taller ( {% ifequal asignaturaDocente.asignatura.tipo "taller" %} <b>X</b> {% endifequal %}  )
		    Otro ( {% ifequal asignaturaDocente.asignatura.tipo "otro" %} <b>X</b> {% endifequal %}  )
		    <br/>
		    Nombre de el/la Docente Evaluado(a): <b> {{ asignaturaDocente.docente }} </b> <br/>
		    Fecha de Evaluacion: <b> {{ fecha }} </b> <br/>
		  </div>
		  {% endif %}

		  <!-- Encuesta dirigida a los DIRECTORES DE CARRERA -->
		  {% if 'Directivo' in cuestionario.informante.tipo %}
		  <div id="datos_academicos" style="line-height: 18px;">
		    AREA: <b> {{ request.session.area }} </b> <br/>
		    CARRERA: <b> {{ request.session.carrera }} </b> <br/>
		    MÓDULO: (<b>X</b>) <br/>
		    NOMBRE DEL PROFESOR (A) EVALUADO (A): <b> {{ request.session.docente_evaluar }} </b>
		  </div>
		  {% endif %}

		  <!-- Encuesta dirigida a los PARES ACADEMICOS -->
		  {% if 'ParAcademico' in cuestionario.informante.tipo %}
		  <div id="datos_academicos" style="line-height: 18px;">
		    AREA: <b> {{ request.session.area }} </b> <br/>
		    CARRERA: <b> {{ request.session.carrera }} </b> <br/>
		    MÓDULO: (<b>X</b>) <br/>
		    NOMBRE DEL PROFESOR (A) EVALUADO (A): <b> {{ request.session.docente_evaluar }} </b>
		  </div>
		  {% endif %}

		  <!-- Encuesta de autoevaluacion para los DOCENTES -->
		  {% if cuestionario.informante.tipo == 'Docente' %}
		  <div id="datos_academicos" style="line-height: 18px;">
		    AREA: <b> {{ request.session.areas_docente }} </b> <br/>
		    CARRERA: <b> {{ request.session.carreras_docente }} </b> <br/>
		    MÓDULO: (<b>X</b>) <br/>
		    NOMBRE DEL PROFESOR (A) EVALUADO (A): <b> {{ request.session.docente }} </b>
		  </div>
		  {% endif %}

		</td>
		<!-- end datos academicos -->
		
		<tbody>

		  {% for seccion in cuestionario.secciones.all %}  <!-- secciones de cuestionario -->
		  <tr>
			<td class="seccion" colspan="3" > 
			  <b> {{ seccion.orden }}. {{ seccion.titulo|upper }} </b> 
			  {{ seccion.descripcion }}  
			</td>
		  </tr>

		  <!-- =============== Tratamiento de preguntas de SUBSECCIONES =============== -->

		  {% for subseccion in seccion.subsecciones.all %}
		  <tr>
			<!-- columna subseccion -->
			<td class="td-tabla-contenedor" width="10%" rowspan="{{ subseccion.preguntas.count }}"> 
			   {% if subseccion.codigo %} {{ subseccion.codigo }} {% else %} {{ subseccion.orden }} {% endif %} 
			</td>
			<!--/columna subseccion -->

			<!-- preguntas de subsecciones -->
			{% for pregunta in subseccion.preguntas.all %}
			<!-- columna  pregunta -->
			<td class="td-tabla-contenedor" width="60%" colspan="{% ifequal pregunta.tipo.tipo 'Ensayo' %} 3 {% endifequal %} ">
			  <b> 
				{% if pregunta.codigo %} {{ pregunta.codigo }}. {% else %} {{ pregunta.orden }}. {% endif %} 
			  </b>
			  {{ pregunta.texto|safe }}
			  <br/>
			  {{ pregunta.descripcion|safe }} <!-- Puede contener HTML --> 
			  {% ifequal pregunta.tipo.tipo 'Ensayo' %}
			  <br/>
			  <textarea colspan="2" name="pregunta-{{ pregunta.id }}" cols="100" rows="10" style="width: 100%;" ></textarea>
			  {% endifequal %}
			</td>
			<!--/ preguntas de subsecciones -->
		
			<!-- items de pregunta si es de seleccion -->
			{% ifequal pregunta.tipo.tipo 'SeleccionUnica' %}
			<td class="td-tabla-contenedor" width="30%">
			  <table id="tabla-items">
				<tbody>
				  <tr>
					{% for item in pregunta.items.all %}
					<!--Dividir para el numero exacto de items de la pregunta Ej. width: 25%-->
					<td class="td-item-seleccion" {% if not item.descripcion %} style="vertical-align: middle;" {% endif %} 
					{% if not pregunta.observaciones %} style="width:25%;" {%else%} style="width:20%;" {% endif %} >
					  <input type="radio" name="pregunta-{{ pregunta.id }}"  value="{{ item.texto }}"> </input> 
					  <b> {{ item.texto }} </b>
					  <br/>
					  {% if item.descripcion %} {{ item.descripcion }} {% endif %}
					</td>
					{% endfor%}

					{% if pregunta.observaciones %}
					<td id="observaciones">
					  {{pregunta.observaciones|safe}}:
					  <textarea name="observaciones-pregunta-{{ pregunta.id }}" rows="6" style="width: 100%;"></textarea>
					</td>
					{% endif %}
				  </tr>
				</tbody>
			  </table>
			</td>
			{% endifequal %}
			<!--/ items pregunta si es de seleccion -->
		  </tr> 
		  {% if not forloop.last %}
		  <tr> <!-- por el rowspan de la primera columna -->
		  {% endif %}
		  {% endfor %} <!--/ preguntas de subseccion -->
		  {% endfor %} <!--/ subsecciones -->

		  <!-- ================= Tratamiento de Peguntas de SECCIONES ================ -->

		  {% for pregunta in seccion.preguntas.all %} <!-- preguntas de secciones  -->
		  <tr>
			<td class="td-tabla-contenedor" width="60%" colspan="{% ifequal pregunta.tipo.tipo 'Ensayo' %} 3 {% endifequal %} ">
			  {% if cuestionario.secciones.all|length > 1 %} 
			  <b>
				{% if pregunta.codigo %} {{ pregunta.codigo }}. {% else %} 	{{ seccion.orden }}.{{ pregunta.orden }}. {% endif %}
			  </b> 
			  {{ pregunta.texto|safe }}
			  {% else %}
			  <b> 
				{% if pregunta.codigo %} {{ pregunta.codigo }}. {% else %} {{ pregunta.orden }}. {% endif %} 
			  </b> 
			  {{ pregunta.texto|safe }}  
			  {% endif %}
			  <br/>
			  {% if pregunta.descripcion %}
			  {{ pregunta.descripcion|safe }} <!-- Puede contener HTML --> 
			  {% endif %}
			  {% ifequal pregunta.tipo.tipo 'Ensayo' %}
			  <br/>
			  <textarea name="pregunta-{{ pregunta.id }}" cols="100" rows="10" style="width: 100%;" ></textarea>
			  {% endifequal %}
			</td>

			<!-- items pregunta de seccion si es de seleccion -->
			{% ifequal pregunta.tipo.tipo 'SeleccionUnica' %}
			<td class="td-tabla-contenedor" width="40%">
			  <table id="tabla-items">
				<tbody>
				  <tr>
					{% for item in pregunta.items.all %}
					<!--TODO: Dividir para el numero exacto de items de la pregunta Ej. width: 25%-->
					<td class="td-item-seleccion" {% if not item.descripcion %} style="vertical-align: middle;" {% endif %} 
					{% if not pregunta.observaciones %} style="width:25%;" {%else%} style="width:20%;" {% endif %} >
					  <input type="radio" name="pregunta-{{ pregunta.id }}"  value="{{ item.texto }}"> </input> 
					  <b> {{ item.texto }} </b>
					  <br/>
					  {% if item.descripcion %} {{ item.descripcion }} {% endif %}
					</td>
					{% endfor%}

					{% if pregunta.observaciones %}
					<td id="observaciones">
					  {{pregunta.observaciones|safe}}:
					  <textarea name="observaciones-pregunta-{{ pregunta.id }}" rows="6" style="width: 100%;"></textarea>
					</td>
					{% endif %}
				  </tr>
				</tbody>
			  </table>
			</td>
			{% endifequal %}
			<!--/ items pregunta de seccion si es de seleccion -->
		  </tr> 
		  {% endfor %} <!--/ preguntas de seccion -->

		  {% endfor %} <!--/ secciones de cuestionario -->

		</tbody>

		<tfoot>
		  <div id="comandos">
			<tr>
			  <td colspan="2"> 
				<input type="button" id="grabar" name="grabar" value="GRABAR"/>  
				<input type="reset" value="BORRAR"/>
				<input type="button" id="cancelar" name="cancelar" value="CANCELAR"/>
			  </td>
			</tr>
		  </div>
		</tfoot>
	  
	  </table>

	</form>

	{% endblock %}


.. _template-user-encuesta_finalizada.html:

Encuesta Finalizada
-------------------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/app/encuesta_finalizada.html``::

	{% extends "admin/base_site.html"%}

	{% block title %}  Encuesta Grabada  {% endblock %}

	{% block content %}

	<h1> Información grabada satisfactoriamente. </h1>

	<p> Gracias por participar de la Encuesta. </p>

	{% if request.session.estudianteAsignaturaDocente %}
	<p>
	  <a href="{% url 'proyecto.app.controllers.estudiante_asignaturas_docentes' num_carrera %}"> 
	    <<< Continuar >>> 
	  </a>
	</p>
	{% else %}
	{% if request.session.director_carrera and num_carrera %}
	<p>
	  <a href="{% url 'proyecto.app.controllers.director_docentes' num_carrera %}"> 
	  <!--a href="{#% url encuestas id_docente=request.session.docente_evaluar.id id_asignatura=0 id_tinformante=6 %#}"--> 
	    <<< Continuar >>> 
	  </a>
	</p>
	{% else %}
	{% if request.session.docente %}
	<p>
	  <a href="{% url 'proyecto.app.controllers.index' %}"> 
	    <<< Continuar >>> 
	  </a>
	</p>
	{% endif %}
	{% endif %}
	{% endif %}

	{% endblock %}


.. _template-user-imprimir_resultados_eaad2012.html:

Resultados eaad 2012
--------------------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/app/imprimir_resultados_eaad2012.html``::

	<html>
	  <head>  
	    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
	    <title> Resultados Evaluación de Actividades Adicionales a la Docencia 2011 - 2012 </title>
	    <style type="text/css">
	    body {
		font-family: Helvetica, Arial, Sans;
	    }
	    table {
		margin-top: 10px;
		margin-bottom: 10px;
		width: 75%;
		borderspacing: 0;
	    }
	    th {
		text-align: center;
		border: 1px solid gray;
	        font-size: 13px;
	    }
	    td {
		text-align: left;
		border: 1px solid gray;
	        font-size: 13px;
	    }
	    textarea {
		font-size: 10px;
		overflow: hidden;
	    }

	    </style>
	  </head>
	  <body>
	    <center>
	      <img width="75px" height="75px" src="/static/images/unl_logo.png"/>
	      <h4> 
	      {{ "Unversidad Nacional de Loja"|upper }}  <br/>
	      Comisión de Evaluación Interna <br/>
	      EVALUACIÓN DEL DESEMPEÑO ACADÉMICO <p/>
	      {{ "Resultados de la Evaluación de Actividades Adicionales a la Docencia 2011-2012"|upper }}

	      <!-- Tabla de encabezado -->

	      <table id="tabla-docente" width="100%" border="0">
		<tr>
		  <td width="10%"><b>AREA</b></td> <td width="90%">{{ area }}</td>
		</tr>
		<tr>
		  <td width="10%"><b>CARRERA</b></td> <td width="90%">{{ carrera }}</td>
		</tr>
		<tr>
		  <td width="10%"><b>DOCENTE</b></td> <td width="90%">{{ docente }}</td>
		</tr>
	      </table>

	      <!-- Tabla Central -->

	      <table id="tabla-resultados" width="100%" cellspacing="0" >
		<th rowspan="3" colspan="2" style="width: 80%;"> ACIVIDADES </th>
		<th colspan="4"> INFORMANTES </th>
		<th rowspan="3"> Fuente de Verificación </th>
		<tr>
		  <th colspan="2"> COMISION </th>
		  <th colspan="2"> DOCENTE </th>
		</tr>
		<tr>
		  <th width="4%"> Esc. </th>
		  <th width="4%"> % </th>
		  <th width="4%"> Esc. </th>
		  <th width="4%"> % </th>
		</tr>
		{% for seccion in contestaciones.secciones %}
		<tr>
		  <td colspan="7"> <b> {{ seccion.titulo|upper }} </b> </td> 
		</tr>
		{% for r in seccion.resultados %}
		<tr>
		  <td width="4%"> {{ r.codigo }} </td>
		  <td width="70%" style=""> {{ r.texto }}  </td>
		  <td style="text-align: center;"> {{ r.valor_comision }} </td>
		  <td style="text-align: center;"> {%if r.porcentaje_comision%} {{ r.porcentaje_comision }}% {%endif%}</td>
		  <td style="text-align: center;"> {{ r.valor_docente }} </td>
		  <td style="text-align: center;"> {%if r.porcentaje_docente%} {{ r.porcentaje_docente }}% {%endif%}</td>
		  <td> 
		    <textarea readonly="readonly" rows="4">COMISION: {{ r.observaciones_comision }} DOCENTE: {{ r.observaciones_docente }}  </textarea>
		  </td>
		</tr>
		{% endfor %}
		{% endfor %}
	      </table>

	      <!-- Tabla de Resultados -->

	      <table width="100%">
		<tr>
		  <td width="70%"> <h3> Número de Actividades:  </h3></td>
		  <td colspan="2" style="text-align: center;"> <h3> {{ contestaciones.num_actividades }} </h3> </td>
		</tr>
		<tr> 
		  <td rowspan="2"> <h3> Valoración Global Actividades Adicionales: </h3></td>
		  <td style="text-align: center;"> <h3> Comision: {{ porcentaje1|floatformat:"2" }}% </h3> </td>
		  <td style="text-align: center;"> <h3> Docente: {{ porcentaje2|floatformat:"2" }}% </h3> </td>
		</tr>
		<tr>
		  <td colspan="2" style="text-align: center; vertical-align: center;"> <h1> {{ total|floatformat:"2" }}% </h1> </td>	  
		</tr>
	      </table>

	      <div id="pie" style="font-size: 9px; color: #666666">
		<hr style="margin-bottom: 6px;"/>
		Unidad de Telecomunicaciones e Infomacion - Departamento de Desarrollo de Software <br>
		Correo Sistema de Evaluacion: 
		<a href="mailto:evaluacion.docente@unl.edu.ec"> <u>evaluacion.docente@unl.edu.ec</u> </a> <br/>
		Loja - Ecuador
	      </div>

	    </center>
	  </body>
	</html>



.. _template-user-imprimir_resultados_ese2012.html:

Resultados Satisfacción Estudiantil 2012
----------------------------------------

Plantilla para imprimir los resultados en la encuesta de Satisfacción Estudiantil 2012.
Ubicación: ``{{TEMPLATE_DIRS}}/app/imprimir_resultados_ese2012.html``::

	<html>
	  <head>  
	    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
	    <title> Resultados Encuesta de Satisfacción Estudiantil 2012 </title>

Estilos::

	    <style type="text/css">
	    body {
		font-family: Helvetica, Arial, Sans;
	    }
	    table {
		margin-top: 10px;
		margin-bottom: 10px;
		width: 75%;
		borderspacing: 0;
	    }
	    td, th {
		text-align: center;
		border: 1px solid gray;
	        font-size: 13px;
	        width: 20%;
	    }
	    </style>
	  </head>

Inicio del body::

	  <body>
	    <center>
	      <img width="75px" height="75px" src="/static/images/unl_logo.png"/>
	      <h4> {{ "Unversidad Nacional de Loja"|upper }} </h4>
	      <h3 style="margin-bottom: 3px;"> Encuesta de Satisfacción Estudiantil 2012 </h3>
	      {{ titulo|safe }}
	      <table  width="100%" cellspacing="0">
		<th rowspan="2"> Indicadores </th>
		<th colspan="4"> Niveles de Satisfacción  </th>
		<tr>
		  <th> Muy Satisfecho MS (4) </th>
		  <th> Satisfecho S (3) </th>
		  <th> Poco Satisfecho PS (2) </th>
		  <th> Insatisfecho INS (1) </th>
		</tr>
		{% for c in conteos %}
		<tr>
		  <td title="{{ c.pregunta.texto }}" width="20%"> 
		    <b>{{ c.pregunta.seccion.orden }}.{{c.pregunta.orden}} </b>
		  </td>
		  <td> {{ c.MS }} </td>
		  <td> {{ c.S }} </td>
		  <td> {{ c.PS }} </td>
		  <td> {{ c.INS }} </td>
		</tr>
		{% endfor %}
	      </table>

	      {% if totales and porcentajes %}
	      <h3> Resultados </h3>
	      <table cellspacing="0">
		<tr>
		  <td> Suman ( {{ totales.total }} ) </td>
		  <td> {{ totales.MS }} </td>
		  <td> {{ totales.S }} </td>
		  <td> {{ totales.PS }} </td>
		  <td> {{ totales.INS }} </td>
		</tr>
		<tr>
		  <td> % de Estudiantes que declaran su Satisfacción </td>
		  <td> {{ porcentajes.MS|floatformat:"-2" }}% </td>
		  <td> {{ porcentajes.S|floatformat:"-2" }}% </td>
		  <td> {{ porcentajes.PS|floatformat:"-2" }}% </td>
		  <td> {{ porcentajes.INS|floatformat:"-2" }}% </td>
		</tr>
		<tr>
		  <td> % de Estudiantes que se declaran Satisfechos </td>
		  <td colspan="2"> {{ porcentajes.MSS|floatformat:"-2" }} % </td>
		  <!--td colspan="2"> {{ porcentajes.MS|add:porcentajes.S }}% </td-->
		  <td colspan="2"> </td>
		</tr>
	      </table>
	      {%endif%}

Sección final::

	      <div id="pie" style="font-size: 9px; ; color: #666666">
		<hr style="margin-bottom: 6px;"/>
		Unidad de Telecomunicaciones e Infomacion - Departamento de Desarrollo de Software <br>
		Correo Sistema de Evaluacion: 
		<a href="mailto:evaluacion.docente@unl.edu.ec"> <u>evaluacion.docente@unl.edu.ec</u> </a> <br/>
		Loja - Ecuador
	      </div>

	    </center>
	  </body>
	</html>



.. _template-user-imprimir_otros_ese2012.html:

Otros resultados Satisfacción Estudiantil 2012
----------------------------------------------

Plantilla para imprimir otros resultados en la encuesta de Satisfacción Estudiantil 2012.
Ubicación: ``{{TEMPLATE_DIRS}}/app/imprimir_otros_ese2012.html``::

	<html>
	  <head>  

Encabezados y estilos::

	    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
	    <title> Resultados Encuesta de Satisfacción Estudiantil 2012 </title>
	    <style type="text/css">
	    body {
		font-family: Helvetica, Arial, Sans;
	    }
	    table {
		margin-top: 10px;
		margin-bottom: 10px;
		width: 75%;
		borderspacing: 0;
	    }
	    td, th {
		border: 1px solid gray;
	        font-size: 13px;
	    }
	    </style>
	  </head>

Inicio del body::

	  <body>
	    <center>
	      <img width="75px" height="75px" src="/static/images/unl_logo.png"/>
	      <h4> {{ "Unversidad Nacional de Loja"|upper }} </h4>
	      <h3 style="margin-bottom: 3px;"> Encuesta de Satisfacción Estudiantil 2012 </h3>
	      {{ titulo|safe }}
	      <table id="tabla-resultados" width="100%" cellspacing="0">
		<th width="25%"> Otros Aspectos </th>
		<th width="70%"> Respuesta  </th>
		<th width="5%"> Frecuencia </th>
		{% for r in resultados %}
		<tr>
		  <td width="25%">  {{ r.pregunta.texto }} </td>
		  <td width="70%"> {{ r.respuesta }} </td>
		  <td width="5%" align="center"> {{ r.frecuencia }} </td>
		</tr>
		{% endfor %}
	      </table>

Pie de página::

	      <div id="pie" style="font-size: 9px; ; color: #666666">
		<hr style="margin-bottom: 6px;"/>
		Unidad de Telecomunicaciones e Infomacion - Departamento de Desarrollo de Software <br>
		Correo Sistema de Evaluacion: 
		<a href="mailto:evaluacion.docente@unl.edu.ec"> <u>evaluacion.docente@unl.edu.ec</u> </a> <br/>
		Loja - Ecuador
	      </div>
	    </center>
	  </body>
	</html>



.. _template-user-imprimir_resultados_edd2013.html:

Resultados Desempeño Docente 2013
----------------------------------

Ubicación: dentro de ``{{TEMPLATE_DIRS}}/app/imprimir_resultados_edd2013.html``::

	<html>
	  <head>  
	    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
	    <title> Resultados Evaluación del Desempeño Académico 2012 - 2013 </title>

Sección de estilos::

	    <style type="text/css">
	    body {
		font-family: Helvetica, Arial, Sans;
	    }
	    table {
		margin-top: 10px;
		margin-bottom: 10px;
		width: 85%;
		borderspacing: 0;
	    }
	    th {
		text-align: center;
		border: 1px solid gray;
	        font-size: 13px;
	    }
	    td {
		text-align: center;
		border: 1px solid gray;
	        font-size: 13px;
	    }
	    textarea {
		font-size: 10px;
		overflow: hidden;
	    }
		.promedio_componente td {
		text-decoration: overline;
		/*font-size: 0%; */
		}
	    </style>
	  </head>

Inicio del body::

	  <body>
	    <center>
	      <img width="75px" height="75px" src="/static/images/unl_logo.png"/>
	      <h4> 
	      {{ "Unversidad Nacional de Loja"|upper }}  <br/>
	      Comisión de Evaluación Interna <br/>
	      EVALUACIÓN DEL DESEMPEÑO ACADÉMICO <p/>
	      {{ "Resultados de la Evaluación del Desempeño Docente 2012"|upper }}

Tabla de encabezado::

	      <table id="tabla-docente"  border="0">
			{% if area %}
			<tr>
			  <td width="10%"><b>AREA</b></td> <td style="text-align: left;" width="90%">{{ area }}</td>
			</tr>
			{% endif %}
			{% if carrera %}
			<tr>
			  <td width="10%"><b>CARRERA</b></td> <td style="text-align: left;" width="90%">{{ carrera }}</td>
			</tr>
			{% endif %}
			{% if docente %}
			<tr>
			  <td width="10%"><b>DOCENTE</b></td> <td style="text-align: left;" width="90%">{{ docente }}</td>
			</tr>
			{% endif %}
		  </table>

Si se filtra por Componente::

		  {% if seccion_componente  %}
		  <table>
			<tr>
			  <td style="text-align: left;" width="40%">
				<h2> {{ seccion_componente.titulo }}:</h2> 
			  </td>
			  <td style="text-align: left;">
				<h3> {{ seccion_componente.descripcion }} </h3>
			  </td>
			</tr>
	      </table>
		  {% endif %}	  

Tabla Central::

	      <table id="tabla-resultados" width="100%" cellspacing="0" >
			<tr>
			  <th rowspan="3" colspan="2">  DESCRIPCIÓN DE LOS INDICADORES DE CALIDAD </th>
			  <th colspan="4">  INFORMANTES </th>
			  <th colspan="3">  EVALUACIÓN </th>
			</tr>
			<tr>
			  <th rowspan="2"> ESTUDIANTES </th>
			  <th rowspan="2"> DOCENTES </th>
			  <th colspan="2"> COMISIÓN ACADÉMICA </th>
			  <th rowspan="2"> PRIMARIA </th>
			  <th rowspan="2">  PONDERAD. </th>
			  <th rowspan="2"> CUALITAT. </th>
			</tr>
			<tr> 
			  <th> PAR ACAD. </th>
			  <th> DIRECTIVO </th>
			</tr>

Indicador, resultado::

			{% for i, r in resultados_indicadores.items %}

Titulos de los componentes::

			{# Titulos de los diferentes componentes o secciones #}
			{% ifchanged r.objeto_seccion.superseccion.codigo %}
			{# Para acortar el nombre del objeto #}
			{% with superseccion=r.objeto_seccion.superseccion %}
			<tr> 
			  <td colspan="9" style="text-align: left;">
				<b>
				  COMPONENTE {{ superseccion.orden }}: {{ superseccion.titulo|upper }} ({{ superseccion.descripcion }})
				</b>
			  </td> 
			</tr>
			{% endwith %}
			{% endifchanged %}

Cada uno de los indicadores::

			<tr>
			  <td style="text-align: left; font-size: 9px;"> 
				{{ r.objeto_seccion.descripcion }} 
			  </td>
			  <td>
				{{ i }} 
			  </td>
			  <td width="10%"> 
			  {% if r.informantes.estudiante >= 0 %} {{ r.informantes.estudiante }} % {% else %} - {% endif %} 
			  </td>
			  <td width="10%"> 
			  {% if r.informantes.docente >= 0 %} {{ r.informantes.docente  }} % {% else %} - {% endif %}
			  </td>
			  <td width="10%"> 
				{% if r.informantes.paracademico >= 0 %} {{ r.informantes.paracademico }} % {% else %} - {% endif %} 
			  </td>
			  <td width="10%"> 
			  {% if r.informantes.directivo >= 0%} {{ r.informantes.directivo }} % {% else %} - {% endif %} 
			  </td>
			  <td width="10%"> 
			  {% if r.primaria >= 0 %} {{ r.primaria|floatformat:"0" }} % {% else %} - {% endif %}
			  </td>
			  <td width="10%"> {{ r.ponderada|floatformat:"2" }} <b>/</b> {{ r.ponderacion_seccion }} </td>
			  <td width="10%"> {{ r.cualitativa }} </td>
			</tr>

Totales por Componente::

			{% if promedios_componentes %} <!-- if promedios_componentes -->
			{# Total CPF #}
			{% ifequal forloop.counter promedios_componentes.CPF.fila %}
			<tr class="promedio_componente"> 
			  <td colspan="2" style="text-align: left;"> 
				TOTAL {{ r.objeto_seccion.superseccion.titulo }}
			  </td>  
			  <td> 
				{{ promedios_componentes.CPF.estudiante }} %
			  </td>
			  <td> 
				{{ promedios_componentes.CPF.docente }} %
			  </td>
			  <td> 
				{{ promedios_componentes.CPF.paracademico }} %
			  </td>
			  <td> 
				{{ promedios_componentes.CPF.directivo }} %
			  </td>
			  <td> 
				{{ promedios_componentes.CPF.primaria|floatformat:"0" }} %
			  </td>
			  <td> 
				{{ promedios_componentes.CPF.ponderada|floatformat:"2" }} / {{ r.objeto_seccion.superseccion.ponderacion }}
			  </td>
			  <td> 
				{{ promedios_componentes.CPF.cualitativa }} 
			  </td>
			</tr>
	 		{% endifequal %}

			{# Total CPG #}
			{% ifequal forloop.counter promedios_componentes.CPG.fila %}
			<tr class="promedio_componente"> 
			  <td colspan="2" style="text-align: left;"> 
				TOTAL {{ r.objeto_seccion.superseccion.titulo }}
			  </td>  
			  <td> 
				{{ promedios_componentes.CPG.estudiante }} %
			  </td>
			  <td> 
				{{ promedios_componentes.CPG.docente }} %
			  </td>
			  <td> 
				{{ promedios_componentes.CPG.paracademico }} %
			  </td>
			  <td> 
				{{ promedios_componentes.CPG.directivo }} %
			  </td>
			  <td> 
				{{ promedios_componentes.CPG.primaria|floatformat:"0" }} %
			  </td>
			  <td> 
				{{ promedios_componentes.CPG.ponderada|floatformat:"2" }} / {{ r.objeto_seccion.superseccion.ponderacion }}
			  </td>
			  <td> 
				{{ promedios_componentes.CPG.cualitativa }} 
			  </td>
			</tr>
	 		{% endifequal %}

			{# Total PV #}
			{% ifequal forloop.counter promedios_componentes.PV.fila %}
			<tr class="promedio_componente"> 
			  <td colspan="2" style="text-align: left;"> 
				TOTAL {{ r.objeto_seccion.superseccion.titulo }}
			  </td>  
			  <td> 
				{{ promedios_componentes.PV.estudiante }} %
			  </td>
			  <td> 
				{{ promedios_componentes.PV.docente }} %
			  </td>
			  <td> 
				{{ promedios_componentes.PV.paracademico }} %
			  </td>
			  <td> 
				{{ promedios_componentes.PV.directivo }} %
			  </td>
			  <td> 
				{{ promedios_componentes.PV.primaria|floatformat:"0" }} %
			  </td>
			  <td> 
				{{ promedios_componentes.PV.ponderada|floatformat:"2" }} / {{ r.objeto_seccion.superseccion.ponderacion }}
			  </td>
			  <td> 
				{{ promedios_componentes.PV.cualitativa }} 
			  </td>
			</tr>
	 		{% endifequal %}
			{% endif %} <!-- end if promedios_componentes -->

			{% endfor %}
		  </table>

Tabla de Resultados::

	      <table>
			<tr>
			 <td colspan="2"> <b> VALOR TOTAL OBTENIDO: </b> </td>
			 <td width="10%"> {{ promedios.estudiante|floatformat:"2" }} % </td>
			 <td width="10%"> {{ promedios.docente|floatformat:"2" }} % </td>
			 <td width="10%"> {{ promedios.paracademico|floatformat:"2" }} % </td>
			 <td width="10%"> {{ promedios.directivo|floatformat:"2" }} % </td>
			 <td width="10%"> <b> {{ promedios.primaria }} %  </b> </td>
			 <td width="10%"> <b> {{ promedios.ponderada|floatformat:"2" }} / 100 </b> </td>
			 <td width="10%"> <b> {{ promedios.cualitativa }} </b> </td>
			 </b>
			</tr>
		  </table>

Tabla para escalas::

		  <table id="tabla-escala">
			<tr> 
			  <td style="text-align: left;"> 
				<b> D </b> = Destacado |
				<b> S </b> = Sastisfactorio | 
				<b> PS </b> = Poco Satistactorio |
				<b> IS </b> = Insatisfactorio
			  </td>
			</tr>
		  </table>

Sección final::

		  <a href='javascript:window.print();'> 
			<img src="http://capacinet.gob.mx/Cursos/Aprendamos%20Juntos/finanzaspersonales/imagenes/icono_imprimir.jpg"/> 
		  </a>
	      <div id="pie" style="font-size: 9px; color: #666666">
			<hr style="margin-bottom: 6px;"/>
			Unidad de Telecomunicaciones e Infomacion - Departamento de Desarrollo de Software <br>
			Correo Sistema de Evaluacion: 
			<a href="mailto:evaluacion.docente@unl.edu.ec"> <u>evaluacion.docente@unl.edu.ec</u> </a> <br/>
			Loja - Ecuador
	      </div>
	    </center>
	  </body>
	</html>



.. _template-user-imprimir_sugerencias_edd2013.html:

Sugerencias Evaluación Desempeño Docente 2013
---------------------------------------------

Plantilla para impresión de sugerencias en la Evaluación Desempeño Docente 2013.
Ubicación: ``{{TEMPLATE_DIRS}}/app/imprimir_sugerencias_edd2013.html``::

	<html>
	  <head>  
	    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
	    <title> Resultados Evaluación del Desempeño Académico 2012 - 2013 </title>

Sección de estilos::

	    <style type="text/css">
	    body {
		font-family: Helvetica, Arial, Sans;
	    }
	    table {
		margin-top: 10px;
		margin-bottom: 10px;
		width: 75%;
		borderspacing: 0;
	    }
	    th {
		text-align: center;
		border: 1px solid gray;
	        font-size: 13px;
	    }
	    td {
		text-align: left;
		border: 1px solid gray;
	        font-size: 13px;
	    }
	    textarea {
		font-size: 10px;
		overflow: hidden;
	    }

	    </style>
	  </head>

Inicio del body::

	  <body>
	    <center>
	      <img width="75px" height="75px" src="/static/images/unl_logo.png"/>
	      <h4> 
	      {{ "Unversidad Nacional de Loja"|upper }}  <br/>
	      Comisión de Evaluación Interna <br/>
	      EVALUACIÓN DEL DESEMPEÑO ACADÉMICO <p/>
	      {{ "Resultados de la Evaluación del Desempeño Académico 2012-2013"|upper }}

Tabla de encabezado::

	      <table id="tabla-docente" width="100%" border="0">
			{% if area %}
			<tr>
			  <td width="10%"><b>AREA</b></td> <td style="text-align: left;" width="90%">{{ area }}</td>
			</tr>
			{% endif %}
			{% if carrera %}
			<tr>
			  <td width="10%"><b>CARRERA</b></td> <td style="text-align: left;" width="90%">{{ carrera }}</td>
			</tr>
			{% endif %}
		  </table>

Tabla Central::

	      <table id="tabla-resultados" width="100%" cellspacing="0" >
			{% for docente, informantes in sugerencias.items %}
			<tr>
			  <td colspan="3"> <h2> {{ docente }} </h2></td>
			</tr>
			{% for informante, resultados in informantes.items %}
			<tr> 
			  <td colspan="3"> <h3> <center> INFORMANTE {{ informante|upper }}  </center> <h3> </td>
			</tr>
			<tr>
			  <h3> 
				<td width="33%"> <b> CAPACIDAD PROFESIONAL </b> </td>
				<td width="33%"> <b> CAPACIDAD PEDAGÓGICA </b> </td>
				<td width="33%"> <b> PRÁCTICA DE VALORES </b> </td>
			  </h3> 
			</tr>
			{% for r in resultados %}
			<tr>		
			  <td width="33%"> {{ r.CPF }} </td>
			  <td width="33%"> {{ r.CPG }} </td>
			  <td width="33%"> {{ r.PV }} </td>
			</tr>
			{% endfor %}
			{% endfor %}
			{% endfor %}
		  </table>

Sección final::

		  <a href='javascript:window.print();'> 
			<img src="http://capacinet.gob.mx/Cursos/Aprendamos%20Juntos/finanzaspersonales/imagenes/icono_imprimir.jpg"/>
		  </a>
		  <div id="pie" style="font-size: 9px; color: #666666">
			<hr style="margin-bottom: 6px;"/>
			Unidad de Telecomunicaciones e Infomacion - Departamento de Desarrollo de Software <br>
			Correo Sistema de Evaluacion: 
			<a href="mailto:evaluacion.docente@unl.edu.ec"> <u>evaluacion.docente@unl.edu.ec</u> </a> <br/>
			Loja - Ecuador
	      </div>
	    </center>
	  </body>
	</html>

..			<!--
			{% for r in resultados %}
			<tr> 
			  <td> {{ r }} </td>
			</tr>
			{% endfor %}
	-->
.. _template-user-menu_resultados_carrera.html:

Menú Resultados Carreras
-------------------------
Permite realizar la presentación de el munú con opciones para presentar y administrar los resultados de las evaluaciones.
Ubicación: ``{{TEMPLATE_DIRS}}/app/menu_resultados_carrera.html``::

	{% extends "admin/base_site.html" %}

	{% block extrahead %}

Estilos básicos::

	<style>
	  #id_periodos {
	      width: 100%;
	      font-size: 15px;
	      font-weight: bold;
	      color: #666666;
	  }
	</style>

	<script type="text/javascript" src="/static/js/jquery-1.6.2.min.js"></script>

Método AJAX para la obtención de datos::

	<script type="text/javascript">
	    $(document).ready(function(){
			$("#procesar").hide();
			$("#id_periodo_evaluacion").change(function(){
			    var id_periodo = $("#id_periodo_evaluacion option:selected").val();

Si existe un periodo académico entonces se genera el menú con las distintas opciones disponibles::

			    if (id_periodo != ""){
				$.ajax({
				    url: "/resultados/periodo/" + id_periodo + "/",
				    method: "GET",
				    //data: "periodo_academico_id={{ object_id }}",
				    success: function (data){
						$("#menu-resultados").html(data);
						$("#procesar").show();
						$("#borrar").show();
				    },
				    error: function(xhr,status,error){
						alert("error: " + status + "-" + error);
				    }
				});

Caso contrario no presentar nada::

			    }else{
					$("#menu-resultados").html("");
					$("#procesar").hide();
					$("#borrar").hide();
			    }
Fín del script::

			});
	    });
	</script>

Bloques heredados de la plantilla Base::

	{{ block.super }}

	//Cierre del bloque 'extrahead'

	{% endblock %}

	{% block content_title %}

	<!--h1> Resultados de la Carrera </h1-->

	{% endblock %}

Despliegue del formulario con una estructura ordenada por tablas::

	{% block content %}

	<form action="/resultados/mostrar/" method="POST" target="_blank">

	  {% csrf_token %} 

	  <table> 

	    {{ form.as_table }}

	  </table>

	  <!--legend> Opciones </legend-->

	  <div id="menu-resultados"> </div>
	  <p/>    
	  <input id="procesar" type="submit" value="Aceptar"/>
	  <input id="borrar" type="reset" value="Borrar"/>
	</form>

	{% endblock %}


.. _template-user-error404:

Error 404
---------
Esta plantilla permite realizar la presentación del error 404 (no encontrado ).
Ubicación: ``{{TEMPLATE_DIRS}}/404.html``::

	{% extends "admin/base_site.html"%}

	{% load i18n %}

Mensaje de página no encontrada::

	{% block title %} {% trans "Page not found" %} {% endblock %}

	{% block content %}

		<h1> {% trans "Page not found" %}. </h1>
		<p> Lo sentimos, pero la página solicitada no puede ser econtrada. </p>

	{% endblock %}



.. _template-user-error500:

Error 500
---------
Esta plantilla permite realizar la presentación del error 500 (Error interno del servidor).
Ubicación: ``{{TEMPLATE_DIRS}}/500.html``::

	<!DOCTYPE html PUBLIC "-//W3C//DTD HTML5.01//EN" "http://w3.org/TR/html4/strict.dtd">
	<html lang="en">
		<head>
		  <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
		  <title> Página no disponible </title>
		</head>
		<body>

Mensajes de error personalizados::

		  <h1> Página no disponible </h1>
		  <p> Lo sentimos, pero la página solicitada no está disponible debido a un error del servidor.!</p>
		  <p> Nuestros ingenieros serán notificados, y lo solucionarán luego.!</p>
		</body>
	</html>













