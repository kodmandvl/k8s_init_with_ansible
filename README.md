k8s-on-rhel8
==========

# Init Kubernetes Cluster on RHEL8/CentOS8/RockyLinux8/AlmaLinux8 with Ansible (a simpler and faster scenario than Kubespray)

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

# Мой пример inventory

Файл hosts.yml:

```yml
[kubernetes_masters]
cp2.k8s

[kubernetes_workers]
cp3.k8s
node3.k8s
```

Здесь у нас в примере на машине cp2.k8s будут компоненты control-plane, а cp3.k8s и node3.k8s будут worker нодами (не обращайте внимание на имя машины cp3.k8s, просто такие три машины свободные остались для этих тестов).

Также стоит отметить, что данная роль k8s-on-rhel8 подразумевает инициализацию кластера только с одним узлом control-plane (master).

Ну и также должен отметить, что запускал эти плейбуки на машинах, на которых уже установлено много нужных инструментов и пакетов. Поэтому в том случае, если подняты виртуальная машина какой-то минимальной конфигурации, возможно, понадобится установить туда еще ряд пакетов и инструментов.

Здесь стоит отметить, что при таком развёртывании берется дефолтный /etc/containerd/config.toml, имеет смысл добавить в него зеркала (как я делал при ручной установке кластера), нужно добавить зеркала после строки `[plugins."io.containerd.grpc.v1.cri".registry.mirrors]`, например:

```
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://mirror.gcr.io","https://registry-1.docker.io"]
```

# Примеры запуска

```bash
# Добавить ключи и проверить коннект:
ansible all -m ping -i hosts.yml -u root
ansible all -m shell -a 'echo "`whoami`@`hostname -f` (`hostname -i`)"' -i hosts.yml -u root
# Развёртывание кластера (запуск роли k8s-on-rhel8):
ansible-playbook -i hosts.yml -l all -u root init.yml
# Развёртывание кластера (запуск роли k8s-on-rhel8), начать с определенного шага:
ansible-playbook -i hosts.yml -l all -u root init.yml --start-at-task "Initialize The Cluster"
# Установка дополнительных инструментов для работы с кластером (etcd, etcdctl, etcdutl, crictl, nerdctl, helm), запуск роли k8s-tools:
ansible-playbook -i hosts.yml -l all -u root tools.yml
# Посмотреть, появились ли эти инструменты:
ansible all -m shell -a 'ls -alFtrh /usr/local/bin/' -i hosts.yml -u root
# Удаление кластера (например, перед переустановкой или для зачистки):
ansible-playbook -i hosts.yml -l all -u root reset.yml
# Удаление кластера и пакетов kubeadm, kubelet, kubectl, containerd (например, перед переустановкой или для зачистки):
ansible-playbook -i hosts.yml -l all -u root destroy.yml
```

# Что дальше

Стоит отметить, что после развёртывания кластера (init.yml) и установки дополнительных инструментов для работы (tools.yml) вам потребуется выполнить еще ряд настроек, выходящих за рамки данных сценариев. В частности, вам могут потребоваться:

- установка MetalLB
- установка Ingress-Nginx
- установка Istio
- установка Cert-Manager
- настройка сетевых политик
- настройка PSP/PSA/OPA
- другие необходимые вам настройки в кластере
- и т.д.

Можно использовать этот списочек действий как чеклист.