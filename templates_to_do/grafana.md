helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

kubectl create namespace monitoring

helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ingress
  namespace: monitoring
  annotations:
    kubernetes.io/ingress.class: traefik
    traefik.ingress.kubernetes.io/router.entrypoints: websecure
    traefik.ingress.kubernetes.io/router.tls: "true"
spec:
  tls:
    - hosts:
        - grafana.example.com
      secretName: grafana-tls
  rules:
    - host: grafana.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: monitoring-grafana
                port:
                  number: 80

GRAFANA_HOST="grafana.example.com"

openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -out ./grafana/tls.crt -keyout ./grafana/tls.key -subj "/CN=$- {GRAFANA_HOST}/O=${GRAFANA_HOST}" -addext "subjectAltName = DNS:${GRAFANA_HOST}"

kubectl create secret tls grafana-tls \
  --cert=grafana/tls.crt \
  --key=grafana/tls.key \
  -n monitoring

kubectl apply -f grafana-ingress.

