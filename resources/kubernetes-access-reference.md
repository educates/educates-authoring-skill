# Kubernetes Access in Workshops

This guide covers key considerations when authoring workshops that provide users with Kubernetes access. It applies to any workshop where users interact with Kubernetes using tools like `kubectl` or deploy applications into a cluster.

## Session Namespace

Each workshop session is assigned a single Kubernetes namespace. The user's kubeconfig and RBAC rules restrict access to only this namespace — users cannot view or modify resources in other namespaces.

The session namespace is configured as the default context in the user's kubeconfig, so commands like `kubectl get pods` will target the correct namespace without requiring the `--namespace` flag.

### Referencing the Namespace Name

When workshop instructions need to include the namespace name, use the appropriate method depending on the context.

**In workshop markdown (rendered instructions):**

Use the Hugo shortcode to insert the session namespace:

```markdown
Deploy the application to the `{{< param session_namespace >}}` namespace.
```

**In terminal commands:**

Use the `SESSION_NAMESPACE` environment variable, which is available in the workshop terminal:

```markdown
```terminal:execute
command: kubectl get pods -n $SESSION_NAMESPACE
```​
```

**IMPORTANT:** Because the session namespace is already the default context in kubeconfig, specifying `-n $SESSION_NAMESPACE` is only necessary when you want to be explicit in the instructions. For most `kubectl` commands, omitting the namespace flag will work correctly.

## Accessing Services from the Workshop Container

The workshop container (where the user's terminal runs) is located in a **different namespace** from the session namespace where the user deploys applications. This means that when the user needs to access a Kubernetes Service they have created — for example, using `curl` from the terminal — they must qualify the hostname with the session namespace.

### Service DNS Within the Cluster

To reach a Service named `app` in the session namespace, use the format `app.<namespace>.svc`:

**In terminal commands:**

```markdown
```terminal:execute
command: curl http://app.$SESSION_NAMESPACE.svc:8080
```​
```

**In workshop markdown (to show the expanded hostname):**

```markdown
Access the application at `app.{{< param session_namespace >}}.svc`.
```

If the fully qualified domain name is needed, append the cluster domain. Educates provides the cluster domain via the `CLUSTER_DOMAIN` environment variable and `cluster_domain` Hugo parameter (the domain is typically `cluster.local`, but this is not guaranteed):

**In terminal commands:**

```markdown
```terminal:execute
command: curl http://app.$SESSION_NAMESPACE.svc.$CLUSTER_DOMAIN:8080
```​
```

**In workshop markdown:**

```markdown
Access the application at `app.{{< param session_namespace >}}.svc.{{< param cluster_domain >}}`.
```

## Ingress Hostname Requirements

When exposing a deployed application via a Kubernetes Ingress, the hostname used in the Ingress resource **must incorporate the session-specific hostname** for that workshop instance. Kyverno policies enforced on the session will block any Ingress that does not follow this convention.

The session hostname is available as:

- `$SESSION_HOSTNAME` — environment variable in the terminal
- `{{< param session_hostname >}}` — Hugo shortcode in workshop markdown
- `$(session_hostname)` — data variable in `spec.session.objects` of the workshop definition

### Constructing Ingress Hostnames

The convention is to prefix the session hostname with the application or service name, separated by a dash. For a service named `app`:

**In workshop markdown (e.g., showing an Ingress resource to apply):**

```markdown
```editor:append-lines-to-file
file: ~/exercises/ingress.yaml
text: |
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: app
  spec:
    rules:
    - host: app-{{< param session_hostname >}}
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: app
              port:
                number: 8080
```​
```

**In `spec.session.objects` of the workshop definition (`resources/workshop.yaml`):**

```yaml
# Path: spec.session
session:
  objects:
  - apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: app
    spec:
      rules:
      - host: app-$(session_hostname)
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app
                port:
                  number: 8080
```

**IMPORTANT:** The Ingress hostname must include the session hostname. Using an arbitrary hostname will be rejected by the Kyverno policies applied to the session.

## Pod Security Policy

By default, the session namespace enforces a **restricted** security policy, aligned with the Kubernetes [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/). This imposes the following constraints on workloads deployed into the namespace:

- Containers **cannot run as the root user**
- Containers **cannot bind to privileged ports** (ports below 1024, such as port 80)

### Overriding the Security Policy

If the workshop requires deploying images that run as root or listen on privileged ports, override the security policy in the workshop definition (`resources/workshop.yaml`):

```yaml
# Path: spec.session
session:
  namespaces:
    security:
      policy: baseline
```

Setting the policy to `baseline` relaxes the restrictions, allowing containers to run as root and bind to privileged ports.

**IMPORTANT:** Only use `baseline` when the workshop genuinely requires it — for example, when deploying third-party images that run as root or web servers that listen on port 80. Keep the default `restricted` policy whenever possible.
