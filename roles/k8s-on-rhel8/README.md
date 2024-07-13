k8s-on-rhel8
==========

Установка и настройка кластера Kubernetes на RHEL8/CentOS8/RockyLinux8/AlmaLinux8 (форк роли kubernetes Рустема Галиева)

- Установить необходимые пакеты;
- Настроить главный сервер;
- Настройка рабочих узлов.

Требования
------------

- Настройки SELinux и брандмауэра не считаются проблемой для этой роли.

Переменные роли
--------------

Переменные, которые можно изменить при желании

| Переменные                                   | Стандартные значения          | Комментарий
| :---                                         | :---                          | :---                                                    
| `kubernetes_user`                            |                               | Пользователь k8s
|                                              |                               |
| `kubernetes_network`                         | calico                        | Сеть для подов
|                                              |                               |


Для Red Hat family:

| Переменные                                   | Значения
|:---                                          |:---
| `kubernetes_url_repo`                        | Kubernetes репозиторий
|                                              |
| `kubernetes_url_key`                         | Kubernetes GPG ключ
|                                              |
| `kubernetes_containerd`                      | Containerd репозиторий
|                                              |
| `kubernetes_pkg`                             | Kubernetes необходимые пакеты
|                                              |
| `containerd_pkg`                             | Containerd необходимые пакеты
|                                              |


Для корректной работы плейбук и инвентарь должны быть созданы, как показано ниже.
---------------------------------------------------------------------------------

Playbook
=========
```
- hosts: kubernetes_masters,kubernetes_workers
  become: yes
  roles:
    - /path/to/roles/k8s-on-rhel8

```
Inventory
=========
```
[kubernetes_masters]
master_node01

[kubernetes_workers]
workers_node01
workers_node02
workers_node03
```
