# 🛡️ Threat Intelligence & OSINT Report: Overheard at Breakfast (TryHackMe)
**Auditora:** Sofia Hernandez Betancur ([sofiH21](https://github.com/sofiH21))  
**Categoría:** Open Source Intelligence (OSINT) & Digital Forensics  
**Nivel de Riesgo:** Informativo / Exposición de Identidad Digital  
---
## 📄 Resumen Ejecutivo
Este informe documenta el proceso de investigación de fuentes abiertas (OSINT) para identificar la huella digital y cuenta expuesta de un objetivo a partir de fragmentos de conversaciones e información de correo electrónico.
---
## 🔍 Metodología de Investigación & Evidencia
### 1. Extracción de Pistas e Identificadores
A partir del análisis de una captura de conversación en redes sociales, se identificaron los siguientes vectores de inteligencia:
- **Correo objetivo:** `lambobytelotushotel@gmail.com`
- **Pista de plataforma:** Servicio para enlazar perfiles iniciado por la letra "G" (Gravatar).
### 2. Generación de Criptografía de Hashes (MD5)
Las plataformas como Gravatar asocian perfiles de usuario utilizando el hash MD5 de la dirección de correo electrónico en minúsculas.
En la terminal de Kali Linux se calculó el hash MD5 correspondiente:
```bash
echo -n "lambobytelotushotel@gmail.com" | md5sum
# Resultado: d4a5fc5d3128890778667e24617d7cc0
3. Consultas a la API de Gravatar & Decodificación
Se realizó una petición a la API pública de Gravatar mediante curl:

bash
curl -s https://www.gravatar.com/d4a5fc5d3128890778667e24617d7cc0.json
En la respuesta en formato JSON, dentro del campo aboutMe, se identificó una cadena codificada en Base64:

text
VEhNe1MzY3JlVF9QcjBmaWwzX0g0c19iMzNuX0lkZW50MWZpM2R9
Se procedió a la decodificación del valor Base64 desde la terminal:

bash
echo "VEhNe1MzY3JlVF9QcjBmaWwzX0g0c19iMzNuX0lkZW50MWZpM2R9" | base64 -d
# Flag extraída: THM{S3creT_Pr0fil3_H4s_b33n_Ident1fi3d}
🛡️ Lecciones de Privacidad & Seguridad (OPSEC)
Evitar la reutilización de correos electrónicos: La correlación de hashes MD5 permite asociar identidades de correo a cuentas en diversas plataformas públicas.
Sanitización de Metadatos Públicos: No almacenar información confidencial o claves en biografías o descripciones públicas de perfiles web.
