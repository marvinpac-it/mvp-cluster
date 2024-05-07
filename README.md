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
