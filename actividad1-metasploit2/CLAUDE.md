# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Actividad #1 - Metasploitable2

## Contexto

Proyecto académico de la Universidad Tecnológica de Pereira — Especialización en Tecnologías de la Información y las Comunicaciones, curso de Ciberseguridad.

**Fecha de entrega:** 10 de abril de 2026

## Entorno

| Máquina | Rol | IP | Plataforma |
|---|---|---|---|
| Metasploitable2 | Víctima (target) | 192.168.128.2 | utm, red Solo Anfitrión |
| Kali Linux | Atacante | 192.168.128.3 | utm, red Solo Anfitrión |

Ambas VMs tienen **solo una tarjeta de red en modo Solo Anfitrión** (host-only). La actividad se realiza en una Mac Mini M4 con UTM como hipervisor.

## Documento LaTeX

El entregable es `src/main.tex`, compilado con `pdflatex`. Las imágenes se referencian con rutas relativas desde `src/` hacia `../img/`.

### Compilar el PDF

```bash
cd src && pdflatex main.tex
```

Para compilación completa con tabla de contenidos:

```bash
cd src && pdflatex main.tex && pdflatex main.tex
```

### Convenciones del documento

- Secciones: `\section{}` (sin numeración, centradas, negrita) — una por tarea
- Subsecciones fijas por tarea: Descripción de la vulnerabilidad, Herramientas utilizadas, Pasos realizados, Funcionamiento del ataque, Resultado
- Bloques de código: `\begin{lstlisting}[language=bash, caption={...}]`
- Figuras: `\begin{figure}[H]` con `\includegraphics[width=0.92\textwidth]{../img/...}` y caption en `\textit{}`
- Formato APA: márgenes 2.54 cm, interlineado doble, fuente Times 12 pt

## Tareas

### Tarea 1 — Explotación de Backdoors ✅
- Vulnerabilidad: CVE-2011-2523 (vsftpd 2.3.4 backdoor, puerto 21 → shell root en puerto 6200)
- Herramientas: Nmap, Metasploit (`exploit/unix/ftp/vsftpd_234_backdoor`)
- Imágenes en: `img/tarea1/`

### Tarea 2 — SQL Injection en DVWA ✅
- Obtener la información de la base de datos de DVWA.
- Herramientas: sqlmap, interfaz web de DVWA.
- Imágenes en: `img/tarea2/` | Shell output en: `shellText/kali/tarea2/`

### Tarea 3 — Command Execution en DVWA ✅
- Obtener `/etc/passwd` y `/etc/shadow` vía Command Execution de DVWA.
- Imágenes en: `img/tarea3/` | Shell output en: `shellText/kali/tarea3/` (archivos `cat_etc_passwd.txt`, `cat_etc_shadow.txt`)

### Tarea 4 — Fuerza Bruta a DVWA ✅
- Herramientas: **Burp Suite** y **Hydra**.
- Imágenes en: `img/tarea4/` | Shell output en: `shellText/kali/tarea4/`

### Tarea 5 — Cracking de contraseñas offline
- Usar `/etc/passwd` y `/etc/shadow` obtenidos en Tarea 3.
- Herramienta: **John the Ripper**.

## Estructura del repositorio

```
actividad1-metasploit2/
├── CLAUDE.md
├── docs/                    # Enunciado y documentación de referencia
├── img/
│   ├── ambiente/            # Capturas del entorno (UTM, Kali, Metasploitable2)
│   ├── tarea1/              # Capturas por tarea (nombradas t<N>_<descripcion>.png)
│   ├── tarea2/
│   ├── tarea3/
│   └── tarea4/
├── shellText/kali/
│   ├── tarea1/              # Texto de terminales copiado de Kali
│   ├── tarea2/
│   ├── tarea3/              # cat_etc_passwd.txt, cat_etc_shadow.txt
│   └── tarea4/              # hydra.txt
└── src/
    ├── main.tex             # Documento principal
    └── .gitignore           # Excluye artefactos de compilación LaTeX
```
