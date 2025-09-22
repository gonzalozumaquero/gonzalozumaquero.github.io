# Gonzalo Zumaquero — Ciberseguridad

Bienvenido a mi espacio técnico. Aquí documento mi trabajo en seguridad ofensiva: writeups de máquinas, diario de bug bounty y proyectos (incluido AutoVPN de mi TFM). El objetivo es consolidar conocimiento, aportar valor práctico y mantener un portafolio público y verificable.

---

## Sobre mí

- Ingeniería con foco en programación, automatización y sistemas Linux.
- Pentesting y red team: enumeración, explotación, privesc, pivoting y AD básico.
- Certificación: eJPTv2 (Junior Penetration Tester).
- En preparación: OSCP.

---

## Navegación rápida

- **Writeups**: documentación paso a paso de máquinas resueltas (orientado a OSCP).
- **Bug Bounty Diary**: metodología, patrones y aprendizajes en programas públicos.
- **Proyectos**: utilidades, scripts y plataforma AutoVPN (TFM).

---

## Writeups

> Solo se publican writeups de máquinas retiradas o de laboratorios públicos/propios. Nada de contenido de máquinas activas.

- **NIBBLES** — Linux. Enumeración web, explotación inicial y escalada de privilegios.  
  [Abrir NIBBLES](NIBBLES.md)

- **ALGERNON** — Windows. Exposición de servicios internos, credenciales y privesc con abusos de servicio.  
  [Abrir ALGERNON](ALGERNON.md)

- **INTERNAL** — Mixto. Enumeración SMB/HTTP, movimiento lateral y pivoting en red simulada.  
  [Abrir INTERNAL](INTERNAL.md)

Próximas incorporaciones: nuevas máquinas con foco en Active Directory, file upload seguro y kerberoasting.

---

## Bug Bounty Diary

Bitácora práctica centrada en metodología reproducible: scoping, reconocimiento, pruebas de IDOR, XSS, SSRF y bypasses comunes. Incluye payloads genéricos, wordlists de parámetros y notas de rate limiting.  
Estado: preparando estructura inicial y primera entrada.

- Próximas entradas:  
  - Cartografía de endpoints y parámetros en APIs.  
  - Patrones de redirecciones y validación de URLs.  
  - Listas de comprobación previas al envío de reporte.

---

## Proyectos

- **AutoVPN (TFM)** — Plataforma de automatización de VPNs personales (WireGuard/OpenVPN), con énfasis en seguridad, despliegue con contenedores y UX. Documentación, arquitectura y roadmap públicos.  
  Repositorio: próximamente desde este sitio.

- **Pentesting Scripts** — Utilidades para acelerar recon y post-explotación (wrappers de nmap, extracción de puertos, helpers SMB/LDAP).  
  Publicación progresiva con ejemplos de uso y licencias abiertas.

---

## Metodología y estándares editoriales

- Estructura uniforme: Reconocimiento → Explotación → Escalada → Evidencias → Lecciones aprendidas.  
- Reproducibilidad: comandos anotados, versiones de herramientas y consideraciones de entorno.  
- Ética: cumplimiento de normas de plataformas y responsible disclosure.

---

## Últimas actualizaciones

- Alta de sitio con MkDocs Material y navegación inicial.  
- Publicados primeros tres writeups en formato Markdown.  
- Preparación de secciones para Bug Bounty y Proyectos.

---

## Contacto

- GitHub: https://github.com/gonzalozumaquero  
- LinkedIn: https://www.linkedin.com/in/gonzalozumaquero

¿Sugerencias o feedback? Se agradecen issues o comentarios en los repositorios vinculados.
