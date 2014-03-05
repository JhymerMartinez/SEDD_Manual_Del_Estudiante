************************************************************
Manual del Programador para el Sistema de Evaluación Docente
************************************************************

Instalación y Ejecución
=======================

1. Descargar e instalar python de http://www.python.org/getit/

2. Descargar el paquete comprimido de Sphinx disponible en: https://pypi.python.org/pypi/Sphinx

3. Configurar el path agregando estas 2 direcciones::

    {%ruta_instalación%}/Python27
    {%ruta_instalación%}/Python27/Scripts

Donde ``{%ruta_instalación%}`` contiene la dirección donde se instaló python por ejemplo en Windows ``C:\Python27`` o en Linux ``/usr/local/bin/Python27``.

4. Para instalar sphinx:

 - Descomprimir el archivo descargado
 - Ingresar a la carpeta y ejecutar ``python setup.py install``

5. Para ejecutar:

 - Ir a la carpeta donde se clonaron los archivos
 - Ingresar a ``_build/html``
 - Ejecutar ``index.html``
 - Listo

6. Cualquier modificación en los archivos ``.rst`` requiere una nueva compilación para poder apreciarlos, haciendo uso del comando ``make html`` en la raiz del proyecto.

Requerimientos
==============

 * Python 2.7
 * Python-sphinx 1.2
 * rst2pdf 0.93 (para generar el pdf, aun esta en desarrollo)
 * sphinx-bootstrap-theme 0.2.7

