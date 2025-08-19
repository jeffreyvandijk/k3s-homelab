1. setup cluster(installatie)
master node(kube-control-01):

curl -sfL https://get.k3s.io | sh -s - server   --cluster-init   --node-taint CriticalAddonsOnly=true:NoExecute   --tls-san 192.168.1.30 --write-kubeconfig-mode 644

sudo cat /var/lib/rancher/k3s/server/node-token

master node(kube-control-02):

curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1.30:6443   K3S_TOKEN=K105e18da37050142192afa12555a71d041fd4aed81321d967ef85959e08c22d032::server:07e234095178c93a4866265fb0323a39 sh -s - server   --node-taint CriticalAddonsOnly=true:NoExecute   --tls-san 192.168.1.30 --write-kubeconfig-mode 644

worker node(kube-worker-01)
curl -sfL https://get.k3s.io | K3S_URL=https://load_balancer_ip_or_hostname:6443 K3S_TOKEN=mynodetoken sh -

worker node(kube-worker-02)
curl -sfL https://get.k3s.io | K3S_URL=https://load_balancer_ip_or_hostname:6443 K3S_TOKEN=mynodetoken sh -




4. Installatie Helm & Longhorn

master node(kube-control-01):

sudo apt update && sudo apt install -y open-iscsi
sudo systemctl enable --now iscsid

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

helm repo add longhorn https://charts.longhorn.io
helm repo update
kubectl create namespace longhorn-system
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
helm install longhorn longhorn/longhorn   --namespace longhorn-system
kubectl patch storageclass longhorn   -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'



