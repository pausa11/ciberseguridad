# Vulnerabilidades seleccionadas — OWASP Juice Shop

Tres vulnerabilidades cubriendo las categorías A01, A02 y A03 del **OWASP Top 10 (2021)**.

Entorno:
- **Atacante:** Kali Linux `192.168.40.3`
- **Víctima:** Ubuntu 22.04 + Juice Shop `192.168.40.11:3000`

---

## Vulnerabilidad 1 — SQL Injection: Bypass de Autenticación

| Campo | Detalle |
|---|---|
| **OWASP 2021** | A03 — Inyección |
| **CWE** | CWE-89 (SQL Injection) |
| **Dificultad** | ⭐ Baja |
| **Impacto** | Crítico — acceso como administrador sin credenciales válidas |

### Descripción

El formulario de login de Juice Shop construye la consulta SQL concatenando directamente los valores ingresados por el usuario. Al inyectar `' OR 1=1--` como email, se convierte la condición WHERE en una tautología y la aplicación autentica al primer usuario de la base de datos (que es el administrador).

Consulta original (simplificada):
```sql
SELECT * FROM Users WHERE email = '<INPUT>' AND password = '<HASH>'
```
Con el payload:
```sql
SELECT * FROM Users WHERE email = '' OR 1=1--' AND password = '<HASH>'
```

### Herramientas
- **Navegador web** (Firefox en Kali) — ejecución manual del payload
- **Burp Suite** — intercepción y análisis del tráfico HTTP

### Pasos de explotación
1. Navegar a `http://192.168.40.11:3000/#/login`
2. En el campo **Email** ingresar: `' OR 1=1--`
3. En el campo **Password** ingresar cualquier valor (ej. `a`)
4. Hacer clic en **Log in**
5. Verificar acceso como `admin@juice-sh.op` en la esquina superior derecha

### Capturas a tomar (`img/tarea1/`)
| Archivo | Contenido |
|---|---|
| `t1_login_form.png` | Formulario de login con el payload ingresado |
| `t1_login_success.png` | Pantalla principal tras acceso exitoso como admin |
| `t1_burp_request.png` | Petición HTTP capturada en Burp Suite con el payload |

### Texto de shell a guardar (`shellText/kali/tarea1/`)
- No aplica (ataque vía navegador); si se usa `sqlmap`: guardar salida como `sqlmap.txt`

### Resultado esperado
Inicio de sesión exitoso como administrador (`admin@juice-sh.op`) sin conocer la contraseña.

---

## Vulnerabilidad 2 — Sensitive Data Exposure: Directorio /ftp Expuesto

| Campo | Detalle |
|---|---|
| **OWASP 2021** | A02 — Fallos Criptográficos / Exposición de Datos Sensibles |
| **CWE** | CWE-538 (File and Directory Information Exposure) |
| **Dificultad** | ⭐ Baja |
| **Impacto** | Alto — descarga de archivos confidenciales del servidor |

### Descripción

Juice Shop expone un directorio `/ftp` sin ningún control de acceso ni autenticación. Cualquier usuario (o atacante) que conozca la ruta puede listar y descargar archivos sensibles como documentos de adquisiciones corporativas, copias de seguridad de configuración y contratos.

### Herramientas
- **Navegador web** — navegación directa al directorio
- **curl / wget** — descarga automatizada de archivos desde Kali Linux

### Pasos de explotación
1. Navegar a `http://192.168.40.11:3000/ftp`
2. Observar el listado de archivos expuestos
3. Descargar el archivo `acquisitions.md` (contiene información corporativa sensible):
   ```bash
   curl http://192.168.40.11:3000/ftp/acquisitions.md -o acquisitions.md
   cat acquisitions.md
   ```
4. Intentar descargar `package.json.bak` para obtener información sobre dependencias:
   ```bash
   curl http://192.168.40.11:3000/ftp/package.json.bak -o package.json.bak
   ```

### Capturas a tomar (`img/tarea2/`)
| Archivo | Contenido |
|---|---|
| `t2_ftp_directory.png` | Listado del directorio `/ftp` en el navegador |
| `t2_acquisitions_download.png` | Descarga y contenido de `acquisitions.md` en terminal |
| `t2_curl_output.png` | Salida de `curl` descargando los archivos |

### Texto de shell a guardar (`shellText/kali/tarea2/`)
- `curl_ftp.txt` — comandos curl y su salida
- `acquisitions.txt` — contenido del archivo descargado

### Resultado esperado
Descarga exitosa de archivos confidenciales (planes de adquisición, configuración) sin ninguna autenticación.

---

## Vulnerabilidad 3 — Broken Access Control: API de Usuarios sin Autenticación

| Campo | Detalle |
|---|---|
| **OWASP 2021** | A01 — Control de Acceso Roto |
| **CWE** | CWE-284 (Improper Access Control) |
| **Dificultad** | ⭐ Baja–Media |
| **Impacto** | Alto — enumeración completa de usuarios registrados (emails, roles, hashes) |

### Descripción

El endpoint REST `/api/Users` de Juice Shop no requiere autenticación. Cualquier petición GET a esa ruta devuelve el listado completo de todos los usuarios registrados en la aplicación, incluyendo direcciones de correo electrónico, roles, y hashes de contraseña. Esto constituye una violación del principio de mínimo privilegio y una falla crítica de control de acceso.

### Herramientas
- **curl** — petición directa al endpoint desde Kali Linux
- **Firefox DevTools / Burp Suite** — inspección de la respuesta JSON

### Pasos de explotación
1. Sin iniciar sesión, ejecutar desde Kali:
   ```bash
   curl -s http://192.168.40.11:3000/api/Users | python3 -m json.tool
   ```
2. Observar en la respuesta JSON los campos: `email`, `role`, `password` (hash), `createdAt`
3. (Opcional) Filtrar solo emails con `jq`:
   ```bash
   curl -s http://192.168.40.11:3000/api/Users | python3 -c "import sys,json; [print(u['email']) for u in json.load(sys.stdin)['data']]"
   ```

### Capturas a tomar (`img/tarea3/`)
| Archivo | Contenido |
|---|---|
| `t3_api_users_raw.png` | Salida completa de `curl` con la lista de usuarios |
| `t3_api_users_json.png` | JSON formateado mostrando emails, roles y hashes |
| `t3_browser_api.png` | Respuesta del endpoint en el navegador (sin login) |

### Texto de shell a guardar (`shellText/kali/tarea3/`)
- `curl_api_users.txt` — salida completa de curl con todos los usuarios
- `emails_extracted.txt` — lista de emails extraídos

### Resultado esperado
Obtención de la base de usuarios completa (emails, hashes de contraseña, roles) sin ningún token de autenticación ni sesión activa.

---

## Resumen de cobertura OWASP

| # | Vulnerabilidad | OWASP 2021 | Impacto |
|---|---|---|---|
| 1 | SQL Injection — Login bypass | A03 Inyección | Admin sin credenciales |
| 2 | Directorio /ftp expuesto | A02 Datos Sensibles | Archivos confidenciales |
| 3 | API /api/Users sin autenticación | A01 Control de Acceso | Enumeración de usuarios |
