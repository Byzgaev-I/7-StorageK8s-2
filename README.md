# Домашнее задание к занятию «Хранение в K8s. Часть 2»

### Цель задания
В тестовой среде Kubernetes нужно создать PV и продемонстрировать запись и хранение файлов.

### Чеклист готовности к домашнему заданию
1. Установленное K8s-решение (MicroK8S)
2. Установленный локальный kubectl
3. Редактор YAML-файлов с подключенным git-репозиторием

## Задание 1: Создание локального PV

### Подготовка окружения

1. Создание директории для данных:
   
```bash
sudo mkdir /mnt/data
sudo chmod 777 /mnt/data
```
### Манифесты

Все манифесты находятся в директории [manifests/task1](https://github.com/Byzgaev-I/7-StorageK8s-2/tree/main/manifests/task1)

StorageClass [storage-class.yaml](https://github.com/Byzgaev-I/7-StorageK8s-2/blob/main/manifests/task1/storage-class.yaml)  
PersistentVolume [pv.yaml](https://github.com/Byzgaev-I/7-StorageK8s-2/blob/main/manifests/task1/pv.yaml)
PersistentVolumeClaim [pvc.yaml](https://github.com/Byzgaev-I/7-StorageK8s-2/blob/main/manifests/task1/pvc.yaml) 
Deployment [deployment.yaml](https://github.com/Byzgaev-I/7-StorageK8s-2/blob/main/manifests/task1/deployment.yaml)

Проверка работоспособности

### 1 Создание и проверка StorageClass и PV:

```bash
kubectl apply -f manifests/task1/storage-class.yaml
kubectl apply -f manifests/task1/pv.yaml
kubectl get sc,pv
```
foto

### 2 Создание PVC и проверка:

```bash
kubectl apply -f manifests/task1/pvc.yaml
kubectl get pvc
```
foto
### 3 Запуск Deployment и проверка подов:

```bash
kubectl apply -f manifests/task1/deployment.yaml
kubectl get pods -l app=storage-test
```
### 4 Проверка записи данных:
```bash
POD_NAME=$(kubectl get pods -l app=storage-test -o jsonpath="{.items[0].metadata.name}")
kubectl exec -it $POD_NAME -c multitool -- cat /data/output.txt
```


### Проверка сохранности данных

### 1 Удаление Deployment и PVC:
```
kubectl delete -f manifests/task1/deployment.yaml
kubectl delete -f manifests/task1/pvc.yaml
kubectl get pv
```
### 2 Проверка данных на ноде:

```bash
sudo cat /mnt/data/output.txt
```

## Задание 2: Создание Deployment с NFS

### Подготовка окружения

Все манифесты находятся в директории [manifests/task2](https://github.com/Byzgaev-I/7-StorageK8s-2/tree/main/manifests/task2)

[nfs-deployment.yaml](https://github.com/Byzgaev-I/7-StorageK8s-2/blob/main/manifests/task2/nfs-deployment.yaml)  
[nfs-pvc.yaml](https://github.com/Byzgaev-I/7-StorageK8s-2/blob/main/manifests/task2/nfs-pvc.yaml)


### 1 Установка и настройка NFS:

```bash
sudo apt install -y nfs-kernel-server
sudo mkdir -p /srv/nfs/kubedata
sudo chown -R nobody:nogroup /srv/nfs/kubedata
sudo chmod 777 /srv/nfs/kubedata
```

### 2 Настройка NFS-сервера:

```bash
echo "/srv/nfs/kubedata *(rw,sync,no_subtree_check,no_root_squash)" | sudo tee -a /etc/exports
sudo exportfs -ra
```

### Проверка работоспособности

### 1 Статус NFS-сервера:

```bash 
sudo systemctl status nfs-kernel-server
```

### 2 Проверка работы NFS:

```bash
kubectl exec -it $POD_NAME -- sh -c "echo 'Test NFS Storage' > /data/test.txt"
kubectl exec -it $POD_NAME -- cat /data/test.txt
sudo cat /srv/nfs/kubedata/test.txt
```













