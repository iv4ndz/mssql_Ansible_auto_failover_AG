# Automatización de Failover con Ansible para Grupos de Disponibilidad (AG) Always On en SQL Server

 > Este repositorio proporciona una serie de playbooks de Ansible diseñados para automatizar el proceso de failover automático en SQL Server, utilizando Grupos de Disponibilidad Always On (AG). Estos playbooks simplifican la gestión y aseguran la alta disponibilidad de las instancias de SQL Server.

### **Estructura del repositorio**:

```
├── inventario/
│   └── inventory.yml             # Inventario para pruebas. En producción se usa el de AWX.
│
├── tasks/
│   ├── alter_availability_group_task.yml  # Tarea para modificar el AG
│   ├── cancel_playbook.yml                # Tarea para cancelar la ejecución del playbook
│   ├── check_synchronized.yml             # Verifica si los AG están sincronizados
│   └── dba_requerimientos.yml             # Chequea requerimientos previos
│
├── failover_Ag_return.yml                # Playbook para revertir el failover
├── failover_Ag_start.yml                 # Playbook para iniciar el failover
└── README.md                             # Archivo de documentación

```

### Configuración del inventario:

En el inventario solo se deben incluir las instancias que serán reiniciadas.

> 💀 **No agregues instancias secundarias de Always On al inventario***. Ansible realiza el failover de los AG configurados como primarios hacia los nodos secundarios. Por lo tanto, un nodo secundario no debe estar relacionado con el primario en el inventario.

## Ejecución del failover automático
Para ejecutar el failover automático, utiliza el playbook failover_ag_start.yml ubicado en el directorio de playbooks. Desde la máquina de control, ejecuta el siguiente comando:

```
ansible-playbook -i inventario/inventory_ivan.yml failover_ag_start.yml
```

# Seguimiento de Tareas:

Descripción de tareas que ejecuta el playbook.

Task:
- [**Chequeo de requerimientos**] - Verifica que la base de datos "DBA" tenga la tabla status_check_pre_failover y el stored procedure insert_check_pre_failover. Si no existen, las tareas se encargan de crearlas automáticamente.
- [**Ejecutar Stored Procedure (inventory Alias AG)**] - Ejecuta el Stored Procedure por el cual inserta el estado de los AG en la tabla para después usar esos datos en el proceso de failover.
- [**Change AG Async to Sync**] - Modifica todos los AG Primary en estado Synchronous commit.
- [**Check SYNC**] - Conjunto de tareas que verifica el estado de los Availability Groups antes de realizar el failover. La tarea tiene un parámetro de reintentos en segundos. Si los AG Primary no se encuentran en SYNCHRONIZED después de un tiempo determinado, se cancela el playbook con la tarea **cancel_playbook**.
- [**Failover de Availabulity Groups**] - Tarea que realiza el Failover de los AG Primary
- [**Check SYNC -----Reboot**] - Se revisa nuevamente que los AG estén sincronizados antes de realizar el reinicio del Servidor. Esta tarea también tiene un límite de reintentos.
- [**Reboot Server**] - Realiza el reinicio del servidor.
- [**Finished work**] - Mensaje de finalización de tareas.

Tareas Faltantes:

- Es necesario completar el desarrollo del playbook failover_ag_return.yml para restaurar las instancias a su estado previo. Asegúrate de usar la tabla status_check_pre_failover como parámetro para este proceso.
