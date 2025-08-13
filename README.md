# glpi-lamp

Este rol de Ansible configura un servidor LAMP (Linux, Apache, MySQL, PHP) y despliega la aplicación GLPI (Gestionnaire Libre de Parc Informatique).

Cabe destacar que este es un Rol de recuperación el cual es personalizable, pero con el fin de recuperar nuestro sistema GLPI tras un desastre.

Sus tareas principales son:

1. Instala todas las dependencias necesarias para implementar un servidor LAMP
2. Crea la Base de datos para el Software GLPI y setea el usuario para administrar la misma.
3. Modifica el archivo php.ini correspondiente a nuestra version php seleccionada en la instalación.
4. Habilita los modulos en apache2
5. Configura los Virtual host
6. Recupera los Certificados LetsEncrypts desde un BK.
7. Recupera el backups de una instalación previa de GLPI.
9. Modifica archivos de configuración de GLPI para adecuar la nueva DB.
10. Habilita todos los sitios. 


## Requisitos

- Un servidor con una distribución Linux compatible, de preferencia Debian 12 Bookworn.
- Ansible instalado en la máquina de control.

## Variables del Rol

El rol `glpi-lamp` utiliza las siguientes variables que pueden ser configuradas según tus necesidades:

 Versiones de php seleccionada para la instalación, puedes optar por la versión 8.2 o 8.3:

```
php_version: 8.2
```

# Listas de extensiones de PHP a activar

```
php_extensions:
  - intl
  - mysqli
```

 Variables para la configuración del VirtualHost.

```
 http_port: 80
 server_admin: "admin@example.com"
 http_domain: "example.com"
 http_host: "glpi"
```

# Variables para la configuración de la base de datos, utilizamos un dicccionario para definir los valores de esta.

 Ten encuenta que si escribes contraseñas o datos sensibles en el archivo de variables seran expuestos, y en el pero de los casos entrara en la historia de un repositorio Git, para evitar esto es recomendable que utilices el Vault para los secretos y en dicha sección hagas referencia a una variable dentro del vault.

```
 site_config_install:
   db:
     glpi_mysql_db: "default"
     glpi_mysql_user: "default"
     glpi_mysql_password: "{{ vault_glpi_mysql_password }}"
```

# Definición de backups.


# Definimos el nombre del archivo de backup de GLPI

 El archivo empaquetado y comprimido contine el software GLPI en la ruta **/var/www/html/glpi**

```
BackupGLPI: "glpi.tar.gz"
```

# Definimos el nombre del archivo de backup de Letsencrypt

 El archivo contiene los certificados ubicados bajo **/etc/letsencrypt**

```
BackupCertificados: "letsencrypt.tar.gz"
```

# Respaldo de DB.

 Respaldo de un dump de la base datos a importar.

 ```
BackupDB: "glpi999_bk.sql.tar.tgz"
 ```

Nota: Recuerda que el nombre del archivo SQL debe ser igual a la clave **glpi_mysql_db** definida en el diccionario  **site_config_install**.


# Recomendación de definición de variables

Podes comentar esta sección y definir estas varibales en tu host_vars, recomendamos este método ya que se integra mejor con nuestro proyecto IaC(Infraestructure as a code).

Recomendamos la siguiente estrucutura:
```
host_vars/glpi.tudominio.lan/
├── files
│   ├── glpi999_bk.sql
│   ├── glpi.tar.gz
│   ├── letsencrypt.tar.gz
├── vars
│   └── 10-glpi-lamp.yml
└── vault
    └── main.yml
```

Utilisamos la Carpeta **files** para dejar el backups a recuperar.


## Dependencias

Este rol no tiene dependencias de otros roles de Ansible.

## Ejemplo de Playbook

Para ejecutar este rol utilizamos un playbook que ejecuta multiples Roles, se indica cual ejecutar indicando el tag **glpi-lamp-install**.

Ubicamos el cplaybook debajo de la carpeta Playbooks de nuestro proyecto
40_deploy_config.yml

```yaml
---
- name: Configuración de host para servicios en VMs y Contenedores
  hosts: all
  remote_user: usuarioConPoderSudo
  become: true
  gather_facts: yes

  pre_tasks:
  - name: ver los facts del host
    ansible.builtin.debug:
      var: ansible_facts
    tags: never,facts

  roles:
     - role: glpi-lamp
     tags:
       - glpi-lamp-install
     when: groups.conGlpi is defined and inventory_hostname in groups.conGlpi
```

Nota: Recuerda que utilizamos un grupo de control **conGlpi** para afectar solo a sistemas destinados a glpi.

En nuestro caso utilizamos el playbok **site.yml** para integrar los demas playbooks que realizan operaciones o llaman a otros Roles.

Ejemplo de ejecución:

```shell
ansible-playbook -i hosts_prod site.yml --limit glpi.tudominio.lan -v --tags glpi-lamp-install
```

     