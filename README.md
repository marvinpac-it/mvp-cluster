# Installation et mise à jour de Kubernetes avec Kubespray

Les étapes suivantes décrivent la mise à jour du cluster K8S de Marvinpac. Cette procédure
peut être utilisée également pour ajouter des noeuds au cluster, ou changer sa configuration.

## Cloner la dernier version de Kubespray et y intégrer ce repository comme inventaire
```
cd /tmp
# Adapter la branche à la dernière release stable de Kubespray
git clone --single-branch --depth=1 --branch v2.24.1 git@github.com:kubernetes-sigs/kubespray.git
cd kubespray/inventory/
git clone --depth 1 git@github.com:marvinpac-it/mvp-cluster.git
cd ..
```

## Création de l'environnement virtuel python pour Ansible
```
# Si python 3 et venv ne sont pas encore installés
sudo apt install python3.10-venv

python3 -m venv venv
source venv/bin/activate
pip install -U -r requirements.txt
```

## Mise à jour du cluster
```
# Adapter la version de kube à la dernière version stable supportée par Kubespray
ansible-playbook -i inventory/mvp-cluster/hosts.yaml -u core --key-file ~/.ssh/flatcar_ssh.pem  -b -e kube_version=v1.28.3 upgrade-cluster.yml

```
