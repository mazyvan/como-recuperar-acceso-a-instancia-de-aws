# Recuperación de acceso a una Instancia de AWS

Lista de pasos a seguir si pertiste acceso a una instancia.  
Este documento también puede ser usado como referencia para generar más claves a un servidor al que si se tengas acceso, consulta únicamente los puntos `7` y `8`. 

**Importante**: Si se usará la dirección IP para establecer la conexión por ssh, es necesario asignar una `Elastic IP` o solo estar consiente de que la direccion IP asignada cambia cada que la instancia se reinicia.

## Lista de pasos

1. Crear respaldo del servidor (Instancia y Volumen) por si todo sale mal
2. Crear una nueva instancia (**en la misma region y la misma subred <subnet>**) y conectarte a ella
3. Detener la instancia a la que se perdió el acceso
4. Desasociar el volumen (Disco duro) de la instancia a la que se perdió el acceso
5. Detener la nueva instancia
6. [Asociar el volumen a la nueva instancia como disco externo](#comandos-para-montar-el-disco)
7. [Generar o copiar las nuevas claves de acceso en el volumen asociado](#notas-sobre-como-generar-el-conjunto-de-claves)

8. [Agregar la clave pública a la lista en el servidor](#Notas-sobre-como-agregar-la-clave-pública-al-servidor)
8. Desasociar el volumen de la nueva instancia
9. Asociar el volumen a como **root disk** (`/dev/sda1`)
10. Reiniciar la instancia
11. Generar un nuevo backup si todo esta funcionando bien de nuevo
12. Tener más cuidado la próxima vez

## Referencias y notas

### Comandos para montar el disco

Creamos un directorio en el nuevo servidor

```sh
sudo mkdir /media/hd_volumen_a_recuperar
```

Lista los volumenes para saber el nombre del que se va a montar (probablemente tenga de nombre `/dev/xvdf1`)

```sh
df -aTh
```

Monta el disco con el siguente comando:

```sh
sudo mount /dev/xvdf1 /media/hd_volumen_a_recuperar
```

### Notas sobre como generar el conjunto de claves

Remplaza `<host>` con la IP estática del servidor

```sh
ssh-keygen -R <host>
```

O si deseas usar la misma clave para todos los servidores unicamente 
modifica tu archivo de configuración de ssh de tu computadora `~/.ssh/config`

Agrega o modifica estas lineas
```
Host *
  StrictHostKeyChecking no
```

y evita el parametro `-R` al momento de generarla

**Importante**: Ten cuidado al generar la clave, no la guardes en la locación por defecto si ya has generado antes un par de claves para evitar reemplazar alguna de las anteriores (a menos que la vayas a usar como tu clave maestra para el acceso a todos tus hosts)

[Visita esta URL para más información](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1604)

### Notas sobre como agregar la clave pública al servidor

Imprime la clave generada *en donde sea que la hayas guardado* y copiala completa

```sh
cat ~/.ssh/id_rsa.pub
```

Deberas ver algo como esto:

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCqql6MzstZYh1TmWWv11q5O3pISj2ZFl9HgH1JLknLLx44+tXfJ7mIrKNxOOwxIxvcBF8PXSYvobFYEZjGIVCEAjrUzLiIxbyCoxVyle7Q+bqgZ8SeeM8wzytsY+dVGcBxF6N4JS+zVk5eMcV385gG3Y6ON3EG112n6d+SMXY0OEBIcO6x+PnUSGHrSgpBgX7Ks1r7xqFa7heJLLt2wWwkARptX7udSq05paBhcpB0pHtA1Rfz3K2B+ZVIPSDfki9UVKzT8JUmwW6NNzSgxUfQHGwnW7kj4jp4AT0VZk3ADw497M2G/12N0PPB5CnhHf7ovgy6nL1ikrygTKRFmNZISvAcywB9GVqNAVE+ZHDSCuURNsAInVzgYo9xgJDW8wUw2o8U77+xiFxgI5QSZX3Iq7YLMgeksaO4rBJEa54k8m5wEiEE1nUhLuJ0X/vh2xPff6SQ1BL/zkOhvJCACK6Vb15mDOeCSq54Cr7kvS46itMosi/uS66+PujOO+xt/2FWYepz6ZlN70bRly57Q06J+ZJoc9FfBCbCyYH7U/ASsmY095ywPsBo1XQ9PqhnN1/YOorJ068foQDNVpm146mUpILVxmq41Cj55YKHEazXGsdBIbXWhcrRf4G2fJLRcGUr9q8/lERo9oxRm5JFX6TCmj6kmiFqv+Ow9gI0x8GvaQ== demo@test

Accede a tu servidor remoto y deberás asegurarte que el directorio `~/.ssh` existe.   
Este comando lo creará unicamente solo si no existe:

```sh
mkdir -p ~/.ssh
```

Ahora ejecuta este comando para agregar la clave pública que copiaste a la lista de claves autorizadas en el servidor

```sh
echo <public_key_string> >> ~/.ssh/authorized_keys
```

En el comando anterior, substitute `<public_key_string>` con el texto copiado (tu clave pública)
Debe iniciar con: *ssh-rsa AAAA....*   
Por último hay que asegurarnos que el directorio `~/.ssh` y el archivo `authorized_keys` tienen los permisos apropiados

Ejecuta este comando para eso: 

```sh
chmod -R go= ~/.ssh
```

También es importante que el directorio le pertenesca al usuario con el que te vas a logear (ubuntu en la mayoria de los casos) y no a `root`. Con el siguente comando puedes cambiar esto:

```sh
chown -R ubuntu:ubuntu ~/.ssh
```

[Visita esta URL para más información](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1604#copying-public-key-manually)

## Troubleshooting: Common issues

Si ocurren problemas al intentar conectarse al servidor p. ej.   
`WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!`  
checa: 
https://stackoverflow.com/questions/20840012/ssh-remote-host-identification-has-

## Más información
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#replacing-lost-key-pair

## Otras maneras
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#replacing-lost-key-pair


# Contribuir

Si detectas algún error o tienes alguna propuesta, [crea un issue](https://github.com/mazyvan/repo-name/issues/new). Si es algo que tú mismo puedes corregir, te sugerimos que mejor hagas un request siguiendo estos pasos:

1. Haz un [fork](https://help.github.com/articles/fork-a-repo) (https://github.com/mazyvan/repo-name/fork)
2. Crea una [rama](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell) con la característica deseada (`git checkout -b my-new-feature`)
3. Haz [commit](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository#_committing_changes) de tus cambios (`git commit -am 'Add some feature'`)
4. Ahora sube a git con el comando [push](https://git-scm.com/book/en/v2/Git-Basics-Working-with-Remotes#_pushing_remotes) a la rama que creaste (`git push origin my-new-feature`)
5. Por último, crea una [Pull Request](https://help.github.com/articles/about-pull-requests/)