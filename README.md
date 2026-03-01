# TallerFebrero2026
Trabajos durante el taller de Linux
# Infraestructura NFS + autofs + systemd HTTP (Ansible)

# Objetivo

Construir una infraestructura mínima y reproducible utilizando Ansible y versionado en GitHub que incluya:

- Servidor NFS en CentOS Stream 9
- Cliente Ubuntu 24.04 con automount (autofs)
- Servicio systemd que publique el share mediante `python3 -m http.server`
- Ejecución idempotente mediante playbook

---

# Arquitectura

NFS Server (fileserver)  
└── /srv/nfs/shared (exportado por NFS)

Clientes (grupo ubuntu)  
└── /mnt/shared (montado con autofs)

Servicio adicional  
└── Servidor web Python en puerto 8080 mostrando el contenido del share


---

#  Requisitos previos

- Ansible instalado en máquina de control
- Acceso SSH a ambos nodos
- Usuario con privilegios sudo
- Servidores recién instalados

---

## Playbooks

| Playbook | Descripción |
|----------|------------|
| `nfs-server.yaml` | Instala y configura el servidor NFS, exporta el directorio y configura firewall |
| `nfs-cliente.yaml` | Configura clientes NFS con automontaje mediante autofs |
| `nfs-systemd.yaml` | Crea y habilita un servicio systemd que publica el share vía HTTP |

---

#  Ejecución

Desde la raíz del repositorio:

ansible-playbook -i inventories/hosts.ini site.yml

La ejecución es idempotente.
Puede ejecutarse múltiples veces sin modificar configuraciones ya aplicadas.

---

#  Verificación

## En nfs01

systemctl is-active nfs-server

exportfs -v

Resultado esperado:
- active
- Export en modo rw para 192.168.10.0/24

---

## En Clientes (autofs)

Antes de acceder:

mount | grep /mnt/shared

No debe mostrar nada.

Forzar montaje:

ls /mnt/shared
mount | grep /mnt/shared

Ahora debe mostrar el mount NFS activo.

---

## Servicio HTTP

systemctl status shared-http --no-pager

curl -I http://localhost:8080/

curl http://localhost:8080/README-NFS.txt

journalctl -u shared-http -n 50 --no-pager

Resultado esperado:
- Servicio activo
- HTTP 200
- Devuelve contenido del archivo README-NFS.txt

---

Infraestructura automatizada, reproducible e idempotente.
