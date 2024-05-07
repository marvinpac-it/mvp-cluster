# Installation et mise à jour de Kubernetes avec Kubespray

Les étapes suivantes décrivent la mise à jour du cluster K8S de Marvinpac. Cette procédure
peut être utilisée également pour ajouter des noeuds au cluster, ou changer sa configuration.

## Cloner la dernier version de Kubespray et y intégrer ce repository comme inventaire
> [!WARNING]  
> Adapter la branche à la dernière release stable de Kubespray.
```
cd /tmp
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

> [!NOTE]
> S'assurer d'avoir la clé privée flatcar_ssh.pem (Vaultwarden) au bon endroit.

> [!WARNING]  
> Adapter kube_version à la dernière version supportée par Kubespray. La version supportée peut être
> trouvée dans le README.md https://github.com/kubernetes-sigs/kubespray?tab=readme-ov-file#supported-components

```
ansible-playbook -i inventory/mvp-cluster/hosts.yaml -u core --key-file ~/.ssh/flatcar_ssh.pem  -b -e kube_version=v1.28.6 upgrade-cluster.yml
```
