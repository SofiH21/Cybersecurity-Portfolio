# 🛡️ Technical Report: Bypass PHP disable_functions (TryHackMe)
**Auditora:** Sofia Hernandez Betancur ([sofiH21](https://github.com/sofiH21))  
**Categoría:** Web Application Security & Evasion Techniques  
**Nivel de Riesgo:** Alto (High - Arbitrary Command Execution)  
**Objetivo:** `10.66.142.141` (Ubuntu Linux / Apache / PHP 7.0)
---
## 📄 Resumen Ejecutivo
Durante la auditoría sobre la aplicación web "Ecorp Jobs", se identificó un formulario de subida de archivos destinado a recibir CVs en formato de imagen. A pesar de que el servidor contaba con directivas de seguridad en `php.ini` (`disable_functions`) para bloquear funciones peligrosas de ejecución de comandos (`system`, `shell_exec`, `exec`), se logró evadir estas restricciones combinando técnicas de manipulación de variables de entorno (`LD_PRELOAD`) y funciones no filtradas (`mail` y `putenv`).
---
## 🔍 Cadena de Ataque & Evidencia Técnica
### 1. Reconocimiento e Inspección de Configuración PHP
Un escaneo con `Gobuster` reveló la existencia de `/uploads/` y `/phpinfo.php`. Al inspeccionar `phpinfo.php`, se identificó la lista de funciones deshabilitadas:
```text
disable_functions: exec, passthru, shell_exec, system, proc_open, popen, curl_exec...
Sin embargo, las funciones putenv() y mail() permanecían habilitadas.

2. Evasión de Filtro de Subida (Magic Bytes)
El servidor validaba la cabecera de las imágenes. Para evadir este filtro y lograr la subida de scripts PHP, se inyectaron los magic bytes de una imagen GIF (GIF89a) al inicio del payload malicioso:

bash
echo -e 'GIF89a<?php ... ?>' > test.php
3. Explotación mediante LD_PRELOAD y Chankro
Se utilizó la herramienta Chankro para generar una shared library (.so) maliciosa que intercepta llamadas del sistema cuando la función mail() invoca al binario sendmail.

Flujo de Ejecución:

putenv("LD_PRELOAD=/ruta/a/libreria.so") define la variable de entorno.
mail(...) invoca un proceso secundario del sistema operativo.
El sistema operativo precarga la librería maliciosa ejecutando el script Bash deseado.
Mediante el descubrimiento de la ruta exacta del servidor (DOCUMENT_ROOT: /var/www/html/fa5fba5f5a39d27d8bb7fe5f518e00db/uploads), se ejecutó la búsqueda y extracción de la bandera del laboratorio.

🛡️ Recomendaciones de Mitigación
Endurecimiento de disable_functions: Agregar putenv, mail y error_log a la lista de funciones restringidas en php.ini si no son estrictamente necesarias para el negocio.

ini
disable_functions = system, shell_exec, exec, passthru, proc_open, popen, putenv, mail
Validación Estricta de Subida de Archivos: No confiar únicamente en la extensión del archivo o la cabecera de bytes (magic bytes). Re-codificar las imágenes en el servidor usando librerías como GD o ImageMagick para eliminar cualquier código inyectado.
