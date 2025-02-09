Linux
=====
Ubuntu/Debian basic apt commands:
```bash
# Install manually from *.deb package
sudo dpkg -i teamviewer_15.49.2_amd64.deb 

# Check for updates and for upgradable packages in one shot
sudo apt update && apt list --upgradable

# Upgrade those upgradables
sudo apt upgrade

# List installed package/s
sudo apt list [packagename]

# Uninstall a program
sudo apt remove [packagename]
# Uninstall and also deletes any configuration files related to the packages
sudo apt purge [packagename]

# In case packages chain gets broken/unstable just run
sudo apt --fix-broken install
```


# Desktop
  Abrir explorador de archivos en carpeta actual:
```bash
  xdg-open .
```

  Hacer logout de usuario actual para volver a autenticarte gráficamente:
```bash
  gnome-session-quit
```
# Teclado
  Codigos de tecla y funcion asignada:
```bash
  xev
```
  Por ejemplo, si pulso el SHIFT derecho muestra:
```bash
state 0x11, keycode 62 (keysym 0xffe2, Shift_R), same_screen YES,
```
  Entonces puedo reasignar SHIFT derecho a Intro haciendo:
```bash
  xmodmap -e "keycode 62 = Return"
```
# Procesos
  Diferentes maneras de mostrar los programas que corren en la máquina:
```bash
  # Simple: 
  ps ex

  # Arbol entero: 
  ps axjf

  # Filtrando para mostrar solo procesos que contengan "ssh" en su nombre 
  # excepto este propio proceso de búsqueda
  ps axjf | grep [s]sh

  # Listar procesos por % cpu, eliminando los que usan 0.0 de CPU:
  ps -e -o pcpu,cpu,nice,state,cputime,args --sort pcpu | sed '/^ 0.0 /d'

  # Listar por uso de memoria:
  ps -e -orss=,args= | sort -b -k1,1n | pr -TW$COLUMNS

  # Matar todos los procesos con un cierto patron en su comando de invocacion
  for pid in $(ps -ef | awk '/[s]ubscribers_pin/ {print $2}'); do kill -9 $pid; done

  # Otra manera de matar todas las instancias de un programa o proceso en ejecución
  killall ssh
```
  Disowning background job, which allows a long running process to continue once your shell is closed:
```bash
  $ gzip extremelylargefile.txt &
  $ bg
  $ disown %1
```
# Ficheros
  Listar ordenando por fecha:
```bash
  ls -lrt
```
  Crear link simbólico a un ejecutable para que quede incluido en el PATH:
```bash
  ln -s ~/dsbulk-1.4.1/bin/dsbulk /home/miusuario/bin/dsbulk
```
  Mostrar permisos de un fichero como numero octal:
```bash
  stat -c '%A %a %n' /path/to/file
```
  Asignar recursivamente a un usuario como dueño de carpeta y todas sus subcarpetas y ficheros contenidos:
```bash
  chown -R USERNAME:GROUPNAME /PATH/TO/FILE
```
  Comparar la versión ordenada de dos ficheros de texto en un solo comando:
```bash
  bash -c 'diff <(sort file1.txt) <(sort file2.txt)'
```
  **grep**
```bash
  # Listar los que contengan cierta expr.: 
  grep -rnw '/path/to/somewhere/' -e \"pattern\"

  # Devolver las lineas en que se encuentre alguna de las contenidas en otro fichero y guardarlas en otro fichero aparte
  grep -h -F -f strs_a_buscar.txt original.json > subset.json
```
  Buscar y remplazar en varios ficheros de una tacada:
```bash
  # Con perl (aprovechando sus expresiones regulares más flexibles y potentes)
  perl -i -p -e 's/old/new/g;' *.k??
```
  **rg** [ripgrep]([https://github.com/BurntSushi/ripgrep/blob/master/GUIDE.md]) (fastest grep alternative):
```bash
  # Buscar solo palabra completa en ficheros de codigo fuente C
  rg -w -g '*.{c,h}' void
  # Solamente contar las lineas que se han encontrado
  ps ax | rg [d]sbulk} -c
```

  **ag** [the silver searcher](https://github.com/ggreer/the_silver_searcher) (grep alternative):
```bash
  # Search for "(brand" in all *.ktr files recursively
  ag -G '\.ktr$' '\(brand'
```

  **find**
```bash
  # Extensiones diferentes (ordenadas alfabéticamente) dentro de árbol de subcarpetas
  # saltándonos las de tests
  find . -type f | grep -v 'test' | perl -ne 'print $1 if m/\\.([^.\\/]+)$/' | sort -u
  # Lo mismo pero usando AWK en vez de perl
  find . -type f | grep -v 'test' | awk -F"." '{print $NF}' | sort -u

  # Buscar todos los ficheros de cierto nombre que se han modificado en los últimos 7 días
  # saltándonos las carpetas de test
  find . -name choices.py -mtime -7 -exec ls -lah {} + | grep -v test

  # Buscar todos los ficheros modificados después de una cierta fecha, p.ej.: 2021-01-01
  # evitando algunos por subcadena (p.ej: "karaf") que no interesan
  touch myts -d 2021-01-01
  find data-integration/ -newer myts | grep -v karaf

  # Borrar ficheros de logs de más de 5 dias de antiguedad
  find . -name '*log' -mtime +5 -delete

  # Recorrer un arbol de directorios moviendo todos los ficheros *.JS juntos a una nueva carpeta
  find /migration/Qvantel/JSON -name '*JS' -exec mv {} ~/one/ \;

  # Buscar recursivamente los ficheros *.csv más grandes que cierto tamaño y comprimirlos al máximo
  find . -name '*.csv' -xdev -type f -size +100000 -exec gzip -v9 {} \;
```

  Mostrar CSV en formato tabular/columnar:
```bash
  column -t -s , SoT.csv | less
  column -t -s \| decrypins.csv | less   # Si el separador es un caracter de pipe
```
  Máxima longitud alcanzada por los diferentes valores del campo 99esimo de un CSV que usa "|" como separador:
```bash
  cut -d '|' -f99 customers.csv | wc -L
```
  Contar el número de columnas/campos en un fichero CSV comprimido
```bash
  zcat myfile.csv.gz | head -n 1 | sed 's/[^~]//g' | wc -c
```
  **awk**
```bash
  # Contar el numero de veces que aparece cada valor distinto del campo n-esimo de un CSV
  #    que usa ~ como separador en lugar de coma
  awk -v FS='~'  '{print $2}' file.csv | sort | uniq -c

  # Imprimir todas las columnas excepto la primera, ignorando la primera fila de cabecera
  awk 'BEGIN{FS="~"} NR>1{for (i=2; i<=NF; i++) printf $i " "; print $NF}' file.csv 
```
  **tar**
```bash
  # Create an archive using maximum compression
  env GZIP=-9 tar -czvf my_archive.tar.gz files*.csv others*.txt

  # List contents of a compressed archive
  tar -tvf my_archive.tar.gz

  # Extract files from compressed archive
  tar -xvf archive.tar.gz
```

  SSH / SCP / RSYNC
```bash
  # Enviar un fichero por SCP comprimiéndolo sobre la marcha
  cat /tmp/fichero.txt | gzip | ssh centos@10.40.30.55 'cat > /data/fichero.txt.gz'

  # Borrar fichero remoto via SSH
  ssh usuario@10.40.30.55 rm "$remote_folder"/"$file_name"

  # Using a specific *pem file, a so no need to use a password
  ssh -i svc_lylemig.pem centos@18.193.176.205

  # *.pem file to public SSH keys so it's used transparently
  $ cd ~/.ssh
  $ openssl pkey < ~/.ssh/svc_lylemig.pem > keyfile.pkcs8
  $ mv keyfile.pkcs8 id_rsa
  $ chmod 400 id_rsa
  $ ssh-keygen -y -f id_rsa > id_rsa.pub

  # Upload only newer RBS ktrs and jobs to PRO
  rsync -avzhie "ssh" ~/projects/migration/ktrs/*rbs*.ktr pro-wapp4:/apps/migration/ktrs --update && rsync -avzhie "ssh" ~/projects/migration/jobs/*rbs*.kjb pro-wapp4:/apps/migration/jobs --update

  # Upload only newer ktrs and jobs to CM PRO
  rsync -avzhie "ssh" ~/projects/migration/ktrs/*.ktr pro-wapp15:/apps/migration/ktrs --update && rsync -avzhie "ssh" ~/projects/migration/jobs/*.kjb pro-wapp15:/apps/migration/jobs --update

  # Upload all to PRO RBS
  scp ~/projects/migration/ktrs/*rbs*.ktr pro-wapp4:/apps/migration/ktrs && scp ~/projects/migration/jobs/*rbs*.kjb pro-wapp4:/apps/migration/jobs

  # Upload with SCP 
  scp migration.tar mavega@10.96.3.35:migration.tar

  # Download DR3 ktrs from Georgia Pro Web Apps after a remote:
  # tar -czvf ~/dr3.tar.gz dry_run_3_ktrs kpi_queries rbs_fixes/
  scp geo-wappro:dr3.tar.gz ~/Descargas/dr3.tar.gz

  # Upload only newer RBS ktrs&jobs to Mig.Test WebAPP 
  rsync -avzhie "ssh" ~/projects/migration/ktrs/*rbs* dmt-wapp:/apps/migration/ktrs --update && rsync -avzhie "ssh" ~/projects/migration/jobs/*rbs* dmt-wapp:/apps/migration/jobs --update

  # Downloading only newer ktrs/jobs from Mig.Test WebDB
  rsync -avzhie "ssh" dmt-wdb:/apps/migration/ktrs/*.ktr ~/projects/migration/ktrs --update && rsync -avzhie "ssh" dmt-wdb:/apps/migration/jobs/*.kjb ~/projects/migration/jobs --update

  # Upload only newer ktrs and jobs to Mig.Test WebDB
  rsync -avzhie "ssh" ~/projects/migration/ktrs/*.ktr dmt-wdb:/apps/migration/ktrs --update && rsync -avzhie "ssh" ~/projects/migration/jobs/*.kjb dmt-wdb:/apps/migration/jobs --update

  # Just compare the contents of local and remote folders by checksum, ignoring times
  rsync -arvnc --delete ~/projects/migration/ktrs/ dmt-wapp:/apps/migration/ktrs/
  rsync -arvnc --delete ~/projects/migration/ktrs/ pro-wdb4:/apps/migration/ktrs/
```
  **Espacios en carpetas y dispositivos**
```bash
  # Espacio usado por cada subcarpeta que cuelga de la actual ordenado por tamaño
  du -hd 1 . | sort -hr

  # Espacio libre disponible en carpeta actual
  df -Ph . | tail -1 | awk '{print $4}'

  # Información de todos el sistema de ficheros
  df -TH

  # Listar todos los dispositivos de almacenamiento con sus tamaños y puntos de montaje
  lsblk

  # Alternativamente se podria consultar algo parecido haciendo:
  df -aTh
```
# CPU
```bash
  lscpu   # Detallada
  nproc   # Nro de cores
```
# Memoria instalada y usada
```bash
  free -h
```
# Estadísticas de uso
```bash
  # Resumen rápido del estado de carga actual del sistema
  iostat -mxz 15
```
# Información del Sistema / Equipo
```bash
  sudo dmidecode | grep "System Information" -A8
```
# Consultar distribución & versión
```bash
  cat /etc/*-release
  # or
  cat /etc/issue

  # Versión de kernel
  uname -r
```
# Network / internet
```bash
  # Direcciones locales IPv4/6 
  ip addr

  # Ruta de red hasta cierto host desde la maquina actual
  ip route get 172.31.89.82

  # See network connection password
  sudo cat /etc/NetworkManager/system-connections/DJEZZY-VIP

  # Automatically resume broken downloads
  wget -c --retry-connrefused --tries=0 --timeout=5 http://www.slax.org/file.zip

  # Mount shared Samba folder (i.e. at Windows machine)
  # sudo apt -y install smbclient cifs-utils gvfs-backends
  # sudo mkdir /mnt/casa
  sudo mount -t cifs -o user=marcos  //192.168.1.24/compartida /mnt/casa
  # or
  gio mount smb://192.168.1.124/Compartida

  # Procesos a la escucha:
  ss -plat
  # o
  netstat -lntp
  # o bien
  lsof -iTCP -sTCP:LISTEN -P -n

  # Puertos concretos en que se esta escuchando
  sudo ss -tulpn | grep LISTEN

  # Comprobar si una IP y puerto son accesibles
  nmap -Pn 172.31.89.82 -p 1521
```
# Fecha y hora
```bash
  # Fecha y hora actual en formato ISO 8601 UTC
  date -u +"%Y-%m-%dT%H:%M:%SZ"
```
# Historial de comandos bash
```bash
  # Ejecutar directamente el comando nº 636 de la lista que muestra history
  !636

  # Hace que los comandos que empiecen por espacio no se almacenen en el histórico
  HISTCONTROL=ignoreboth

  # Borrar todo el historial (p.ej.: para que nadie espíe passwords en linea)
  history -c && history -w
```