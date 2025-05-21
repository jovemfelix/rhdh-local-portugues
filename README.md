# 🚀 Test Locally with Red Hat Developer Hub (RHDH)

RHDH Local allows platform engineers to quickly test:
- Software catalogs
- Technical documentation
- Plugins and templates
- Homepage customizations
- RHDH configurations

**No Kubernetes cluster needed!**

> ⚠️ **Important Notes**  
> - 🚫 **NOT** a replacement for official Red Hat Developer Hub  
> - 🏭 **NOT** suitable for production environments  
> - 👥 **NOT** team-ready (no RBAC)  
> - 🔧 **NO** official support - use at your own risk  
> - 💡 Extremely useful for individual development  

## 📋 Table of Contents
- [Prerequisites](#-prerequisites)
- [Quick Start](#-quick-start)
- [Configuration Management](#-configuration-management)
- [Dynamic Plugins](#-dynamic-plugins)
- [Advanced Configurations](#-advanced-configurations)
- [Cleanup](#-cleanup)
- [Support & Contributions](#-support--contributions)
- [License](#-license)

## 📋 Prerequisites

| Requirement            | Mandatory? | Details                         |
|------------------------|------------|---------------------------------|
| x86 64-bit PC          | Yes        | ARM support coming soon         |
| Docker/Podman          | Yes        | With adequate resources         |
| Internet connection    | Yes        | To download images/plugins      |
| Git                    | Optional   | To clone repository             |
| GitHub account         | Optional   | For GitHub integration          |
| Node.js (npx)          | Optional   | For GitHub authentication       |
| Red Hat account        | Optional   | For PostgreSQL usage            |

## 🚀 Quick Start

1. **Clone the repository**:
   ```bash
   git clone https://github.com/redhat-developer/rhdh-local.git
   cd rhdh-local
   ```

2. **ARM Mac (M1/M2) configuration**:
   ```bash
   echo 'RHDH_IMAGE=quay.io/rhdh-community/rhdh:next' >> .env
   ```

3. **Customize your configuration (optional)**:
   ```bash
   cp configs/app-config/app-config.local.example.yaml configs/app-config/app-config.local.yaml
   cp configs/dynamic-plugins/dynamic-plugins.override.example.yaml configs/dynamic-plugins/dynamic-plugins.override.yaml
   ```

4. **Start containers**:
   ```bash
   # With Docker
   docker compose up -d
   
   # Or with Podman
   podman-compose up -d
   ```

5. **Access the interface**:
   Open [http://localhost:7007](http://localhost:7007) in your browser

## 🔧 Configuration Management

### Updating App Config
```bash
podman-compose stop rhdh && podman-compose start rhdh
```

### Updating Plugins
```bash
podman-compose run install-dynamic-plugins
podman-compose stop rhdh && podman-compose start rhdh
```

## 🧩 Dynamic Plugins

To add local plugins:

1. Copy files to `local-plugins/`
2. Set permissions:
   ```bash
   chmod -R 777 local-plugins
   ```
3. Edit the appropriate config file:
   - `configs/dynamic-plugins/dynamic-plugins.override.yaml` (preferred)
   - `configs/dynamic-plugins/dynamic-plugins.yaml`

<details>
<summary>📄 dynamic-plugins.override.yaml example</summary>

```yaml
includes:
  - dynamic-plugins.default.yaml
# Your additional configurations here
```
</details>

## ⚙️ Advanced Configurations

### 🔄 Switching Container Images
Edit your `.env`:
```bash
# For community version
RHDH_IMAGE=quay.io/rhdh-community/rhdh:next

# For official version
RHDH_IMAGE=quay.io/rhdh/rhdh-hub-rhel9:1.y
```

### 🗃️ Using PostgreSQL
1. Login to registry:
   ```bash
   podman login registry.redhat.io
   ```
2. Uncomment `db` section in `compose.yaml`
3. Comment SQLite config in `app-config.local.yaml`

### 🕵️ Debugging with VSCode
1. Start in debug mode:
   ```bash
   podman-compose -f compose.yaml -f compose-debug.yaml up
   ```
2. Configure `launch.json`:
   ```json
   {
     "version": "0.2.0",
     "configurations": [
       {
         "name": "Attach to RHDH",
         "type": "node",
         "request": "attach",
         "port": 9229,
         "localRoot": "${workspaceFolder}",
         "remoteRoot": "/opt/app-root/src/dynamic-plugins-root/plugin-name"
       }
     ]
   }
   ```

## 🧹 Cleanup
To remove everything:
```bash
# Remove containers and volumes
podman-compose down --volumes

# Full cleanup (Docker)
docker system prune --volumes

# Full cleanup (Podman)
podman system prune --volumes
```

## ❓ Support & Contributions

To report issues:
- [JIRA](https://issues.redhat.com/browse/RHIDP) (Component: RHDH Local)

Contributions are welcome!

## 📜 License
```text
Copyright Red Hat

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0
```
