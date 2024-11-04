# Proyecto 2 – Cluster Kubernetes

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


