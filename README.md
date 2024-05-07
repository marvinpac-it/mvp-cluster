# Installation et mise à jour de Kubernetes avec Kubespray

Les étapes suivantes décrivent la mise à jour du cluster K8S de Marvinpac. Cette procédure
peut être utilisée également pour ajouter des noeuds au cluster, ou changer sa configuration.

## Cloner la dernier version de Kubespray et y intégrer ce repository comme inventaire
> [!WARNING]  
> Adapter la branche à la dernière release stable de Kubespray.
```Shell
cd /tmp
git clone --single-branch --depth=1 --branch v2.24.1 git@github.com:kubernetes-sigs/kubespray.git
cd kubespray/inventory/
git clone --depth 1 git@github.com:marvinpac-it/mvp-cluster.git
cd ..
```

## Création de l'environnement virtuel python pour Ansible
```Shell
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
> trouvée dans le README.md en version stable de Kubernetes. Attention à ne pas prendre le README.md de la
> branche master mais de bien afficher la release en question.
> https://github.com/kubernetes-sigs/kubespray/tree/release-2.24?tab=readme-ov-file#supported-components

```Shell
ansible-playbook -i inventory/mvp-cluster/hosts.yaml -u core --key-file ~/.ssh/flatcar_ssh.pem  -b -e kube_version=v1.28.6 upgrade-cluster.yml
```
## Mise à jour de la commande kubectl
```Shell
curl -LO "https://dl.k8s.io/release/v1.28.6/bin/linux/amd64/kubectl"
chmod +x kubectl
./kubectl version
sudo mv kubectl /usr/local/bin/kubectl
sudo chown root:root /usr/local/bin/kubectl
```

## Vérifier les numéros de version
```Shell
kubectl version
Client Version: v1.28.6
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.28.6
```

## Revue des changements du template inventaire
De temps en temps il peut être bon de revoir la partie inventaire en la comparant à celle qui est fournie comme example.
Un diff entre `inventory/sample` et `inventory/mvp-cluster` (ce repo) permet de revoir ce qui a été ajouté dans l'exemple
depuis que ce repo a été créé. S'il y a des sections nouvelles on peut les incorporer.
```Shell
diff -r inventory/sample/ inventory/mvp-cluster/
```
