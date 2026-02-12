# Workshop Environment Tools Reference

The workshop environment includes a variety of pre-installed command-line tools that workshop users can access without needing to install additional software. This reference documents the available tools and their primary use cases in workshop exercises.

## JSON and YAML Processing

### jq

**Purpose:** Command-line JSON processor for querying, filtering, and transforming JSON data.

**Primary use case:** Parsing and extracting specific fields from JSON output in workshop exercises.

**Examples:**
- Extract a specific field: `jq '.metadata.name' deployment.json`
- Filter array elements: `jq '.items[] | select(.spec.replicas > 1)' deployments.json`
- Pretty-print JSON: `jq '.' raw-output.json`

**Common flags:**
- `-r`: Output raw strings (remove JSON quotes)
- `-s`: Read entire input into a single array
- `-c`: Compact output (one-line JSON)

---

### yq

**Purpose:** YAML query processor, similar to jq but for YAML files.

**Primary use case:** Parsing and transforming YAML manifests, configuration files, and Kubernetes resources in workshop exercises.

**Examples:**
- Extract a YAML field: `yq '.spec.replicas' deployment.yaml`
- Filter array elements: `yq '.spec.containers[] | select(.name == "app")' pod.yaml`
- Update a field: `yq -i '.spec.replicas = 3' deployment.yaml`

**Common flags:**
- `-r`: Output raw strings
- `-i`: In-place edit (modifies the file)
- `-Y`: Output as YAML (forced)
- `-j`: Output as JSON

---

## Python Dependency and Package Management

### uv

**Purpose:** Fast Python package installer and resolver, designed as a replacement for pip and pip-tools.

**Primary use case:** Installing Python dependencies and managing virtual environments quickly within workshop exercises.

**Examples:**
- Install a package: `uv pip install flask requests`
- Create and activate a virtual environment: `uv venv workshops-env && source workshops-env/bin/activate`
- Install from requirements file: `uv pip install -r requirements.txt`

**Common flags:**
- `--no-cache`: Disable pip cache
- `--python <version>`: Specify Python version
- `-p`: Short for `--python`

**Note:** This workshop environment includes a modern version of uv that generates `main.py` when creating new Python projects. Older uv versions generated `hello.py` as the default entry point; if you encounter workshop instructions written for older versions, adjust file names accordingly.

---

## Text Editors

### nano

**Purpose:** Simple, beginner-friendly text editor for editing files in the terminal.

**Primary use case:** Quick file edits when a graphical editor is not available or for workshop exercises emphasizing terminal-based workflows.

**Examples:**
- Open a file for editing: `nano configuration.yaml`
- Create a new file: `nano newfile.txt`
- Common commands within nano: `Ctrl+X` to exit, `Ctrl+O` to save, `Ctrl+W` to search

---

## System Utilities

### tree

**Purpose:** Displays the directory structure in a tree format.

**Primary use case:** Visualizing project structure or directory layouts in workshop exercises, particularly useful in documentation or diagrams.

**Examples:**
- Display directory tree: `tree`
- Limit depth: `tree -L 2`
- Show files only: `tree -f`
- Ignore certain directories: `tree -I 'node_modules|.git'`

**Common flags:**
- `-L <depth>`: Limit recursion depth
- `-d`: Show directories only
- `-f`: Use full paths
- `-I <pattern>`: Exclude directories matching pattern

---

### watch

**Purpose:** Executes a command repeatedly at regular intervals and displays the output full-screen.

**Primary use case:** Monitoring command output in real-time, particularly useful for observing changing state in workshop exercises.

**Examples:**
- Monitor a directory: `watch ls -la`
- Monitor Kubernetes pods: `watch kubectl get pods`
- Monitor with custom interval: `watch -n 2 kubectl get deployments`
- Use with kubectl instead of -w: `watch kubectl get pods -n default` shows current state snapshots rather than streaming updates

**Common flags:**
- `-n <seconds>`: Update interval in seconds (default: 2)
- `-d`: Highlight differences between updates
- `-x`: Execute the command exactly as specified (disables shell interpretation)

**Note:** The `watch` command is particularly useful with kubectl for monitoring cluster state. While `kubectl get pods -w` streams live updates showing change events, `watch kubectl get pods` shows the current state at regular intervals, which often provides clearer visibility of the actual cluster state rather than intermediate changes.

---

### tmux

**Purpose:** Terminal multiplexer that allows multiple terminal sessions within a single window.

**Primary use case:** Managing multiple concurrent terminal sessions or splitting the terminal view during complex, multi-step workshop exercises.

**Examples:**
- Create a new session: `tmux new-session -s myworkshop`
- Attach to a session: `tmux attach-session -t myworkshop`
- Split window vertically: `Ctrl+B %`
- Split window horizontally: `Ctrl+B "`
- Switch panes: `Ctrl+B <arrow-key>`

**Common keybindings (after `Ctrl+B`):**
- `%`: Split vertically
- `"`: Split horizontally
- `c`: Create new window
- `n`/`p`: Next/previous window
- `d`: Detach session

---

## Networking and Load Testing

### ncat / netcat

**Purpose:** Networking utility for reading and writing data across TCP and UDP networks.

**Primary use case:** Testing network connectivity, debugging services, or basic data transmission in workshop exercises.

**Examples:**
- Open a listening port: `ncat -l 8080`
- Connect to a remote service: `ncat example.com 80`
- Send a message: `echo "Hello" | ncat example.com 9000`

**Note:** Both `ncat` (netcat flavor from nmap project) and `netcat` are always available in the workshop environment. They have similar core functionality but differ in command-line options and feature support. Use `--help` to see available options for each tool.

---

### siege

**Purpose:** HTTP load testing and benchmarking tool.

**Primary use case:** Load testing web applications or APIs to measure performance and stability under concurrent traffic.

**Examples:**
- Simple load test: `siege -c 10 -r 5 http://localhost:8080`
- Load test with custom header: `siege -c 10 -r 5 -A "Custom-Header: value" http://localhost:8080`
- Run for a specific duration: `siege -c 50 -t 1M http://localhost:8080` (1 minute)

**Common flags:**
- `-c <num>`: Number of concurrent users
- `-r <num>`: Number of repetitions
- `-t <duration>`: Time-based testing (S/M/H for seconds/minutes/hours)
- `-g <url>`: Generate a URLs file from a logfile
- `-v`: Verbose output

---

### bombardier

**Purpose:** Fast HTTP load testing tool, similar to siege but with emphasis on simplicity and speed.

**Primary use case:** Benchmarking HTTP endpoints with high concurrency and straightforward output.

**Examples:**
- Basic load test: `bombardier -c 10 -n 1000 http://localhost:8080`
- Target request rate: `bombardier -r 100 -d 10s http://localhost:8080` (100 req/s for 10s)
- Save results to file: `bombardier -c 50 -n 5000 http://localhost:8080 > results.txt`

**Common flags:**
- `-c <num>`: Number of concurrent connections
- `-n <num>`: Total number of requests
- `-r <num>`: Rate limit (requests per second)
- `-d <duration>`: Duration of test (s/m/h)

---

## Container Image Management

### skopeo

**Purpose:** Tool for working with container images and registries without requiring a container runtime.

**Primary use case:** Copying, inspecting, and manipulating container images across registries or from registries to local storage.

**Examples:**
- Inspect a remote image: `skopeo inspect docker://registry.example.com/myapp:latest`
- Copy an image: `skopeo copy docker://docker.io/library/nginx:latest docker://registry.example.com/nginx:latest`
- Convert image format: `skopeo copy docker://myapp:latest oci:myapp-oci:latest`
- List tags in a registry: `skopeo list-tags docker://quay.io/kubernetes/hyperkube`

**Common flags:**
- `--src-creds <user:pass>`: Credentials for source registry
- `--dest-creds <user:pass>`: Credentials for destination registry
- `--no-creds`: Skip credentials
- `--format`: Specify output format (json, yaml)

---

## Kubernetes Tools

### kubectl

**Purpose:** Command-line interface for Kubernetes cluster management.

**Primary use case:** Interacting with Kubernetes clusters, deploying applications, and managing cluster resources in workshop exercises.

**Examples:**
- List resources: `kubectl get pods`, `kubectl get services`, `kubectl get deployments`
- Describe a resource: `kubectl describe pod my-pod -n default`
- Apply a manifest: `kubectl apply -f deployment.yaml`
- Get logs from a pod: `kubectl logs my-pod`
- Port forward to a pod: `kubectl port-forward my-pod 8080:8080`

**Common flags:**
- `-n <namespace>`: Specify namespace
- `--context <context>`: Switch Kubernetes context
- `-l <label>`: Filter by label selector
- `-o <format>`: Output format (json, yaml, wide, etc.)

---

### k9s

**Purpose:** Terminal user interface (TUI) for browsing and interacting with Kubernetes resources.

**Primary use case:** Visual navigation and management of Kubernetes clusters, providing a more user-friendly alternative to kubectl commands.

**Examples:**
- Start k9s: `k9s`
- Navigate resources: Use arrow keys to browse pods, services, and deployments
- Filter resources: Type `/` to open filter mode
- Execute commands on resources: Select a resource and press `:` for actions
- View logs: Select a pod and press `l`

**Common hotkeys:**
- `?`: Show help
- `/`: Filter resources
- `Enter` or `E`: Edit resource
- `L`: View logs
- `D`: Describe resource
- `:`: Command mode for actions
- `Q`: Quit

---

### helm

**Purpose:** Package manager for Kubernetes applications.

**Primary use case:** Installing, upgrading, and managing pre-configured Kubernetes applications using Helm charts.

**Examples:**
- Add a Helm repository: `helm repo add stable https://charts.helm.sh/stable`
- Install a chart: `helm install my-release stable/nginx`
- List releases: `helm list`
- Upgrade a release: `helm upgrade my-release stable/nginx --values values.yaml`
- Uninstall a release: `helm uninstall my-release`

**Common flags:**
- `-n <namespace>`: Target namespace
- `-f <values-file>`: Use custom values file
- `--version <version>`: Specify chart version
- `--dry-run`: Simulate installation without applying

---

### kustomize

**Purpose:** Kubernetes native configuration management tool for customizing YAML manifests.

**Primary use case:** Managing environment-specific Kubernetes configurations without duplicating manifest files.

**Examples:**
- Build a kustomization: `kustomize build .`
- Apply kustomized config: `kubectl apply -k .`
- Generate customized output: `kustomize build . > final-manifest.yaml`

**Common use cases:**
- Overlays for different environments (dev, staging, production)
- Patching resources (replicas, image versions, labels)
- Composing multiple base configurations

---

## Carvel Tools Suite

The Carvel project provides a suite of specialized Kubernetes configuration and deployment tools. Each tool serves a distinct purpose in the configuration and deployment workflow.

### ytt

**Purpose:** YAML templating tool for configuring and composing YAML files with powerful templating capabilities.

**Primary use case:** Generating Kubernetes manifests and configuration files dynamically based on input values and logic.

**Examples:**
- Render a template: `ytt -f template.yaml -f values.yaml`
- Use data values: `ytt -f config.yaml --data-value replicas=3`
- Define and use functions: `ytt -f manifest.yaml -f functions.yaml`

**Common flags:**
- `-f <file>`: Input file(s)
- `--data-value <key>=<value>`: Pass data values
- `-d <file>`: Load data file
- `-o <format>`: Output format (yaml, json)

---

### imgpkg

**Purpose:** Tool for working with container images, image bundling, and image relocation.

**Primary use case:** Packaging application dependencies as image bundles and relocating images across registries.

**Examples:**
- List images in a bundle: `imgpkg describe bundle -b registry.example.com/my-app-bundle:v1.0`
- Create a bundle: Use configuration files to define bundle contents
- Copy a bundle: `imgpkg copy -b registry.example.com/app-bundle:v1.0 --to-repo target-registry.example.com/app`

---

### kbld

**Purpose:** Tool for building and managing container images as part of deployment pipelines.

**Primary use case:** Building container images and integrating image building into Kubernetes deployment workflows.

**Examples:**
- Build images: `kbld -f config.yaml`
- Resolve image references: `kbld -f deployment.yaml --imgpkg-lock-output lock.yml`

---

### kapp

**Purpose:** Kubernetes application deployment and lifecycle management tool (successor to kubectl apply workflow).

**Primary use case:** Deploying and managing Kubernetes applications with safer update semantics and built-in change visualization.

**Examples:**
- Deploy an application: `kapp deploy -a my-app -f deployment.yaml`
- List deployed applications: `kapp list`
- Inspect application resources: `kapp inspect -a my-app`
- Delete an application: `kapp delete -a my-app`

**Common flags:**
- `-a <app-name>`: Application name
- `-f <file>`: Manifest file(s)
- `--vcenter <context>`: Display changes before deploying
- `-n <namespace>`: Target namespace

---

### kwt

**Purpose:** Kubernetes workload tool for local development workflows.

**Primary use case:** Running Kubernetes workloads locally with network connectivity to a remote cluster.

**Examples:**
- Create a local network connection: `kwt net start`
- Run a pod locally: `kwt pod exec -i my-pod --command <command>`

---

### vendir

**Purpose:** Directory vendor tool for managing and synchronizing external dependencies into a directory.

**Primary use case:** Vendoring external manifests, scripts, and dependencies as part of a configuration management workflow.

**Examples:**
- Sync dependencies: `vendir sync`
- Specify sources in `vendir.yml` file to define what gets vendored

**Common use cases:**
- Pulling remote Kubernetes manifests
- Vendoring Helm chart dependencies
- Managing versions of shared configuration

---

### kctrl

**Purpose:** Command-line tool for managing Kubernetes package and dependency management (experimental/beta).

**Primary use case:** Package management and dependency resolution workflows for Kubernetes applications.

**Examples:**
- Manage packages (usage and flags may vary based on kctrl version)
- Resolve dependencies declaratively

**Note:** kctrl functionality and command structure may change, so consult `kctrl --help` and Carvel documentation for current usage.

---

## See Also

For workshops using these tools, consider the following resources:

- [Carvel Project Documentation](https://carvel.dev/)
- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [Helm Chart Repository](https://artifacthub.io/)
