# Installation et mise à jour de Kubernetes avec Kubespray

Les étapes suivantes décrivent la mise à jour du cluster K8S de Marvinpac. Cette procédure
peut être utilisée également pour ajouter des noeuds au cluster, ou changer sa configuration.

## Cloner la dernier version de Kubespray et y intégrer ce repository comme inventaire
> [!WARNING]  
> Adapter la branche à la dernière release stable de Kubespray.
```Shell
cd /tmp
git clone --single-branch --depth=1 --branch v2.27.0 git@github.com:kubernetes-sigs/kubespray.git
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
> https://github.com/kubernetes-sigs/kubespray/tree/release-2.25?tab=readme-ov-file#supported-components

```Shell
ansible-playbook -i inventory/mvp-cluster/hosts.yaml -u core --key-file ~/.ssh/flatcar_ssh.pem  -b upgrade-cluster.yml
```

Kubespray 1.26.0 fails on Flatcar OS. I created an issue and PR, as the PR is not yet merged, a workaround is the following command for installation:
```Shell
ansible-playbook -i inventory/mvp-cluster/hosts.yaml -u core --key-file ~/.ssh/flatcar_ssh.pem -b -e '{"ansible_interpreter_python_fallback":['/opt/bin/python']}' upgrade-cluster.yml
```
Once this first bug resolved, a second one appears which is fixed by applying the following patch: https://github.com/kubernetes-sigs/kubespray/pull/11224

## Mise à jour de la commande kubectl
```Shell
curl -LO "https://dl.k8s.io/release/v1.31.4/bin/linux/amd64/kubectl"
chmod +x kubectl
./kubectl version
sudo mv kubectl /usr/local/bin/kubectl
sudo chown root:root /usr/local/bin/kubectl
```

## Vérifier les numéros de version
```Shell
kubectl version
Client Version: v1.31.4
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.31.4
```

## Mise à jour du fichier de config avec lequel on se connecte
```Shell
ssh -i ~/.ssh/flatcar_ssh.pem core@flatcar01 "sudo cat /etc/kubernetes/admin.conf | sed 's/127\.0\.0\.1/flatcar01/g'" > ~/.kube/config
```

## Revue des changements du template inventaire
De temps en temps il peut être bon de revoir la partie inventaire en la comparant à celle qui est fournie comme example.
Un diff entre `inventory/sample` et `inventory/mvp-cluster` (ce repo) permet de revoir ce qui a été ajouté dans l'exemple
depuis que ce repo a été créé. S'il y a des sections nouvelles on peut les incorporer.
```Shell
diff -r inventory/sample/ inventory/mvp-cluster/
```

**Output:**
```Diff
Only in mvp-cluster/: .git
Only in mvp-cluster/: .gitignore
diff -r sample/group_vars/all/all.yml mvp-cluster/group_vars/all/all.yml
3c3
< bin_dir: /usr/local/bin
---
> bin_dir: /opt/bin
diff -r sample/group_vars/k8s_cluster/addons.yml mvp-cluster/group_vars/k8s_cluster/addons.yml
16c16
< metrics_server_enabled: false
---
> metrics_server_enabled: true
104,105c104,105
< ingress_nginx_enabled: false
< # ingress_nginx_host_network: false
---
> ingress_nginx_enabled: true
> ingress_nginx_host_network: true
119,121c119,121
< # ingress_nginx_namespace: "ingress-nginx"
< # ingress_nginx_insecure_port: 80
< # ingress_nginx_secure_port: 443
---
> ingress_nginx_namespace: "ingress-nginx"
> ingress_nginx_insecure_port: 80
> ingress_nginx_secure_port: 443
129,130c129,134
< # ingress_nginx_extra_args:
< #   - --default-ssl-certificate=default/foo-tls
---
> ingress_nginx_extra_args:
>   - --default-ssl-certificate=ingress-nginx/star-marvinpac-com
>   - --publish-status-address=192.168.77.149
>   - --update-status=true
>   - --controller-class=k8s.io/ingress-nginx
>   - --watch-ingress-without-class=true
132,133c136,137
< # ingress_nginx_class: nginx
< # ingress_nginx_without_class: true
---
> ingress_nginx_class: nginx
> ingress_nginx_without_class: true
180c184
< metallb_enabled: false
---
> metallb_enabled: true
187,203c191,212
< # metallb_config:
< #   speaker:
< #     nodeselector:
< #       kubernetes.io/os: "linux"
< #     tolerations:
< #       - key: "node-role.kubernetes.io/control-plane"
< #         operator: "Equal"
< #         value: ""
< #         effect: "NoSchedule"
< #   controller:
< #     nodeselector:
< #       kubernetes.io/os: "linux"
< #     tolerations:
< #       - key: "node-role.kubernetes.io/control-plane"
< #         operator: "Equal"
< #         value: ""
< #         effect: "NoSchedule"
---
> metallb_config:
>   speaker:
>     nodeselector:
>       kubernetes.io/os: "linux"
>     tolerations:
>       - key: "node-role.kubernetes.io/control-plane"
>         operator: "Equal"
>         value: ""
>         effect: "NoSchedule"
>   controller:
>     nodeselector:
>       kubernetes.io/os: "linux"
>     tolerations:
>       - key: "node-role.kubernetes.io/control-plane"
>         operator: "Equal"
>         value: ""
>         effect: "NoSchedule"
>   address_pools:
>     primary:
>       ip_range:
>         - 192.168.77.110-192.168.77.150
>       auto_assign: true
217,218c226,227
< #   layer2:
< #     - primary
---
>   layer2:
>     - primary
diff -r sample/group_vars/k8s_cluster/k8s-cluster.yml mvp-cluster/group_vars/k8s_cluster/k8s-cluster.yml
129c129
< kube_proxy_strict_arp: false
---
> kube_proxy_strict_arp: true
161a162,163
> searchdomains:
>   - intranet-mvp.ch
200,201c202,203
< enable_coredns_k8s_external: false
< coredns_k8s_external_zone: k8s_external.local
---
> enable_coredns_k8s_external: true
> coredns_k8s_external_zone: k8s.intranet-mvp.ch
Only in mvp-cluster/: hosts.yaml
Only in mvp-cluster/: README.md
```
