# Evidencia de práctica de Linux

## Registro de comandos y resultados

````text
Script started on 2026-11-13 04:05:09+00:00 [TERM="xterm-256color" TTY="/dev/pts/0" COLUMNS="263" LINES="58"]
JorgeGus@UbuntuVM:~$ whoami

hostname

pwd

uname -a
JorgeGus@UbuntuVM:~$ whoami

hostname

pwd

uname -a

JorgeGus

UbuntuVM

/home/JorgeGus

Linux UbuntuVM 7.0.0-38-generic #38-Ubuntu SMP PREEMPT_DYNAMIC Fri Sep  4 09:10:14 UTC 2026 x86_64 GNU/Linux
JorgeGus@UbuntuVM:~$ mkdir -p ~/prueba_calificada

cd ~/prueba_calificada

pwd

echo "Primera evidencia" > documento1.txt

echo "Segunda evidencia" > documento2.txt

ls -l
JorgeGus@UbuntuVM:~$ mkdir -p ~/prueba_calificada

cd ~/prueba_calificada

pwd

echo "Primera evidencia" > documento1.txt

echo "Segunda evidencia" > documento2.txt

ls -l

/home/JorgeGus/prueba_calificada

total 8
-rw-rw-r-- 1 JorgeGus JorgeGus 18 Nov 13 04:05 documento1.txt
-rw-rw-r-- 1 JorgeGus JorgeGus 18 Nov 13 04:05 documento2.txt
JorgeGus@UbuntuVM:~/prueba_calificada$ cat documento1.txt

echo "Linea adicional" >> documento1.txt

cat documento1.txt
JorgeGus@UbuntuVM:~/prueba_calificada$ cat documento1.txt

echo "Linea adicional" >> documento1.txt

cat documento1.txt

Primera evidencia

Primera evidencia
Linea adicional
JorgeGus@UbuntuVM:~/prueba_calificada$ chmod 640 documento1.txt

ls -l documento1.txt
JorgeGus@UbuntuVM:~/prueba_calificada$ chmod 640 documento1.txt

ls -l documento1.txt

-rw-r----- 1 JorgeGus JorgeGus 34 Nov 13 04:05 documento1.txt
JorgeGus@UbuntuVM:~/prueba_calificada$ for i in 1 2 3; do

    echo "Prueba numero $i"

done
JorgeGus@UbuntuVM:~/prueba_calificada$ for i in 1 2 3; do

    echo "Prueba numero $i"

done

Prueba numero 1
Prueba numero 2
Prueba numero 3
JorgeGus@UbuntuVM:~/prueba_calificada$ ls -lh /media/sf_UbuntuCompartido/ls -lh /media/sf_UbuntuCompartido/

ls: cannot access '/media/sf_UbuntuCompartido/': No such file or directory
JorgeGus@UbuntuVM:~/prueba_calificada$ ls -lh /media/sf_UbuntuCompartido////////////////B/U/N/T/U/H/A///S/H/A/E//R/E/D/

total 4.5K
-rwxrwx--- 1 root vboxsf 867 Nov 12 21:27 LAB10-5.md
-rwxrwx--- 1 root vboxsf  29 Nov 13 04:04 prueba_conexion.txt
-rwxrwx--- 1 root vboxsf   0 Nov 13 04:05 prueba_linux.log
JorgeGus@UbuntuVM:~/prueba_calificada$ EXITexit

exit

Script done on 2026-11-13 04:06:34+00:00 [COMMAND_EXIT_CODE="0"]
````
