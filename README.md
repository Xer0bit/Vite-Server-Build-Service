# Vite Server Build Service

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Node.js Version](https://img.shields.io/badge/Node.js-v18%2B-green.svg)]()
[![Docker Support](https://img.shields.io/badge/Docker-Supported-blue.svg)]()

## Overview
The Vite Server Build Service is an enterprise-grade backend infrastructure designed to securely provision, orchestrate, and proxy ephemeral Vite environments. It provides isolated, containerized execution contexts for arbitrary user-supplied code, exposing dynamic preview URLs via an automated reverse proxy layer.

## System Architecture
* **Orchestration Layer:** Manages the lifecycle of ephemeral containers via the Docker API, handling instantiation, resource allocation, and termination operations.
* **Execution Sandbox:** Executes the Vite development server (`vite dev`) within heavily constrained, non-root container environments. Employs read-only base image mounts with writable overlay layers for user file modifications.
* **Proxy & Routing Engine:** Dynamically updates routing tables to map internal container ports to unique external subdomains (e.g., `[instance-id].sandbox.domain.com`). Injects required CORS headers and host configurations directly into the Vite process.

## Core Capabilities
* **Zero-Trust Execution:** Prevents host system compromise through strict cgroup limits, namespace isolation, and default-deny network egress policies.
* **Automated Lifecycle Management:** Implements active state monitoring and TTL (Time-To-Live) enforcement to terminate idle instances and mitigate resource exhaustion vectors.
* **High-Fidelity Preview Rendering:** Supports hot-module replacement (HMR) and WebSockets seamlessly through the proxy layer.

## Infrastructure Requirements
* Node.js v18.0.0 or higher
* Docker Engine v20.10+ or Podman equivalent
* Reverse proxy infrastructure (Nginx, Traefik, or HAProxy) configured for wildcard subdomain routing

## Installation & Deployment
1. **Clone the repository:**
2. 
```bash
git clone https://github.com/Xer0bit/Vite-Server-Build-Service.git
cd Vite-Server-Build-Service

```

2. **Install core dependencies:**

```bash
   npm install --omit=dev

```

3. **Environment Configuration:**

```bash
   cp .env.example .env

```

Configure the required variables:

* `DOCKER_HOST_URI`: URI of the container daemon.
* `PREVIEW_DOMAIN_BASE`: Root domain for dynamic subdomain generation.
* `INSTANCE_MEMORY_LIMIT`: Hard memory limit per container (e.g., `512m`).
* `INSTANCE_TTL_SECONDS`: Maximum idle duration before automated teardown.

4. **Service Initialization:**

```bash
   npm run start

```

## API Specification

### Provision Sandbox Environment

`POST /api/v1/sandbox/provision`

**Request Payload:**

```json
{
  "framework": "react",
  "dependencies": {
    "axios": "^1.6.0"
  },
  "files": {
    "src/App.jsx": "export default function App() { return <h1>Secure Sandbox Preview</h1>; }"
  }
}

```

**Response Payload (201 Created):**

```json
{
  "status": "provisioned",
  "instanceId": "sbx-8f72a9b3",
  "previewUrl": "[https://sbx-8f72a9b3.sandbox.domain.com](https://sbx-8f72a9b3.sandbox.domain.com)",
  "hmrWebSocket": "wss://[sbx-8f72a9b3.sandbox.domain.com/_hmr](https://sbx-8f72a9b3.sandbox.domain.com/_hmr)"
}

```

### Terminate Sandbox Environment

`DELETE /api/v1/sandbox/:instanceId`

**Response Payload (200 OK):**

```json
{
  "status": "terminated",
  "instanceId": "sbx-8f72a9b3",
  "resourcesReclaimed": true
}

```

## Security Posture

* **Privilege De-escalation:** All sandbox processes execute under an unprivileged system user account (`uid 1000`). Root execution is explicitly disabled.
* **Storage Isolation:** User modifications are written to temporary, instance-specific volumes that are purged upon instance termination.
* **Network Constraints:** Internal container networks are isolated from the host network. Egress traffic to internal subnets is dropped at the virtual bridge layer to prevent Server-Side Request Forgery (SSRF) vulnerabilities.

## License

Licensed under the [MIT License](https://www.google.com/search?q=LICENSE).

