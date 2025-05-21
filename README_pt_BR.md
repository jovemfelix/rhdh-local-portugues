# 🚀 Teste Localmente com o Red Hat Developer Hub (RHDH)

O RHDH Local permite que engenheiros de plataforma testem rapidamente:
- Catálogos de software
- Documentos técnicos 
- Plugins e modelos
- Personalizações de homepage
- Configurações do RHDH

**Sem a necessidade de um cluster Kubernetes!**

> ⚠️ **Avisos Importantes**  
> - 🚫 **NÃO** é um substituto para o Red Hat Developer Hub oficial  
> - 🏭 **NÃO** adequado para ambientes de produção  
> - 👥 **NÃO** suporta trabalho em equipe (sem RBAC)  
> - 🔧 **NÃO** tem suporte oficial - use por sua conta e risco  
> - 💡 Extremamente útil para desenvolvimento individual  

## 📋 Sumário
- [Requisitos](#-prerequisites)
- [Inicio Rápido](#-quick-start)
- [Gestão de Configuração](#-configuration-management)
- [Plugins Dinâmicos](#-dynamic-plugins)
- [Configurações Avançadas](#-advanced-configurations)
- [Limpeza](#-cleanup)
- [Suporte e Contribuições](#-support--contributions)
- [Licença](#-license)

## 📋 Pré-requisitos

| Requisito               | Obrigatório? | Detalhes                          |
|-------------------------|--------------|-----------------------------------|
| PC x86 64-bit           | Sim          | Suporte ARM em breve              |
| Docker/Podman           | Sim          | Com recursos adequados            |
| Conexão com Internet    | Sim          | Para baixar imagens e plugins     |
| Git                     | Opcional     | Para clonar o repositório         |
| Conta GitHub            | Opcional     | Para integração com GitHub        |
| Node.js (npx)           | Opcional     | Para autenticação GitHub          |
| Conta Red Hat           | Opcional     | Para usar PostgreSQL              |

## 🚀 Início Rápido

1. **Clone o repositório**:
   ```bash
   git clone https://github.com/redhat-developer/rhdh-local.git
   cd rhdh-local
   ```

2. **Configuração para Mac ARM (M1/M2)**:
   ```bash
   echo 'RHDH_IMAGE=quay.io/rhdh-community/rhdh:next' >> .env
   ```

3. **Personalize sua configuração (opcional)**:
   ```bash
   cp configs/app-config/app-config.local.example.yaml configs/app-config/app-config.local.yaml
   cp configs/dynamic-plugins/dynamic-plugins.override.example.yaml configs/dynamic-plugins/dynamic-plugins.override.yaml
   ```

4. **Inicie os contêineres**:
   ```bash
   # Com Docker
   docker compose up -d
   
   # Ou com Podman
   podman-compose up -d
   ```

5. **Acesse a interface**:
   Abra [http://localhost:7007](http://localhost:7007) no seu navegador

## 🔧 Gerenciamento de Configurações

### Atualizando Configurações do App
```bash
podman-compose stop rhdh && podman-compose start rhdh
```

### Atualizando Plugins
```bash
podman-compose run install-dynamic-plugins
podman-compose stop rhdh && podman-compose start rhdh
```

## 🧩 Plugins Dinâmicos

Para adicionar plugins locais:

1. Copie os arquivos para `local-plugins/`
2. Configure as permissões:
   ```bash
   chmod -R 777 local-plugins
   ```
3. Edite o arquivo de configuração apropriado:
   - `configs/dynamic-plugins/dynamic-plugins.override.yaml` (preferencial)
   - `configs/dynamic-plugins/dynamic-plugins.yaml`

<details>
<summary>📄 Exemplo de dynamic-plugins.override.yaml</summary>

```yaml
includes:
  - dynamic-plugins.default.yaml
# Suas configurações adicionais aqui
```
</details>

## ⚙️ Configurações Avançadas

### 🔄 Alternando Imagens de Contêiner
Edite seu `.env`:
```bash
# Para versão comunitária
RHDH_IMAGE=quay.io/rhdh-community/rhdh:next

# Para versão oficial
RHDH_IMAGE=quay.io/rhdh/rhdh-hub-rhel9:1.y
```

### 🗃️ Usando PostgreSQL
1. Efetue login no registry:
   ```bash
   podman login registry.redhat.io
   ```
2. Descomente a seção `db` no `compose.yaml`
3. Comente a configuração SQLite no `app-config.local.yaml`

### 🕵️ Depuração com VSCode
1. Inicie em modo debug:
   ```bash
   podman-compose -f compose.yaml -f compose-debug.yaml up
   ```
2. Configure o `launch.json`:
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

## 🧹 Limpeza
Para remover tudo:
```bash
# Remover contêineres e volumes
podman-compose down --volumes

# Limpar completamente (Docker)
docker system prune --volumes

# Limpar completamente (Podman)
podman system prune --volumes
```

## ❓ Suporte e Contribuições

Para reportar problemas:
- [JIRA](https://issues.redhat.com/browse/RHIDP) (Componente: RHDH Local)

Contribuições são bem-vindas!

## 📜 Licença
```text
Copyright Red Hat

Licenciado sob a Licença Apache, Versão 2.0 (a "Licença");
você não pode usar este arquivo, exceto em conformidade com a Licença.
Você pode obter uma cópia da Licença em

http://www.apache.org/licenses/LICENSE-2.0
```
