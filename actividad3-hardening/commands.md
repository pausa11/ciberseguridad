     ╭───────────────────────────────────────────────────────────────────────────────────────╮
     │ Plan: Actividad #3 — Hardening con openSCAP en Ubuntu Server 24.04 (UTM / Mac M4)     │
     │                                                                                       │
     │ Contexto                                                                              │
     │                                                                                       │
     │ La actividad requiere aplicar un proceso de hardening a Ubuntu Server 24.04 usando    │
     │ openSCAP y la política SCAP Security Guide (ssg-ubuntu2404-ds.xml). El usuario tiene  │
     │ un Mac Mini M4 con UTM y ya tiene el ISO descargado. La fecha límite es el 6 de junio │
     │ de 2026 (mañana).                                                                     │
     │                                                                                       │
     │ Problema de plataforma: openSCAP Workbench es una aplicación GUI de Linux/Windows; no │
     │ corre en macOS de forma nativa. La solución es usar las herramientas CLI de openSCAP  │
     │ (oscap) directamente dentro de la VM Ubuntu Server, que producen los mismos reportes  │
     │ HTML que el Workbench.                                                                │
     │                                                                                       │
     │ ---                                                                                   │
     │ Fase 1 — Instalar y configurar la VM en UTM                                           │
     │                                                                                       │
     │ Pasos en UTM (Mac):                                                                   │
     │ 1. Abrir UTM → "Create a New Virtual Machine" → "Virtualize" → Linux                  │
     │ 2. Seleccionar el ISO de Ubuntu Server 24.04 ya descargado                            │
     │ 3. Configuración recomendada para M4:                                                 │
     │   - CPU: 2 cores                                                                      │
     │   - RAM: 2 GB                                                                         │
     │   - Disco: 20 GB                                                                      │
     │   - Activar "Enable Clipboard Sharing" y "Enable Directory Sharing" (para transferir  │
     │ reportes al Mac fácilmente)                                                           │
     │ 4. Instalar Ubuntu Server con opciones mínimas (sin extras salvo OpenSSH server —     │
     │ marcarlo para facilitar acceso)                                                       │
     │ 5. Al terminar la instalación, iniciar sesión en la VM                                │
     │                                                                                       │
     │ ---                                                                                   │
     │ Fase 2 — Instalar openSCAP en la VM Ubuntu Server                                     │
     │                                                                                       │
     │ Dentro de la VM, ejecutar:                                                            │
     │                                                                                       │
     │ sudo apt update && sudo apt upgrade -y                                                │
     │ sudo apt install -y libopenscap8 openscap-scanner                                     │
     │                                                                                       │
     │ Verificar instalación:                                                                │
     │ oscap --version                                                                       │
     │                                                                                       │
     │ ---                                                                                   │
     │ Fase 3 — Descargar la SCAP Security Guide (SSG)                                       │
     │                                                                                       │
     │ Dentro de la VM:                                                                      │
     │                                                                                       │
     │ # Opción A: desde repositorio oficial de Ubuntu (más simple)                          │
     │ sudo apt install -y ssg-base ssg-debderived                                           │
     │                                                                                       │
     │ # Verificar que existe el archivo de política                                         │
     │ ls /usr/share/xml/scap/ssg/content/ssg-ubuntu2404-ds.xml                              │
     │                                                                                       │
     │ Si el paquete no tiene Ubuntu 24.04, usar la opción B:                                │
     │                                                                                       │
     │ # Opción B: descargar release desde GitHub (ComplianceAsCode)                         │
     │ wget https://github.com/ComplianceAsCode/content/releases/download/v0.1.75/scap-secur │
     │ ity-guide-0.1.75.zip                                                                  │
     │ sudo apt install -y unzip                                                             │
     │ unzip scap-security-guide-0.1.75.zip                                                  │
     │ # El archivo estará en: scap-security-guide-0.1.75/ssg-ubuntu2404-ds.xml              │
     │                                                                                       │
     │ ---                                                                                   │
     │ Fase 4 — Escaneo inicial (Reporte 1)                                                  │
     │                                                                                       │
     │ # Definir la ruta al archivo de política (ajustar según la opción usada arriba)       │
     │ SSG="/usr/share/xml/scap/ssg/content/ssg-ubuntu2404-ds.xml"                           │
     │ # O si se usó opción B:                                                               │
     │ # SSG="$HOME/scap-security-guide-0.1.75/ssg-ubuntu2404-ds.xml"                        │
     │                                                                                       │
     │ # Ejecutar escaneo con perfil CIS Level 1 (recomendado para servidores)               │
     │ sudo oscap xccdf eval \                                                               │
     │   --profile xccdf_org.ssgproject.content_profile_cis_level1_server \                  │
     │   --results /tmp/resultado-inicial.xml \                                              │
     │   --report /tmp/reporte-inicial.html \                                                │
     │   "$SSG"                                                                              │
     │                                                                                       │
     │ ▎ Nota: Si el perfil CIS no existe, listar perfiles disponibles con:                  │
     │ ▎ oscap info "$SSG" y elegir uno del listado.                                         │
     │                                                                                       │
     │ El reporte HTML se guarda en /tmp/reporte-inicial.html. Se puede copiar al Mac        │
     │ mediante la carpeta compartida de UTM o scp.                                          │
     │                                                                                       │
     │ ---                                                                                   │
     │ Fase 5 — Remediaciones                                                                │
     │                                                                                       │
     │ Método A — Script bash automático (recomendado para maximizar el score):              │
     │                                                                                       │
     │ sudo oscap xccdf generate fix \                                                       │
     │   --fix-type bash \                                                                   │
     │   --profile xccdf_org.ssgproject.content_profile_cis_level1_server \                  │
     │   --output /tmp/remediation.sh \                                                      │
     │   "$SSG"                                                                              │
     │                                                                                       │
     │ # Revisar el script antes de ejecutar                                                 │
     │ less /tmp/remediation.sh                                                              │
     │                                                                                       │
     │ # Ejecutar remediaciones                                                              │
     │ sudo bash /tmp/remediation.sh 2>&1 | tee /tmp/remediation-log.txt                     │
     │                                                                                       │
     │ Método B — Remediación directa durante el escaneo:                                    │
     │                                                                                       │
     │ sudo oscap xccdf eval \                                                               │
     │   --remediate \                                                                       │
     │   --profile xccdf_org.ssgproject.content_profile_cis_level1_server \                  │
     │   --results /tmp/resultado-remediated.xml \                                           │
     │   --report /tmp/reporte-post-remediation.html \                                       │
     │   "$SSG"                                                                              │
     │                                                                                       │
     │ Después de las remediaciones, reiniciar la VM:                                        │
     │ sudo reboot                                                                           │
     │                                                                                       │
     │ ---                                                                                   │
     │ Fase 6 — Escaneo final (Reporte 2)                                                    │
     │                                                                                       │
     │ Luego del reinicio, ejecutar el escaneo nuevamente:                                   │
     │                                                                                       │
     │ sudo oscap xccdf eval \                                                               │
     │   --profile xccdf_org.ssgproject.content_profile_cis_level1_server \                  │
     │   --results /tmp/resultado-final.xml \                                                │
     │   --report /tmp/reporte-final.html \                                                  │
     │   "$SSG"                                                                              │
     │                                                                                       │
     │ Guardar ambos reportes: reporte-inicial.html y reporte-final.html.                    │
     │                                                                                       │
     │ ---                                                                                   │
     │ Fase 7 — Transferir reportes al Mac                                                   │
     │                                                                                       │
     │ Con la carpeta compartida de UTM (la más fácil):                                      │
     │ cp /tmp/reporte-inicial.html /media/share/                                            │
     │ cp /tmp/reporte-final.html /media/share/                                              │
     │                                                                                       │
     │ O por SSH/SCP desde el Mac:                                                           │
     │ # En el Mac (Terminal)                                                                │
     │ scp usuario@IP-DE-LA-VM:/tmp/reporte-inicial.html ~/Desktop/                          │
     │ scp usuario@IP-DE-LA-VM:/tmp/reporte-final.html ~/Desktop/                            │
     │                                                                                       │
     │ Los archivos HTML se pueden abrir directamente en Chrome/Safari para visualizarlos.   │
     │                                                                                       │
     │ ---                                                                                   │
     │ Fase 8 — Redactar el informe APA en LaTeX                                             │
     │                                                                                       │
     │ El archivo src/actividad3.tex ya existe (actualmente vacío). Llenarlo con:            │
     │                                                                                       │
     │ 1. Portada con datos institucionales (UTP, Especialización TIC, Ciberseguridad)       │
     │ 2. Introducción: qué es hardening y por qué es importante                             │
     │ 3. Marco teórico: openSCAP, SCAP, CIS Benchmarks, SCAP Security Guide                 │
     │ 4. Metodología: pasos seguidos (con capturas de pantalla en img/)                     │
     │ 5. Resultados: comparación de score antes y después                                   │
     │ 6. Conclusiones                                                                       │
     │ 7. Referencias en formato APA                                                         │
     │ 8. Anexos: los dos reportes HTML (referenciados o incluidos)                          │
     │                                                                                       │
     │ Imágenes a capturar para el informe:                                                  │
     │ - Instalación de UTM/VM                                                               │
     │ - Instalación de openSCAP                                                             │
     │ - Ejecución del escaneo inicial (terminal)                                            │
     │ - Reporte HTML inicial (screenshot del navegador)                                     │
     │ - Ejecución de remediaciones                                                          │
     │ - Reporte HTML final (screenshot del navegador)                                       │
     │                                                                                       │
     │ ---                                                                                   │
     │ Archivos críticos a modificar                                                         │
     │                                                                                       │
     │ - src/actividad3.tex — documento principal del informe (actualmente vacío, escribir   │
     │ desde cero)                                                                           │
     │ - img/ — agregar capturas de pantalla numeradas                                       │
     │ - (nuevos) reports/reporte-inicial.html y reports/reporte-final.html — reportes       │
     │ generados por oscap                                                                   │
     │                                                                                       │
     │ ---                                                                                   │
     │ Verificación                                                                          │
     │                                                                                       │
     │ 1. oscap --version retorna versión válida dentro de la VM                             │
     │ 2. ls ssg-ubuntu2404-ds.xml confirma que el archivo de política existe                │
     │ 3. El reporte inicial genera un porcentaje de cumplimiento (ej. 30–40% típico en      │
     │ instalación base)                                                                     │
     │ 4. El reporte final muestra un porcentaje mayor (ej. 60–80%) tras las remediaciones   │
     │ 5. Ambos archivos .html se pueden abrir en el navegador y muestran el dashboard de    │
     │ resultados                                                                            │
     │ 6. El PDF del informe compila sin errores con pdflatex actividad3.tex                 │
     │                                                                                       │
     │ ---                                                                                   │
     │ Riesgo y nota importante                                                              │
     │                                                                                       │
     │ - Las remediaciones del script pueden cambiar configuraciones de red/SSH. Hacer       │
     │ snapshot de la VM en UTM antes del paso de remediación.                               │
     │ - En UTM (Mac M4, ARM): Ubuntu Server 24.04 para ARM64. Confirmar que el ISO          │
     │ descargado es la versión arm64, no amd64.                                             │
     ╰───────────────────────────────────────────────────────────────────────────────────────╯

❯ ubuntu@ubuntu:~$ sudo apt install -y libopenscap8 openscap-scanner
  Reading package lists... Done
  Building dependency tree... Done
  Reading state information... Done
  Package libopenscap8 is not available, but is referred to by another package.
  This may mean that the package is missing, has been obsoleted, or
  is only available from another source
  However the following packages replace it:
    openscap-common libopenscap25t64

  E: Package 'libopenscap8' has no installation candidate
  ubuntu@ubuntu:~$

⏺ El paquete cambió de nombre en Ubuntu 24.04. Usa este comando en su lugar:

  sudo apt install -y openscap-scanner openscap-common

  Verifica después con:

  oscap --version

✻ Brewed for 4s

※ recap: Estás instalando openSCAP en la VM Ubuntu Server para hacer el hardening de la
  actividad. El siguiente paso es ejecutar `oscap --version` para confirmar que quedó
  instalado correctamente. (disable recaps in /config)

❯ ubuntu@ubuntu:/usr/share/xml/scap/ssg/content$ ls
  ssg-ubuntu1604-cpe-dictionary.xml
  ssg-ubuntu1604-cpe-oval.xml
  ssg-ubuntu1604-ds-1.2.xml
  ssg-ubuntu1604-ds.xml
  ssg-ubuntu1604-ocil.xml
  ssg-ubuntu1604-oval.xml
  ssg-ubuntu1604-xccdf.xml
  ssg-ubuntu1804-cpe-dictionary.xml
  ssg-ubuntu1804-cpe-oval.xml
  ssg-ubuntu1804-ds-1.2.xml
  ssg-ubuntu1804-ds.xml
  ssg-ubuntu1804-ocil.xml
  ssg-ubuntu1804-oval.xml
  ssg-ubuntu1804-xccdf.xml
  ssg-ubuntu2004-cpe-dictionary.xml
  ssg-ubuntu2004-cpe-oval.xml
  ssg-ubuntu2004-ds-1.2.xml
  ssg-ubuntu2004-ds.xml
  ssg-ubuntu2004-ocil.xml
  ssg-ubuntu2004-oval.xml
  ssg-ubuntu2004-xccdf.xml
  ssg-ubuntu2204-cpe-dictionary.xml
  ssg-ubuntu2204-cpe-oval.xml
  ssg-ubuntu2204-ds-1.2.xml
  ssg-ubuntu2204-ds.xml
  ssg-ubuntu2204-ocil.xml
  ssg-ubuntu2204-oval.xml
  ssg-ubuntu2204-xccdf.xml

⏺ El paquete ssg-debderived del repositorio no incluye Ubuntu 24.04 todavía. Necesitas
  descargar el SSG directamente desde GitHub:

  cd ~
  wget https://github.com/ComplianceAsCode/content/releases/download/v0.1.75/scap-security-gui
  de-0.1.75.zip
  sudo apt install -y unzip
  unzip scap-security-guide-0.1.75.zip
  ls scap-security-guide-0.1.75/ssg-ubuntu2404-ds.xml

  Si el archivo existe, ya puedes usarlo con esa ruta. Avísame el resultado.

✻ Crunched for 5s

❯ ubuntu@ubuntu:~$ sudo oscap xccdf eval --profile
  xccdf_org.ssgproject.content_profile_cis_level1_server --results /tmp/resultado-inicial.xml
  --report /tmp/reporte-inicial.html "$SSG"
  OpenSCAP Error: Unable to open file:
  '/home/ubuntu/scap-security-guide-0.1.75/ssg-ubuntu2404-ds.xml'
  [./src/source/oscap_source.c:298]
  ubuntu@ubuntu:~$  instale la # Opción B: descargar release desde GitHub (ComplianceAsCode)

⏺ La variable $SSG no está definida. Establécela primero y luego ejecuta el escaneo:

  SSG="$HOME/scap-security-guide-0.1.75/ssg-ubuntu2404-ds.xml"
  sudo oscap xccdf eval \
    --profile xccdf_org.ssgproject.content_profile_cis_level1_server \
    --results /tmp/resultado-inicial.xml \
    --report /tmp/reporte-inicial.html \
    "$SSG"

✻ Churned for 3s

❯ ubuntu@ubuntu:~$ SSG="$HOME/scap-security-guide-0.1.75/ssg-ubuntu2404-ds.xml"
    sudo oscap xccdf eval \
      --profile xccdf_org.ssgproject.content_profile_cis_level1_server \
      --results /tmp/resultado-inicial.xml \
      --report /tmp/reporte-inicial.html \
      "$SSG"
  OpenSCAP Error: Unable to open file:
  '/home/ubuntu/scap-security-guide-0.1.75/ssg-ubuntu2404-ds.xml'
  [./src/source/oscap_source.c:298]
  ubuntu@ubuntu:~$ ls
  scap-security-guide-0.1.75  scap-security-guide-0.1.75.zip
  ubuntu@ubuntu:~$

⏺ El directorio existe pero el archivo puede tener un nombre distinto. Verifica:

  ls ~/scap-security-guide-0.1.75/ | grep ubuntu2404

✻ Baked for 2s

❯ ubuntu@ubuntu:~/scap-security-guide-0.1.75$ ls
  Contributors.md           ssg-macos1015-ds.xml
  LICENSE                   ssg-ocp4-ds.xml
  README.md                 ssg-ol10-ds.xml
  ansible                   ssg-ol7-ds.xml
  bash                      ssg-ol8-ds.xml
  guides                    ssg-ol9-ds.xml
  kickstart                 ssg-openembedded-ds.xml
  manifests                 ssg-openeuler2203-ds.xml
  ssg-al2023-ds.xml         ssg-opensuse-ds.xml
  ssg-alinux2-ds.xml        ssg-rhcos4-ds.xml
  ssg-alinux3-ds.xml        ssg-rhel10-ds.xml
  ssg-anolis23-ds.xml       ssg-rhel8-ds.xml
  ssg-anolis8-ds.xml        ssg-rhel9-ds.xml
  ssg-centos8-ds.xml        ssg-rhv4-ds.xml
  ssg-chromium-ds.xml       ssg-sle12-ds.xml
  ssg-cs10-ds.xml           ssg-sle15-ds.xml
  ssg-cs9-ds.xml            ssg-slmicro5-ds.xml
  ssg-debian11-ds.xml       ssg-ubuntu1604-ds.xml
  ssg-debian12-ds.xml       ssg-ubuntu1804-ds.xml
  ssg-eks-ds.xml            ssg-ubuntu2004-ds.xml
  ssg-fedora-ds.xml         ssg-ubuntu2204-ds.xml
  ssg-firefox-ds.xml        tables
  ssg-kylinserver10-ds.xml

⏺ La versión 0.1.75 no incluye Ubuntu 24.04. Necesitas una versión más reciente. Voy a
  actualizar el plan con la versión correcta.

  Read 1 file (ctrl+o to expand)

⏺ Updated plan
  ⎿  /plan to preview

⏺ Updated plan
  ⎿  /plan to preview

⏺ Updated plan
  ⎿  /plan to preview

⏺ El problema es claro: la versión 0.1.75 no incluye Ubuntu 24.04. El soporte se añadió en
  versiones posteriores. Ejecuta esto en la VM para obtener la versión correcta:

  # Eliminar la descarga anterior
  rm -rf ~/scap-security-guide-0.1.75 ~/scap-security-guide-0.1.75.zip

  # Descargar v0.1.76 (primera versión con Ubuntu 24.04)
  wget https://github.com/ComplianceAsCode/content/releases/download/v0.1.76/scap-security-gui
  de-0.1.76.zip
  unzip scap-security-guide-0.1.76.zip
  ls ~/scap-security-guide-0.1.76/ssg-ubuntu2404-ds.xml

  Si el archivo aparece en el ls, luego define la variable y ejecuta el escaneo:

  SSG="$HOME/scap-security-guide-0.1.76/ssg-ubuntu2404-ds.xml"
  sudo oscap xccdf eval \
    --profile xccdf_org.ssgproject.content_profile_cis_level1_server \
    --results /tmp/resultado-inicial.xml \
    --report /tmp/reporte-inicial.html \
    "$SSG"

  Avísame si aparece el archivo o si hay otro error.

✻ Worked for 48s

※ recap: Estás haciendo hardening de Ubuntu Server 24.04 con openSCAP para la actividad
  universitaria. La versión 0.1.75 del SSG no incluye Ubuntu 24.04; el siguiente paso es
  descargar la versión 0.1.76 desde GitHub y verificar que el archivo `ssg-ubuntu2404-ds.xml`
  esté presente. (disable recaps in /config)

❯ hagamos un scp a la mac de los reportes iniciales: ubuntu@ubuntu:/tmp$ ls
  reporte-inicial.html
  resultado-inicial.xml

⏺ Primero verifica la IP de la VM desde Ubuntu:

  ip a | grep "inet " | grep -v 127.0.0.1

  Luego, en la Terminal del Mac, ejecuta:

  scp ubuntu@<IP-DE-LA-VM>:/tmp/reporte-inicial.html ~/Desktop/
  scp ubuntu@<IP-DE-LA-VM>:/tmp/resultado-inicial.xml ~/Desktop/

  scp ubuntu@<IP-DE-LA-VM>:/tmp/reporte-inicial.html ~/Desktop/
  scp ubuntu@<IP-DE-LA-VM>:/tmp/resultado-inicial.xml ~/Desktop/

  ¿Cuál es la IP que muestra el ip a?

✻ Cooked for 5s

❯ 192.168.64.13

⏺ Ejecuta esto en la Terminal de tu Mac:

  scp ubuntu@192.168.64.13:/tmp/reporte-inicial.html ~/Desktop/
  scp ubuntu@192.168.64.13:/tmp/resultado-inicial.xml ~/Desktop/

  Te pedirá la contraseña de la VM. Los archivos quedarán en el Escritorio.

✻ Cooked for 2s

❯ ya puedo correr las remediaciones

⏺ Antes de remediar, haz un snapshot de la VM en UTM por si algo se rompe (especialmente SSH o red).

  Luego, en la VM ejecuta:

  SSG="$HOME/scap-security-guide-0.1.76/ssg-ubuntu2404-ds.xml"

  sudo oscap xccdf eval \
    --remediate \
    --profile xccdf_org.ssgproject.content_profile_cis_level1_server \
    --results /tmp/resultado-post-remediation.xml \
    --report /tmp/reporte-post-remediation.html \
    "$SSG"

  Este proceso puede tardar varios minutos. Al terminar, reinicia:


✻ Sautéed for 4s

※ recap: Estamos haciendo hardening de Ubuntu Server 24.04 con openSCAP para la Actividad #3. Las remediaciones ya corrieron y la VM está reiniciando — el siguiente paso es ejecutar el
  escaneo final y hacer SCP del reporte al Mac. (disable recaps in /config)

❯ cuanto puede durar la remediacion?[Image #1] lleva asi mas de 15 minutosss
  ⎿  [Image #1]

⏺ Es normal, déjalo correr. La remediación CIS Level 1 aplica decenas de controles (configuración de SSH, auditoría, permisos de archivos, módulos del kernel, etc.) y puede durar entre 20 y
  45 minutos en una VM limpia.

  Está trabajando — el --- Starting Remediation --- confirma que ya pasó la fase de escaneo y está aplicando los fixes. Solo espera.

✻ Brewed for 5s

❯ ya hice el reboot

⏺ Perfecto. Ahora ejecuta el escaneo final:

  SSG="$HOME/scap-security-guide-0.1.76/ssg-ubuntu2404-ds.xml"

  sudo oscap xccdf eval \
    --profile xccdf_org.ssgproject.content_profile_cis_level1_server \
    --results /tmp/resultado-final.xml \
    --report /tmp/reporte-final.html \
    "$SSG"

  Al terminar, copia los reportes al Mac:

  # Desde la Terminal del Mac
  scp ubuntu@192.168.64.13:/tmp/reporte-final.html ~/Desktop/
  scp ubuntu@192.168.64.13:/tmp/resultado-final.xml ~/Desktop/

✻ Baked for 5s

※ recap: Hardening de Ubuntu Server 24.04 con openSCAP para la Actividad #3. El escaneo final está corriendo en la VM; cuando termine, copiar `reporte-final.html` al Mac con `scp
  ubuntu@192.168.64.13:/tmp/reporte-final.html ~/Desktop/`. (disable recaps in /config)

❯  ~/universidad/ciberseguridad/actividad3-hardening | main !1 ?2  scp ubuntu@192.168.64.13:/tmp/reporte-final.html ~/Desktop/                                                       1 err |
  6s
  Ubuntu 24.04.4 LTS
  ubuntu@192.168.64.13's password:
  scp: remote open "/tmp/reporte-final.html": Permission denied
   ~/universidad/ciberseguridad/actividad3-hardening | main !1 ?2

⏺ El archivo fue creado por root (con sudo). Primero cópialo a tu home en la VM:

  # En la VM
  sudo cp /tmp/reporte-final.html ~/reporte-final.html
  sudo cp /tmp/resultado-final.xml ~/resultado-final.xml
  sudo chown ubuntu:ubuntu ~/reporte-final.html ~/resultado-final.xml

  Luego desde el Mac:

  scp ubuntu@192.168.64.13:~/reporte-final.html ~/Desktop/
  scp ubuntu@192.168.64.13:~/resultado-final.xml ~/Desktop/