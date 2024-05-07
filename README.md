# Inventory files for Kubespray deployment

## Clone latest stable Kubespray version and clone the mvp_cluster inventory
```
cd /tmp
# Adapt branch tag to latest release
git clone --single-branch --depth=1 --branch v2.24.1 git@github.com:kubernetes-sigs/kubespray.git
cd kubespray/inventory/
git clone --depth 1 git@github.com:marvinpac-it/mvp-cluster.git
cd ..
```

## Create python virtual environment for Kubespray
```
# Si python 3 et venv ne sont pas encore installés
sudo apt install python3.10-venv

python3 -m venv venv
source venv/bin/activate
pip install -U -r requirements.txt
```

## Installation du cluster
```
ansible-playbook -i inventory/mvp-cluster/hosts.yaml -u core --key-file ~/.ssh/flatcar_ssh.pem  -b -e kube_version=v1.28.3 cluster.yml
```

## Mise à jour du cluster
```
ansible-playbook -i inventory/mvp-cluster/hosts.yaml -u core --key-file ~/.ssh/flatcar_ssh.pem  -b -e kube_version=v1.28.3 upgrade-cluster.yml

```
