# Backup de QlikSense

Es importante realizar correctamente los backups de QlikSense con el objetivo de restaurar ante un inconveniente no sólo las aplicaciones si no también la configuración del propio servidor, en términos de seguridad, tareas, streams, y demás. No alcanza con solamente resguardar los archivos .qvf (aplicaciones).

Para automatizar el backup de QlikSense, en TiNB creamos una pequeña utilidad que automatiza este proceso, y entregamos a nuestros clientes como parte de nuestro valor agregado.

## Requerimientos
* http://www.7-zip.org/  (herramienta gratis para comprimir los backup)
* Acceso al servidor QlikSense
La implementación está basada en la documentación oficial de Qlik Sense: https://goo.gl/f5pNFM automatizando los puntos de backup excepto el punto 3 (Make a backup of the certificates used to secure the Qlik Sense services. This only needs to be done once.) que debe ser realizado manualmente una única vez. Este procedimiento está detallado en https://goo.gl/jFhNaU

## Implementación
Crear un archivo bat llamado backup.bat

```
@echo off
cls

rem Utilidad para comprimir el backup.-
set EXE="C:\Program Files\7-Zip\7z.exe"


rem Nombre del archivo de backup
rem Nota: el formato de timestamp puede diferir dependiendo la configuración regional.
set BACKUP_FILE=backup%DATE:~10,4%%DATE:~4,2%%DATE:~7,2%.zip
echo %BACKUP_FILE%

rem Ir al directorio de backup
cd \backup
rmdir /s /q work
mkdir work
mkdir work\Log
mkdir work\Apps
mkdir work\Files

"C:\Program Files\Qlik\Sense\Repository\PostgreSQL\9.3\bin\pg_dump.exe" -h localhost -p 4432 -U postgres -b -F t -f "work\QSR_backup.tar" QSR
xcopy /d /s /y "%ProgramData%\Qlik\Sense\Apps" "work\Apps"
xcopy /d /s /y "C:\development\edelap" "work\Files"
xcopy /d /s /y "certificates" "work\"

cd work
%EXE% a -mx9 -tzip ..\%BACKUP_FILE% *
cd ..

:fin

echo Listo.
```

Para finalizar, esta utilidad puede ser programada dentro del administrador de tareas según las necesidades de cada instalación, creando así copias de seguridad programadas, y puede ser complementada con un mecanismo de copia de los archivos a otra unidad (externa o remota) para una mayor tolerancia a fallos.