# mssql_Ansible_auto_failover_AG
Auto failover Allways On


# Ansible Failover Availability Groups

 > Este repositorio contiene un conjunto de playbooks de Ansible que te permiten automatizar el proceso de failover automático de SQL Server utilizando grupos de disponibilidad (AG) Always On.

### **Estructura del repositorio**:

```
├── inventario/
│ └── inventory.yml             # Inventario para testing. En Prod se usa el de AWX
│── 
│ └── tasks/
|   |__ alter_availability_group_task.yml
|   |__ cancel_playbook.yml
│   └── check_synchronized.yml
│   └── dba_requerimientos.yml
│
├── failover_Ag_return.yml
├── failover_Ag_start.yml
└── README.md
```

### Configuración del inventario:

 En el inventario solo se deben agregar las instancias a reiniciar.

> 💀 ***No colocar instancias Secundarias de Always On.*** En el inventario no puede figurar un nodo secundario que esté relacionado al primario. Ansible realiza el failover de los AG que están como Primarios al nodo secundario. 

## Ejecución del failover automático
Para ejecutar el failover automático, utiliza el playbook failover_ag_start.yml ubicado en el directorio. Ejecuta el siguiente comando desde la máquina de control:

ansible-playbook -i inventory/inventory_ivan.yml failover_ag_start.yml

# Seguimiento de Tareas:

Descripción de tareas que ejecuta el playbook.

Task:
- [**Chequeo de requerimientos**] - Conjunto de tareas que verifica que la Base "DBA" tenga la tabla "status_check_pre_failover" y el Sp "insert_check_pre_failover". Si no existen las crea ✨Magic ✨
- [**Ejecutar Stored Procedure (inventory Alias AG)**] - Ejecuta el Stored Procedure por el cual inserta el estado de los AG en la tabla para después usar esos datos en el proceso de failover.
- [**Change AG Async to Sync**] - Modifica todos los AG Primary en estado Synchronous commit.
- [**Check SYNC**] - Conjunto de tareas que verifica el estado de los Availability Groups antes de realizar el failover. La tarea tiene un parámetro de reintentos en segundos. Si los AG Primary no se encuentran en SYNCHRONIZED después de un tiempo determinado, se cancela el playbook con la tarea **cancel_playbook**.
- [**Failover de Availabulity Groups**] - Tarea que realiza el Failover de los AG Primary
- [**Check SYNC -----Reboot**] - Se revisa nuevamente que los AG estén sincronizados antes de realizar el reinicio del Servidor. Esta tarea también tiene un límite de reintentos.
- [**Reboot Server**] - Realiza el reinicio del servidor.
- [**Finished work**] - Mensaje de finalización de tareas.

Tareas Faltantes:

- Terminar de desarrollar el Playbook de failover_Ag_retunr.yml para volver las instancias a su estado previo. Tener en cuenta como parámetro la tabla status_check_pre_failover
