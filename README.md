# Proyecto 2 – Cluster Kubernetes

[Sustentación (Video)](https://eafit-my.sharepoint.com/:v:/g/personal/lmgiraldo4_eafit_edu_co/EZRAovwrIuhAuUAPl4JDKRkBaRdktIzHNYlmIMu84OuTJA?e=WLFT2O)


### Materia: ST0263 - Tópicos Especiales en Telemática, 2024-2
### Estudiante(s): 
- Vanessa Velez Restrepo, vavelezr@eafit.edu.co
- Luis Miguel Giraldo, lmgiraldo4@eafit.edu.co
- Luisa Maria Polanco, lmpolanco1@eafit.edu.co
- Sara Maria Cardona Villada, smcardonav@eafit.edu.co
- Santiago Arias Higuita, sariash@eafit.edu.co
### Profesor: Alvaro, @eafit.edu.co

---

## Descripción del Proyecto

Este proyecto busca desplegar una aplicación en un clúster Kubernetes, usando microk8s y aprovechando instancias EC2 en AWS. La idea es recrear lo que hicimos en el reto anterior (Reto 2), pero ahora montando el clúster desde cero en un ambiente IaaS, con alta disponibilidad en varias capas (aplicación, base de datos, y almacenamiento), además de un sistema de archivos compartido a través de un servidor NFS.

## 1. Aspectos que Cumplimos

- **Infraestructura en AWS**: Configuramos un clúster de Kubernetes usando tres instancias EC2 `t2.small` con Ubuntu, todo gestionado por microk8s.
- **Alta Disponibilidad**: Configuramos alta disponibilidad para la aplicación, así que no hay un solo punto de falla en esa capa.
- **Sistema de Archivos Compartido**: Montamos un servidor NFS en el clúster, permitiendo que varios nodos puedan acceder al mismo almacenamiento.
- **Escalabilidad Manual**: Aunque no llegamos a la escalabilidad automática, sí configuramos el clúster para permitir agregar nodos manualmente si es necesario.
- **Dominio y HTTPS**: Configuramos un dominio y acceso seguro (HTTPS) usando Ingress.

## 1.1. ¿Qué logramos?

- Instalación y configuración de microk8s en un clúster de Kubernetes en AWS.
- Implementación de almacenamiento compartido usando NFS.
- Dominio seguro con HTTPS para acceso al servicio.
- Escalabilidad manual de nodos.

## 1.2. ¿Qué no logramos?

- **Escalado Automático de Nodos**: Solo se logró la configuración para escalado manual.
- **Alta Disponibilidad en la Base de Datos**: Aunque la base de datos está funcional, la alta disponibilidad en esta capa no fue alcanzada.
- **Visualización del certificado SSL en el navegador**: El certificado si se encuentra listo y en correctas condiciones pero a la hora de visualizar en el navegador no fue posible que se aplicaran los cambios

---

## 2. Diseño General y Arquitectura

- **Arquitectura**: Configuramos tres instancias de EC2 dentro de una VPC dedicada, con subredes y un grupo de seguridad específico. Una instancia actúa como `master` y las otras dos como `slave`.
- **Buenas Prácticas**: Usamos Kubernetes para gestionar contenedores y configuramos microk8s para implementar el clúster en un ambiente IaaS. También incluimos NFS como sistema de archivos compartido.
- **Seguridad**: Configuramos un grupo de seguridad que permite solo los puertos esenciales (ver la lista de puertos en la imagen proporcionada).

---

## 3. Ambiente de Desarrollo


- **Librerías y Paquetes**: 
  - Kubernetes (`microk8s`) - última versión
  - DNS, Storage, Ingress habilitados en microk8s
- **Instrucciones para Configurar y Ejecutar**:
  ```bash
  sudo snap install microk8s --classic
  sudo usermod -a -G microk8s $USER
  sudo chown -f -R $USER ~/.kube
  newgrp microk8s
  microk8s enable dns storage ingress
  ```

  - **Parámetros de Configuración**:
  - **Puertos**: Configurados en el grupo de seguridad (ver imagen con la lista completa): 16443, 10250, 10255, 25000, entre otros.
  - **Token para Unión al Clúster**: Generado en el `master` y utilizado en los `slaves` para que se unan al clúster.
  - **IP del Master**: `10.0.1.147:25000` para la unión de nodos.

## 4. Ambiente de Ejecución


- **Dominio y Dirección IP**: Configuración de un dominio (`modapeluda.tech`) para el acceso.
- **Configuración de Parámetros**:
- Configuración del balanceador de carga en Ingress.
- Variables de entorno y parámetros necesarios para el dominio y HTTPS.

## 5. Configuración del Servidor NFS (en slave1)

Para habilitar un sistema de almacenamiento compartido en el clúster, configuramos un servidor NFS en el nodo `slave1`. Esto permite que múltiples nodos accedan a los mismos archivos, útil para mantener la consistencia y compartir datos entre pods en el clúster.

Pasos principales:

1. **Instalación del Servidor NFS**: Se instaló el servidor NFS en el nodo `slave1` y se creó un directorio para almacenamiento compartido.
2. **Configuración de Permisos**: Se ajustaron los permisos del directorio compartido para que todos los nodos pudieran acceder.
3. **Configuración de Exportación**: Se definieron las IPs de los nodos permitidos para acceder al almacenamiento NFS.
4. **Ajustes de Firewall**: Se permitió el tráfico NFS en el firewall para las IPs del clúster.

---

## 6. Creación de PersistentVolume (PV) y PersistentVolumeClaim (PVC)

Para que los pods en Kubernetes puedan utilizar el almacenamiento NFS, se crearon un PersistentVolume (PV) y un PersistentVolumeClaim (PVC). Esto asegura que cualquier pod que lo necesite pueda acceder al almacenamiento compartido sin problemas.

---

## 7. Configuración en el Nodo Master

En el nodo `master` se llevaron a cabo varias configuraciones importantes para desplegar los servicios de MySQL y WordPress con almacenamiento persistente. A continuación se describe de manera general cada paso clave que realizamos.

### 7.1 Creación de Archivos YAML para Configuración

Se crearon los archivos `.yaml` necesarios para definir los volúmenes persistentes, servicios, y despliegues de MySQL y WordPress.

- **`pv_mysql.yaml`**: Este archivo define un `PersistentVolume` (PV) y un `PersistentVolumeClaim` (PVC) para MySQL. El propósito del `PersistentVolume` es proporcionar almacenamiento duradero dentro del clúster de Kubernetes, asegurando que los datos de la base de datos persistan incluso en caso de reinicio o eliminación de los pods. El volumen se asigna al directorio `/mnt/data` en el nodo.

  Para aplicar esta configuración, ejecutamos el comando:
  ```bash
  microk8s kubectl apply -f pv_mysql.yaml
  ```

Este comando le indica a MicroK8s que lea el archivo `pv_mysql.yaml` y cree el volumen persistente especificado en el clúster.

**`mysql_deployment.yaml`**: Este archivo contiene la configuración del despliegue y servicio de MySQL. Al aplicar este archivo, se despliega un contenedor que utiliza la imagen `mysql:5.6`, se configura una variable de entorno para la contraseña de MySQL y se asigna el puerto 3306 para la base de datos. Además, el despliegue está diseñado para gestionar el ciclo de vida de los pods de MySQL, lo que permite realizar actualizaciones y garantizar que el servicio esté siempre disponible.

Para aplicar este despliegue, ejecutamos:

```bash
microk8s kubectl apply -f mysql_deployment.yaml
```

Al ejecutar este comando, MicroK8s lee el archivo `mysql_deployment.yaml` y aplica su contenido en el clúster, creando o actualizando el despliegue especificado para MySQL, incluyendo los pods, contenedores, y demás recursos necesarios para ejecutar la base de datos.

**Acceso al Pod de MySQL**: Para interactuar directamente con el contenedor MySQL dentro del pod, utilizamos el siguiente comando:

```bash
microk8s kubectl exec -it mysql-<pod-id> -- bash
```

Aquí, `mysql-<pod-id>` se reemplaza con el identificador del pod específico generado por el despliegue de MySQL. Este comando abre una sesión de shell interactiva en el contenedor de MySQL, lo cual es útil para verificar el estado de la base de datos o realizar configuraciones adicionales.

**`pv_wordpress.yaml`**: Este archivo especifica el `PersistentVolume` y el `PersistentVolumeClaim` para WordPress, lo que permite que los archivos de la aplicación se almacenen de forma persistente. El volumen se asigna al directorio `/mnt/wp_data` en el nodo.

**`wordpress.yaml`**: Este archivo define la configuración de despliegue y servicio para WordPress, incluyendo:

- La imagen `wordpress:4.8-apache`.
- Variables de entorno necesarias para la conexión a la base de datos, como la IP del servicio MySQL y la contraseña.
- Exposición del puerto 80 para permitir el acceso al servicio web de WordPress.
- Configuración de un `LoadBalancer` para gestionar el tráfico entrante y permitir acceso externo.
- Montaje del volumen persistente en `/var/www/html` para almacenar los archivos de WordPress de manera duradera.

### 7.2 Despliegue de los Servicios

Después de crear los archivos de configuración, ejecutamos `kubectl apply -f` para aplicar cada archivo al clúster de Kubernetes, desplegando tanto MySQL como WordPress en el entorno de producción. Esta configuración asegura que ambos servicios cuenten con almacenamiento persistente, permitiendo que los datos de la base de datos y los archivos de WordPress se mantengan disponibles aunque los pods se reinicien o eliminen.

### 7.3 Resumen del Proceso de Despliegue

Esta arquitectura de almacenamiento persistente permite que WordPress funcione correctamente en un entorno distribuido, con una base de datos confiable y almacenamiento compartido, asegurando que el contenido y la configuración de la aplicación se mantengan accesibles en todo momento dentro del clúster de Kubernetes. Gracias a esta configuración, es posible garantizar una persistencia de datos adecuada para ambos servicios, haciendo el entorno más robusto y preparado para un uso prolongado.


## Imágenes del Proceso de Desarrollo del Proyecto

A continuación, se presentan capturas que documentan el desarrollo y configuración del proyecto, desde la creación de volúmenes persistentes hasta el despliegue y la configuración de servicios en Kubernetes. Estas imágenes ilustran los comandos ejecutados, los archivos `.yaml` utilizados y el estado final de los servicios desplegados en el clúster.

---
![image](https://github.com/user-attachments/assets/a23d9390-b060-41c5-b81a-a102898a1973)

![image](https://github.com/user-attachments/assets/06551a53-dcc2-458c-9751-72f347daec35)

![image](https://github.com/user-attachments/assets/201d4b7b-c0d4-487f-aadf-5d80caedcf92)

![image](https://github.com/user-attachments/assets/7556c47b-d321-4739-8f2c-79a4e291ce4b)



