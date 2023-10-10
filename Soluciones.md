# 1. Línea de comandos
## 1.1. Utilizando el manual del sistema
## 1.2. Gestión y manipulación de archivos

1.Crear una carpeta llamada AS en vuestro directorio raíz de usuario.

```bash
# Desde dentro de /home/aingeru
mkdir AS
```

2.Entrar dentro de la carpeta y comprobar que el directorio coincide con el contenido de la variable de entorno PWD.

```bash
cd AS
echo $PWD
```

3.Instalar cal con el comando apt install ncal. Utilizar cal para mostrar un calendario y redirigir la salida a un fichero de texto. Comprobar que ese fichero se crea correctamente y que su contenido es el esperado

```bash
# Instalar cal
sudo apt install ncal
# Rederigir la salida de cal a un fichero de texto
cal > calendario.txt
# Comprobar que el contenido es el esperado
cat calendario.txt
```

4.Copiar el fichero recién creado al directorio raíz del usuario.

```bash
cp calendario.txt /home/aingeru
```

5.Moverse al directorio raíz del usuario y listar en formato extendido (parámetro -l) los directorios y archivos presentes. Redirigir esa información a un fichero.

```bash
cd /home/aingeru
ls -l | fichero.txt
```

6.Listar los 5 ficheros...

```bash
ls -ltp /etc | grep -v / | head -n 5
```
Con la opción `-t` se listan los archivos y directorios por orden de modificación y con la opción `-p` se les añade un '/' al final a los directorios.

Con `grep -v /` quitamos las lineas que tienen un símbolo '/'

Con `head -n 5` nos quedamos con las 5 primeras líneas. 

7.Cambiar los permisos...

```bash
chmod o-rw calendario
chmod g-rw calendario
chmod u+rw calendario
```

8.Cambiar los permisos para evitar...

(El grupo del directorio es mi usuario y en ese grupo solo estoy yo)

9.Usuarios en el sistema y SHELL...

Para saber cuantos usuarios hay en el sistema, vamos a consultar el archivo `/etc/passwd`:

```
wc -l /etc/passwd
```

Para saber cuál es nuestro Shell, consultamos la variable de entorno `SHELL`:

```bash
echo $SHELL
```

10.Comprobar cuando y desde donde...

```bash
who
```

11.Comprimir y descomprimir...

```bash
# Comprimir
tar cfvz compressed.tar.gz $HOME
# Descomprimir (desde dentro del directorio /tmp)
tar xfvz $HOME/compressed.tar.gz
```

12.Buscar los archivos de mi usuario desde el directorio '/'...

```bash
sudo find / -user aingeru -ls
``` 

13.Como usuario “root”, mostrar las últimas 30 líneas de /var/log/syslog.

```bash
sudo cat /var/log/syslog | tail -n 30
```

# 2 Shell Scripting


1.Crear un script llamado lsdirs.sh que muestre los directorios contenidos en el directorio actual.

```bash
#!/bin/bash

for archivoODir in `ls`; do
	# Comprobar si es un directorio
	if [ -d $archivoODir ]; then
		echo $archivoODir
	fi
done
```

2.Crear un script llamado see.sh que reciba un nombre de fichero/directorio como parámetro. Si el nombre
corresponde a un fichero, el script muestra el contenido del fichero con more, sino muestra el contenido del directorio con ls.

```bash
#!/bin/bash

if [ -f $1 ]; then
	more $1
else
	ls $1
fi
```

3.Crear un script que pida al usuario que teclee una palabra y escriba por pantalla el número de caracteres de esa palabra.

```bash
#!/bin/bash
echo "Introduce una palabra: "
read palabra
length=$(echo $palabra | wc -m)
caracteres=$((length-1))
echo $caracteres
```

4.Crear un script que pida al usuario que teclee una palabra y compruebe si es un comando del sistema o no.

```bash
#!/bin/bash
echo "Introduce una palabra para probar si es un comando del sistema: "
read palabra
# Comprobamos que exista un fichero ejecutable para ese comando
execFile=$(which $palabra)

# Si el fichero es ejecutable
if [[ -z $execFile ]]; then
        echo "La palabra no es ejecutable"
else
        echo "La palabra es ejecutable"
fi
```

5.Crear un script que cree una carpeta llamada cosas y después cree 100 ficheros vacíos llamados
`fich<numero>.txt` dentro de ella, donde `<numero>` es un número entre 0 y 99.

```bash
#!/bin/bash

mkdir cosas
cd cosas
for num in {0..99}; do
        touch "fich$num.txt"
done
```

6.Extender el script anterior para que cada fichero contenga la N-sima línea delmanual de ls (man ls). El fichero fich0.txt tendrá la línea 0 del manual, fich1.txt tendrá la línea 1 del manual, …

```bash
#!/bin/bash

mkdir cosas
cd cosas
for num in {0..99}; do
        touch "fich$num.txt"
		linea=$((num+1))
        man ls | sed "${linea}q;d" > "fich$num.txt"
done
```

7.Crear un script que modifique la extensión de todos los ficheros .txt de un directorio a .t.

```bash
#!/bin/bash

for archivo in $(ls *.txt); do
        sin_extension=${archivo%.*}
        mv $archivo "$sin_extension.t"
done
```

8.Crear un script `borra.sh` que reciba un número indefinido de parámetros (de 0 a 9) y borre el fichero correspondiente a la suma del valor de los parámetros que reciba. Por ejemplo, `borra.sh 1 4 5 9`, borraría el
fichero fich19.txt (1 + 4 + 5 + 9 = 19).

```bash
#!/bin/bash

# '$*' contiene todos los parametros recibidos
for parametro in $*; do
        suma=$((suma+parametro))
done

rm "fich$suma.txt"
```

9.Crear un script `orden.sh` que muestre el contenido del fichero `/etc/passwd` ordenado por nombre de usuario, UID o GID. El script recibirá un parámetro que indique cuál de los 3 utilizar como clave de ordenación.

```bash
#!/bin/bash

if [ $1 == 'usuario' ]; then
        sort -t ':' -k 1 /etc/passwd
elif [ $1 == 'UID' ]; then
        sort -t ':' -k 3 -n /etc/passwd
elif [ $1 == 'GID' ]; then
        sort -t ':' -k 4 -n /etc/passwd
else
        echo "Introduce 'usuario', 'UID' o 'GID'"
fi
```

10.Imagina que quieres mandar un e-mail con el mismo cuerpo a varios destinatarios, pero personalizando la
primera línea con el nombre de cada uno. Crear un fichero cuerpo.txt con el texto del cuerpo del e-mail y la palabra NOMBRE en cada lugar donde querrías poner el nombre, y un fichero nombres.txt con varios
nombres personales. Crear un script que genere varios ficheros con el cuerpo del e-mail personalizado para
cada destinatario.

```bash
#!/bin/bash

cuerpo=`cat cuerpo.txt`

for linea in `cat nombres.txt`; do
        touch "mail_$linea.txt"
        cuerpo_nombre=${cuerpo/NOMBRE/$linea}
        echo $cuerpo_nombre > "mail_$linea.txt"
done
```
