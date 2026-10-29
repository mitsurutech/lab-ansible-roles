# Laboratorio de roles de Ansible para el aula: un rol, handlers y Vault en una VM

[English](README.md) | [Français](README.fr.md) | [日本語](README.ja.md) | [Español](README.es.md) | [Deutsch](README.de.md)

[![check](https://github.com/mitsurutech/lab-ansible-roles/actions/workflows/check.yml/badge.svg)](https://github.com/mitsurutech/lab-ansible-roles/actions/workflows/check.yml)

Un **laboratorio de roles de Ansible** listo para usar para enseñar infraestructura como código, de la [plataforma de aprendizaje MitsuruTech](https://mitsurutech.co.uk/es/learning), donde los centros educativos dan a cada estudiante sus propias máquinas y su terminal en el navegador. Es el segundo nivel después de [lab-ansible-basics](https://github.com/mitsurutech/lab-ansible-basics). Cada estudiante recibe una **VM** de Ubuntu con **ansible-core** instalado y un pequeño proyecto, y construye un **rol** desde cero: sus variables, una plantilla Jinja2, tareas con etiquetas (tags), un **handler** que se ejecuta una sola vez y un token guardado en **Ansible Vault**. La propia máquina es el destino, así que no hace falta nada más y no se instala nada en el ordenador del estudiante.

## Qué hace el estudiante

El laboratorio deja un proyecto en `~/roles-lab` y un `README.md` en la carpeta personal que explica qué hacer en siete pasos sin dar ninguna respuesta:

```
ansible.cfg          apunta a inventory.ini y a la carpeta de roles
inventory.ini        dos hosts, web1 y spare1, ambos esta máquina
site.yml             un play sobre web que usa el rol appconf
vault-password.txt   la contraseña de Vault de este laboratorio
roles/               donde va el rol
```

1. Ejecutar el playbook y ver cómo se detiene, porque el rol `appconf` todavía no existe.
2. Crear el rol con `ansible-galaxy role init` y rellenar su `meta/main.yml`.
3. Definir variables en `defaults/main.yml` y `vars/main.yml`. `site.yml` ya fija `app_env` y `app_dir` en el play, así que el estudiante tiene que descubrir cuál gana.
4. Escribir las tareas, con la etiqueta `config` o `user`: una carpeta, un archivo creado a partir de una plantilla con un bucle y una condición, un archivo por cada función, un archivo con el token, un usuario del sistema y dos archivos pequeños, uno de ellos solo si el usuario se creó de verdad.
5. Conseguir que un handler notificado por tres tareas se ejecute una sola vez, y que el rol ejecute sus handlers a mitad de camino.
6. Guardar `app_api_token` en un archivo creado con `ansible-vault`, abierto con el archivo de contraseña y escrito en texto plano en ningún otro sitio.
7. Ejecutarlo dos veces (la segunda no debe cambiar nada) y probar después `--check --diff`, `--tags` y `--limit`.

## Cómo se califica

El laboratorio tiene una **comprobación**, un script que califica una VM entregada sin que el profesor tenga que leerla. Se ejecuta sobre una copia de la máquina guardada, tras un arranque limpio. Primero lee el rol: las carpetas estándar, un `meta/main.yml` rellenado, al menos seis tareas en el rol y no en el playbook, un handler, un bucle, etiquetas y una plantilla con un bucle `for` y un `if`. Rechaza `shell`, `command`, `raw` y `script` (y alguna otra forma de saltarse los módulos de Ansible), y un lookup `pipe` que ejecuta un programa. Abre el archivo de la caja fuerte con la contraseña del laboratorio y rechaza un token escrito en texto plano en el proyecto.

Después **ejecuta el `site.yml` del estudiante siete veces con valores que nadie ha visto**: en una máquina limpia, otra vez sin nada que cambiar, como simulación (dry run) que debe mostrar el cambio sin escribir nada, con otros valores, con cada etiqueta y con `--limit` sobre el host que el play no usa. Cuenta las ejecuciones del handler en la salida real, así que un handler que se ejecuta tres veces se rechaza, y lee los archivos resultantes. La primera línea que imprime dice qué no está bien.

Lo que no hace: nunca reinicia un servicio real, porque una comprobación no puede demostrarlo, y un estudiante con root en su propia VM podría aun así manipular el propio Ansible. Es una comprobación para el aula, no una prueba.

## Qué hay en el archivo

`ansible-roles.lab.yaml` es una **definición de laboratorio de VM** completa, exactamente lo que ejecuta la plataforma:

| Clave | Qué es |
| --- | --- |
| `cpus`, `memory`, `disk` | el tamaño de la VM de cada estudiante: 1 CPU, 1024 MB y 10 GB |
| `open` | los puertos que se abren en el navegador: ninguno, el laboratorio es el terminal |
| `setup` | un script que se ejecuta una vez, como root, la primera vez que arranca la VM de cada estudiante: instala `ansible-core` desde el archivo de Ubuntu y escribe el proyecto, el playbook, una contraseña de Vault falsa y el README |
| `check` | el script de calificación, ejecutado como root sobre una copia del disco guardado tras un arranque limpio; su código de salida es el resultado y `0` significa aprobado |

La contraseña de Vault del proyecto es un **dato de prueba**: dice ella misma que no es un secreto, y el token que el estudiante cifra es un dato de demostración, nunca un secreto real.

## Usarlo en la plataforma

**Para ponerlo como tarea a una clase:**

1. En su módulo, añada una tarea y elija la tarjeta **Laboratorio de VM Ubuntu**.
2. Pulse el ejemplo de roles de Ansible, o pegue el contenido de `ansible-roles.lab.yaml`. Usar un ejemplo sustituye el texto, y Deshacer recupera el suyo.
3. Publique y pruebe primero su propia copia. Cada estudiante tiene su propia VM, que se conserva entre sesiones.
4. Asígnelo como cualquier tarea. Cuando se entregue, ejecute la comprobación en la fila de un estudiante en la lista de la clase: arranca una copia de su máquina guardada, que no se guarda de vuelta, y muestra si aprobó y lo que imprimió.

Una lección de Basics de la plataforma se trabaja en este laboratorio. La [página de este laboratorio](https://mitsurutech.co.uk/es/help/lab-ansible-roles) en el manual, su [guía de los laboratorios de VM Ubuntu](https://mitsurutech.co.uk/es/help/labs) y su [página de laboratorios de ejemplo](https://mitsurutech.co.uk/es/help/sample-labs) cuentan el resto.

¿Aún no tiene cuenta de centro? [El primer año es gratis](https://mitsurutech.co.uk/es/pricing) hasta 50 estudiantes, y [registrarse](https://mitsurutech.co.uk/es/signup) solo pide una dirección de correo.

## Comprobar el archivo

En cada push, `.github/workflows/check.yml` comprueba que el archivo es YAML válido con las claves de arriba y que `setup` y `check` caben cada uno en el límite de 16 KB de la plataforma. No arranca ninguna VM: eso se hace en las máquinas de la propia plataforma.

## Lo que no funciona en un laboratorio

La VM llega al archivo de paquetes de Ubuntu, PyPI, npm y GitHub, y a nada más. El laboratorio solo usa el archivo de Ubuntu, así que cada trabajo se hace con los módulos propios de Ansible sobre archivos, carpetas y un usuario del sistema. No hay ningún servicio que reiniciar: el handler se muestra por lo que una comprobación puede ver, las veces que se ejecutó y un archivo que escribe. El destino es la propia VM, por conexión local.

## Acerca de

Creado por [Mitsuru Technologies](https://mitsurutech.co.uk/es) para la [plataforma de aprendizaje MitsuruTech](https://mitsurutech.co.uk/es/learning): Python, Java, Go, Node, SQL y Linux en el navegador para escuelas y centros educativos, con laboratorios de contenedores, laboratorios de VM, aplicaciones web desplegadas desde GitHub y cuestionarios que se corrigen solos. Pruebe código sin cuenta en el [playground](https://mitsurutech.co.uk/es/playground).

Más laboratorios del mismo sitio: [lab-netlab](https://github.com/mitsurutech/lab-netlab), [lab-juice-shop](https://github.com/mitsurutech/lab-juice-shop), [lab-dvwa](https://github.com/mitsurutech/lab-dvwa), [lab-webgoat](https://github.com/mitsurutech/lab-webgoat), [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin), [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea), [lab-sample](https://github.com/mitsurutech/lab-sample), [lab-sdn-lab](https://github.com/mitsurutech/lab-sdn-lab), [lab-reverse-lab](https://github.com/mitsurutech/lab-reverse-lab), [lab-formal-lab](https://github.com/mitsurutech/lab-formal-lab), [lab-hdl-lab](https://github.com/mitsurutech/lab-hdl-lab), [lab-k3s-lab](https://github.com/mitsurutech/lab-k3s-lab), [lab-tofu-lab](https://github.com/mitsurutech/lab-tofu-lab) y [lab-ansible-basics](https://github.com/mitsurutech/lab-ansible-basics).

## Licencia

Esta obra está bajo la licencia [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.es). Puede compartirla y adaptarla para aprender y enseñar, si cita a Mitsuru Technologies Ltd, no la usa comercialmente y comparte lo que cree a partir de ella bajo la misma licencia. El texto completo está en el archivo `LICENSE`.

[Ansible](https://www.ansible.com) (`ansible-core`, GPL-3.0) se instala desde el archivo de Ubuntu cuando arranca una VM; aquí no se distribuye nada de él.
