# external-dns

Deploy a controller which can interface with ExternalDNS-compatible webhooks [providers](https://kubernetes-sigs.github.io/external-dns/v0.14.0/tutorials/webhook-provider/).

This works similarly to the way ExternalDNS does, where the webhook itself is deployed alongside the controller container, except in this case the controller container is an `external-dns-integration` pod which translates Kubizone Zones into webhook calls.

An example deployment of this chart might look like this, using the [Hetzner ExternalDNS webhook integration](https://github.com/mconfalonieri/external-dns-hetzner-webhook) to do the heavy lifting:

```yaml
# Since the webhook will be running as a sidecar in the pod, it can be accessed using `localhost`
webhookEndpoint: "http://localhost:8888"

sidecars:
  - name: hetzner-webhook
    image: ghcr.io/mconfalonieri/external-dns-hetzner-webhook:v0.6.0
    ports:
      - containerPort: 8888
        name: webhook
      - containerPort: 8080
        name: http
    livenessProbe:
      httpGet:
        path: /health
        port: http
      initialDelaySeconds: 10
      timeoutSeconds: 5
    readinessProbe:
      httpGet:
        path: /ready
        port: http
      initialDelaySeconds: 10
      timeoutSeconds: 5
    env:
      - name: WEBHOOK_HOST
        value: "0.0.0.0"
      - name: HETZNER_API_KEY
        valueFrom:
          secretKeyRef:
            name: my-hetzner-dns-api-token
            key: HCLOUD_TOKEN
```

Then configure the secret with your API token:

```bash
kubectl create secret generic --from-literal=HCLOUD_TOKEN=oQgFUb5abNGNABZR4JwTzwM8IQre4aPl my-hetzner-dns-api-token
```