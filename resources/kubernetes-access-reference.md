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

## Workshop Session Proxy for Services

When a deployment has a Service but no Ingress — or when creating a manual Ingress is not appropriate — the **workshop session proxy** provides an alternative way to expose the Service for browser access. Unlike `curl` from the terminal, this approach makes the Service accessible in the user's web browser and can embed it in the workshop dashboard.

### Configuring Session Ingresses

Add `spec.session.ingresses` in the workshop definition to route external access through the workshop session proxy. Multiple entries can be listed:

```yaml
# Path: spec.session
session:
  ingresses:
  - name: app
    protocol: http
    host: app.$(session_namespace).svc
    port: 8080
```

This automatically creates an Ingress that routes external access at `app-$(session_hostname)` through the workshop session proxy, which proxies the request to the Service `app.$(session_namespace).svc` in the session namespace.

### Advantages Over Manual Ingress

The workshop session proxy solves several problems that arise with manually created Ingress resources:

- **Automatic HTTPS:** If Educates is deployed with secure ingress, the proxied ingress is automatically secured with HTTPS. A manually created Ingress would only support HTTP because no TLS certificate is available for it.
- **No mixed content errors:** A manually created HTTP-only Ingress cannot be embedded in a dashboard tab via an iframe when the workshop dashboard itself is served over HTTPS — browsers block this as mixed content. The session proxy eliminates this problem.
- **Authentication gating:** Access through the session proxy is gated by the same authentication as the workshop dashboard itself, restricting access to only the workshop user.

### Embedding in a Dashboard Tab

Combine `spec.session.ingresses` with `spec.session.dashboards` to embed the proxied service directly in the workshop dashboard:

```yaml
# Path: spec.session
session:
  ingresses:
  - name: app
    protocol: http
    host: app.$(session_namespace).svc
    port: 8080
  dashboards:
  - name: App
    url: "$(ingress_protocol)://app-$(session_hostname)/"
```

This creates a dashboard tab named "App" that safely embeds the Service. Workshop instructions can then use the `dashboard:open-dashboard` clickable action to reveal the tab at the appropriate point after the application is deployed:

````markdown
```dashboard:open-dashboard
name: App
```
````

This pattern is preferable to using `curl` from the terminal when the deployed application provides an interactive web interface.

**IMPORTANT:** If you define a session ingress with a `name` property of `app`, you cannot also have the user create a separate Kubernetes Ingress using a hostname of the form `app-$(session_hostname)`. The names would conflict because the workshop session proxy has already claimed that hostname.

## Interacting with the Kubernetes Web Console

When the Kubernetes web console is enabled (`spec.session.applications.console.enabled: true`), it appears as a "Console" dashboard tab. Clickable actions can guide users to specific views within the console.

### Opening the Console Tab

Use the `dashboard:open-dashboard` clickable action to reveal the Console tab:

````markdown
```dashboard:open-dashboard
name: Console
```
````

### Navigating to Specific Console Views

Use `dashboard:reload-dashboard` to direct the Console tab to a specific URL within the Kubernetes web console. The console provides URLs for common resource views:

| View | URL Path |
|------|----------|
| Overview | `/#/overview` |
| Deployments | `/#/deployment` |
| Pods | `/#/pod` |
| Services | `/#/service` |
| ConfigMaps | `/#/configmap` |
| Secrets | `/#/secrets` |
| Ingresses | `/#/ingress` |

**Example — navigate to the Deployments view:**

````markdown
```dashboard:reload-dashboard
name: Console
url: {{< param ingress_protocol >}}://console-{{< param session_hostname >}}/#/deployment?namespace={{< param session_namespace >}}
```
````

**Example — drill down into a specific Deployment:**

````markdown
```dashboard:reload-dashboard
name: Console
url: {{< param ingress_protocol >}}://console-{{< param session_hostname >}}/#/deployment/{{< param session_namespace >}}/nginx-deployment?namespace={{< param session_namespace >}}
```
````

The URL pattern for drilling into a specific resource is `/#/<resource-type>/<namespace>/<resource-name>`.

**IMPORTANT:** Always include the query string parameter `namespace={{< param session_namespace >}}` in every console URL. Without it, the namespace selector dropdown in the console reverts to the "default" namespace, which will cause confusion if the user navigates elsewhere within the console.

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

**Common images that require `baseline` policy:** Many popular container images run as root or bind to privileged ports by default and will fail under the `restricted` policy. Notable examples include:

- **nginx** — runs as root and listens on port 80
- **httpd** (Apache HTTP Server) — runs as root and listens on port 80
- **mysql** / **mariadb** — run as root
- **postgres** — runs as root
- **redis** — runs as root
- **mongo** — runs as root
- **memcached** — runs as root
- **elasticsearch** — runs as root

If a workshop uses any of these images (or similar ones), set the security policy to `baseline`. When in doubt, check whether an image runs as a non-root user by looking for a `USER` directive in its Dockerfile — if there is none, it almost certainly runs as root and needs `baseline`.

## Resource Budgets and Quotas

When a workshop deploys workloads into the session namespace, resource quotas and limit ranges control how much CPU and memory can be consumed. Understanding these constraints is critical to ensuring workshop deployments succeed without hitting quota limits.

### Budget Sizes

The resource budget is set via `spec.session.namespaces.budget`. If not specified, no quotas or limits are applied. The available budget sizes and their total resource quotas are:

| Budget    | CPU   | Memory |
|-----------|-------|--------|
| small     | 1000m | 1Gi    |
| medium    | 2000m | 2Gi    |
| large     | 4000m | 4Gi    |
| x-large   | 8000m | 8Gi    |
| xx-large  | 8000m | 12Gi   |
| xxx-large | 8000m | 16Gi   |

A value of 1000m is equivalent to 1 CPU.

### Default Limit Ranges

Each budget applies a limit range with default resource requests and limits. When a pod specification does **not** include explicit resource requests or limits, these defaults are automatically applied to every container.

The default memory limit ranges are:

| Budget    | Default Request | Default Limit |
|-----------|-----------------|---------------|
| small     | 128Mi           | 256Mi         |
| medium    | 128Mi           | 512Mi         |
| large     | 128Mi           | 512Mi         |
| x-large   | 128Mi           | 512Mi         |
| xx-large  | 128Mi           | 512Mi         |
| xxx-large | 128Mi           | 512Mi         |

The default CPU limit ranges are:

| Budget    | Default Request | Default Limit |
|-----------|-----------------|---------------|
| small     | 50m             | 250m          |
| medium    | 50m             | 500m          |
| large     | 50m             | 500m          |
| x-large   | 50m             | 500m          |
| xx-large  | 50m             | 500m          |
| xxx-large | 50m             | 500m          |

### Avoiding Quota Exhaustion with Multiple Replicas

The default limit ranges have a significant impact when deploying workloads with multiple replicas. Because every container without an explicit resource specification receives the default limits, a Deployment scaled to several replicas can quickly exhaust the namespace memory quota.

**Example problem:** With a `medium` budget (2Gi total memory), the default memory limit per container is 512Mi. A Deployment with 4 replicas would require 4 × 512Mi = 2Gi — consuming the entire quota and leaving no room for any other workloads.

**Best practice — set explicit resource requests and limits on deployments:** Rather than relying on the default limit ranges, specify resource requests and limits directly in the pod template of each Deployment. If the application actually needs only 64Mi of memory, setting that explicitly avoids wasting quota on the 512Mi default:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 4
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app
        image: myimage:latest
        resources:
          requests:
            cpu: 50m
            memory: 64Mi
          limits:
            cpu: 100m
            memory: 64Mi
```

With this approach, 4 replicas consume only 4 × 64Mi = 256Mi instead of 4 × 512Mi = 2Gi.

**When generating workshop content that deploys workloads into Kubernetes, follow this decision process:**

1. **Calculate the total memory required** — multiply the number of replicas by the default memory limit for the chosen budget size (e.g., 512Mi for `medium`).
2. **If the total exceeds the budget quota** — add explicit `resources.requests` and `resources.limits` to the Deployment's pod template, sized to what the application actually needs. This is the preferred solution.
3. **Only if right-sizing resources is still insufficient** — increase the budget to the next size that accommodates the workload. Prefer the smallest budget that fits.

### Overriding Default Limit Ranges

If the budget quota is sufficient but the default request/limit values need adjustment, override them with `spec.session.namespaces.limits`:

```yaml
# Path: spec.session
session:
  namespaces:
    budget: medium
    limits:
      defaultRequest:
        cpu: 50m
        memory: 64Mi
      default:
        cpu: 250m
        memory: 128Mi
```

Only the properties you specify are overridden — the rest retain their defaults.

### Setting the Budget in the Workshop Definition

Set the resource budget in the workshop definition (`resources/workshop.yaml`):

```yaml
# Path: spec.session
session:
  namespaces:
    budget: medium
```

**IMPORTANT:** The resource budget controls resources in the **session namespace** where the user deploys workloads. It is separate from `spec.session.resources`, which controls the resources available to the workshop container itself (the container running the terminal and dashboard).
