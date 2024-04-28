# Getting Started with KubeVirt

## Install KubeVirt operator

```bash
export VERSION=$(curl -s https://storage.googleapis.com/kubevirt-prow/release/kubevirt/kubevirt/stable.txt)
echo $VERSION
kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-operator.yaml
```

## Create KubeVirt CR:

```bash
kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-cr.yaml
```

## Verify components

```bash
kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.phase}"
```

## Wait to be ready

```bash
kubectl wait kubevirt.kubevirt.io/kubevirt -n kubevirt \
    --for=jsonpath='{.status.phase}=Deployed' --timeout=120s

kubectl get all -n kubevirt
```

## You need to install virtctl binary or krew plugin

```bash
VERSION=$(kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.observedKubeVirtVersion}")
ARCH=$(uname -s | tr A-Z a-z)-$(uname -m | sed 's/x86_64/amd64/') || windows-amd64.exe
echo ${ARCH}
curl -L -o virtctl https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/virtctl-${VERSION}-${ARCH}
chmod +x virtctl
sudo install virtctl /usr/local/bin

# or
kubectl krew install virt
```

## Check VM manifest

```bash
bat vm.yaml
```

## Create the VM

```bash
kubectl apply -f vm.yaml
```

## Check the VM --> it's stopped

```bash
kubectl get vms
kubectl get vms -o yaml testvm
```

## start the VM

```bash
virtctl start testvm

# or
kubectl virt start testvm

# or
kubectl patch virtualmachine testvm --type merge -p \
    '{"spec":{"running":true}}'
```

## Check the VMIs

>[!TIP]
>VirtualMachine defines the desired state of a VM, while VirtualMachineInstance represents the actual running instance of the VM within Kubernetes.

```bash
kubectl get vmis
kubectl get vmis -o yaml testvm
```

## Connect to the console of the VM:

```bash
virtctl console testvm
```

## Stop and delete VM

```bash
kubectl virt stop testvm
kubectl delete vm testvm
```

## Place SSH public key into a Secret

```bash
kubectl create secret generic my-pub-key --from-file=key1=${HOME}/.ssh/id_rsa.pub
```

## See ssh configuration for the VM

```
bat ssh-vm.yaml
```

## Create and start the VM

```bash
kubectl create -f ssh-vm.yaml
```

## Check status

```bash
kubectl get vm,vmi,pod
```

## Connect via SSH

```bash
kubectl virt ssh fedora@testvm --local-ssh=true
```

## Try to access a kubernetes service:

```bash
# run this on the VM
curl -k https://kubernetes:443
```

## Create nginx service on a seperate tab

```bash
# run this in the k8s cluster
kubectl create -f nginx.yaml
```

## try to access the service

```bash
# run this on the VM
curl http://nginx-service 
```
