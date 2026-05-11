# Copilot Instructions

Repositorio académico — Universidad Tecnológica de Pereira, Especialización en TIC, curso de **Ciberseguridad**. Cada actividad es un informe LaTeX que documenta ataques ejecutados desde Kali Linux contra máquinas virtuales víctima en una red host-only (UTM, Mac Mini M4).

## Entorno de virtualización

| Rol | SO | IP | Hipervisor |
|---|---|---|---|
| Atacante | Kali Linux | 192.168.128.3 | UTM, red Solo Anfitrión |
| Víctima (actividad 1) | Metasploitable2 | 192.168.128.2 | UTM, red Solo Anfitrión |
| Víctima (actividad 2) | OWASP Juice Shop | — | UTM, red Solo Anfitrión |

## Compilar el PDF

```bash
# Desde la raíz de cada actividad:
cd actividad1-metasploit2/src && pdflatex main.tex && pdflatex main.tex

# actividad2 usa un path diferente:
cd actividad2-juice-shop/docs/latex && pdflatex actividad2-juice-shop.tex && pdflatex actividad2-juice-shop.tex
```

Se necesitan dos pasadas de `pdflatex` para que la tabla de contenidos quede correcta.

## Estructura de cada actividad

```
actividadN-<nombre>/
├── docs/           # Enunciado y documentación de referencia
├── img/
│   ├── ambiente/   # Capturas del entorno (VMs, red)
│   └── tareaN/     # Capturas por tarea — nombradas t<N>_<descripcion>.png
├── shellText/ (o shelltext/)
│   └── kali/tareaN/  # Texto de terminales copiado de Kali (.txt)
└── src/ (o docs/latex/)
    └── main.tex (o actividad2-juice-shop.tex)
```

## Convenciones del documento LaTeX

El documento sigue el estilo APA en español: márgenes 2.54 cm, interlineado doble (`\doublespacing`), fuente Times 12 pt.

**Secciones:** `\section{}` sin numeración automática, centradas, negrita — una por tarea.

**Subsecciones fijas por tarea:**
1. Descripción de la vulnerabilidad
2. Herramientas utilizadas
3. Pasos realizados
4. Funcionamiento del ataque
5. Resultado

**Bloques de código (shell):**
```latex
\begin{lstlisting}[language=bash, caption={Descripción breve}]
...
\end{lstlisting}
```

**Figuras:**
```latex
\begin{figure}[H]
  \centering
  \includegraphics[width=0.92\textwidth]{../img/tareaN/tN_descripcion.png}
  \caption{\textit{Descripción de la figura.}}
\end{figure}
```

Las rutas de imágenes son relativas desde `src/` (o `docs/latex/`) hacia `../img/`.

## Convenciones de nombres de archivos

- Imágenes: `t<N>_<descripcion_corta>.png` (ej. `t1_nmap_scan.png`)
- Texto de shell: nombre descriptivo del comando (ej. `hydra.txt`, `cat_etc_passwd.txt`)
- No incluir espacios; usar guion bajo como separador.
