# Teste Localmente com o Red Hat Developer Hub (RHDH)

- Bem-vindo ao RHDH Local!
- A maneira mais rápida e simples para engenheiros de plataforma testarem seus catálogos de software, documentos técnicos, plugins, modelos, personalizações de homepage, configurações e muito mais com o RHDH!
- O RHDH Local é ideal para testar os recursos básicos do RHDH (como Catálogos de Software ou Documentos Técnicos) sem a necessidade de um cluster Kubernetes. 
- O RHDH Local também é ótimo para testar plugins dinâmicos e suas configurações. 
- Para usar o RHDH Local, tudo o que você precisa é de conhecimento básico de Docker ou Podman, um PC e um navegador web. 
- Você pode executá-lo em seu laptop, desktop ou em seu homelab. 
- Melhor ainda, quando terminar de trabalhar, é fácil de remover.

> - **O RHDH Local NÃO substitui o Red Hat Developer Hub**.
> - Não tente usar o RHDH Local como um sistema de produção.
> - O RHDH Local foi projetado para ajudar desenvolvedores individuais a testar diversos recursos do RHDH.
> - Ele não foi projetado para escalar e **não** é adequado para uso em equipes (**não** há RBAC, por exemplo). 
> - Também **não** há (atualmente) suporte _oficial_ para o RHDH Local.
> - Você usa o RHDH Local por sua conta e risco.
> - Dito isso, achamos que ele é incrivelmente útil e qualquer contribuição que você possa ter para melhorar o RHDH Local é bem-vinda!

## O que você precisa antes de começar

Para usar o RHDH Local, você precisará de alguns itens:

1. Um PC com arquitetura x86 64 bits (amd64) (ARM em breve)
1. Uma instalação do Docker ou Podman (com os recursos adequados disponíveis)
1. Uma conexão com a internet (para baixar imagens de contêiner, plugins, etc.)
1. (_Opcional_) O cliente de linha de comando `git` para clonar este repositório (ou você pode baixar e extrair o arquivo ZIP do GitHub)
1. (_Opcional_) Uma conta no GitHub (se desejar integrar os recursos do GitHub ao RHDH)
1. (_Opcional_) A ferramenta node `npx` (se desejar usar a autenticação do GitHub no RHDH)
1. (_Opcional_) Uma [conta Red Hat](https://access.redhat.com/RegistryAuthentication#getting-a-red-hat-login-2) (se desejar usar um banco de dados PostgreSQL)

### Observação para usuários de Mac M1

- Se você estiver usando um Mac Apple Silicon (M1/M2), a imagem RHDH padrão (`quay.io/rhdh/rhdh-hub-rhel9:1.4`) **não** é compatível com a arquitetura ARM64.
- Para corrigir isso, você pode adicionar esta linha ao seu arquivo `.env` (crie o arquivo se ele não existir):

`RHDH_IMAGE=quay.io/rhdh-community/rhdh:next`

- Esta imagem suporta `amd64` e `arm64`.

## Introdução ao RHDH Local

1. Clone este repositório para um local no seu PC

   ```sh
   git clone https://github.com/redhat-developer/rhdh-local.git
   ```

1. Vá para a pasta `rhdh-local`.

   ```sh
   cd rhdh-local
   ```

1. (_Opcional_) Você pode criar um arquivo local `.env` e substituir qualquer uma das variáveis padrão definidas no arquivo [`default.env`](./default.env) fornecido. Você também pode adicionar variáveis adicionais. Na maioria dos casos, quando você **não** precisa do GitHub Auth ou de testar versões diferentes, pode pular esta etapa, e deve funcionar.

1. (_Opcional_) Faça a substituição da configuração local. O RHDH Local suporta substituições de configuração específicas do usuário usando um diretório estruturado `configs/`. Você **não** precisa modificar os arquivos padrão. No entanto, se quiser personalizar sua configuração:

   - Adicione as substituições de configuração do seu aplicativo a: `configs/app-config/app-config.local.yaml`
      > Você pode usar os arquivos `.example.yaml` incluídos para começar rapidamente:
      >
      > ```sh
      > cp configs/app-config/app-config.local.example.yaml configs/app-config/app-config.local.yaml
      > cp configs/dynamic-plugins/dynamic-plugins.override.example.yaml configs/dynamic-plugins/dynamic-plugins.override.yaml
      > ```

   - Adicione suas as configurações a serem sobrescritas do plugin a:
     `configs/dynamic-plugins/dynamic-plugins.override.yaml`
     > O arquivo de substituição deve começar com:
     > ```yaml
     > includes:
     >   - dynamic-plugins.default.yaml
     > ```
     > Isso garante que a lista de plugins base seja preservada e estendida, em vez de substituída.

   - Adicione quaisquer arquivos extras (como credenciais do GitHub) para: `configs/extra-files/`

   > Se presentes, esses arquivos serão carregados automaticamente pelo sistema na inicialização.

   Se precisar de recursos que busquem arquivos do GitHub, configure `integrations.github`.
   A maneira recomendada é usar o GitHub Apps. Você pode encontrar dicas sobre como configurá-lo em [github-app-credentials.example.yaml](configs/github-app-credentials.example.yaml) ou instruções mais detalhadas na [documentação do Backstage](https://backstage.io/docs/integrations/github/github-apps).

2. Inicie o RHDH Local.
   Este repositório deve funcionar com `docker compose` usando o Docker Engine ou `podman-compose` usando o Podman. Ao usar o Podman, há algumas exceções. Consulte [Problemas conhecidos ao usar o Podman Compose](#known-issues-when-using-podman-compose) para mais informações.

   ```sh
   podman-compose up -d
   ```

   Se preferir `docker compose`, você pode simplesmente substituir `podman-compose` por `docker compose`

   ```sh
   docker compose up -d
   ```

3. Abra [http://localhost:7007](http://localhost:7007) no seu navegador para acessar o RHDH.

## Alterando sua configuração

Ao alterar `app-config.local.yaml`, você deve reiniciar o contêiner `rhdh` para carregar a configuração atualizada do RHDH.

```sh
podman-compose stop rhdh && podman-compose start rhdh
```

Ao alterar `dynamic-plugins.yaml`, você precisa executar novamente o contêiner `install-dynamic-plugins` e reiniciar a instância do RHDH.

```sh
podman-compose run install-dynamic-plugins
podman-compose stop rhdh && podman-compose start rhdh
```

## Carregando plugins dinâmicos de um diretório local

Durante a inicialização, o contêiner `install-dynamic-plugins` lê o conteúdo do arquivo de configuração do plugin e ativa, configura ou baixa quaisquer plugins listados. O RHDH Local suporta duas maneiras de especificar a configuração dinâmica de plugins:

1. Caminho **padrão**: `configs/dynamic-plugins/dynamic-plugins.yaml`

1. Caminho de **substituição** do usuário: `configs/dynamic-plugins/dynamic-plugins.override.yaml` ou `configs/dynamic-plugins.yaml`. Se presente, este arquivo substituirá automaticamente o padrão e será usado pelo contêiner `install-dynamic-plugins`. `configs/dynamic-plugins/dynamic-plugins.override.yaml` tem precedência sobre `configs/dynamic-plugins.yaml`.

Além disso, o diretório `local-plugins` é montado no contêiner `install-dynamic-plugins` em `/opt/app-root/src/local-plugins`. Quaisquer plugins instalados lá podem ser ativados/configurados da mesma forma (sem download).

Para carregar plugins dinâmicos da sua máquina local:

1. Copie o arquivo binário do plugin dinâmico para o diretório `local-plugins`.
2. Certifique-se de que as permissões permitam que o contêiner leia os arquivos (por exemplo, `chmod -R 777 local-plugins` para testes rápidos).
3. Configure seu plugin em um dos arquivos de configuração suportados:
- Prefira `configs/dynamic-plugins/dynamic-plugins.override.yaml` para substituições de usuários locais.
- Se nenhum arquivo de substituição estiver presente, `configs/dynamic-plugins/dynamic-plugins.yaml` será usado.
4. Consulte [Alterando sua configuração](#change-your-configuration) para obter mais informações sobre como atualizar e recarregar configurações.

## Opcional: Personalize `.npmrc` para instalação de plugins

Se estiver instalando plugins dinâmicos a partir de um registro privado ou usando um proxy, você pode personalizar seu próprio arquivo `.npmrc`. Um arquivo `.npmrc.example` é fornecido no diretório `configs/` como modelo.

1. Copie o arquivo de exemplo para criar seu próprio `.npmrc`:

```sh
cp configs/.npmrc.example configs/.npmrc
```

2. Abra o arquivo `.npmrc` recém-criado e adicione sua configuração, como URLs de registro privado ou tokens de autenticação:

    ```sh
    //registry.npmjs.org/:_authToken=YOUR_TOKEN
    registry=https://your-private-registry.example.com/
    ```

Quando presente, este arquivo `.npmrc` será montado automaticamente no contêiner `install-dynamic-plugins` e a variável de ambiente `NPM_CONFIG_USERCONFIG` será definida para apontar para ele.

Se você não criar um `.npmrc`, a instalação do plugin ainda funcionará usando as configurações padrão do registro público.

> Para obter mais informações sobre como configurar `.npmrc`, consulte a [documentação de configuração do npm](https://docs.npmjs.com/cli/v10/configuring-npm/npmrc).

## Alterando a Imagem do Contêiner

Você pode alternar entre as versões [downstream](https://quay.io/repository/rhdh/rhdh-hub-rhel9?tab=tags) e [community](https://quay.io/repository/rhdh-community/rhdh?tab=tags) do RHDH alterando o nome da imagem do contêiner mantido pela variável de ambiente `RHDH_IMAGE` no seu arquivo `.env`.

Por exemplo, para usar a compilação noturna da comunidade, defina a variável da seguinte forma:

```sh
RHDH_IMAGE=quay.io/rhdh-community/rhdh:next
```

Para usar a versão oficial do RHDH 1.y, defina a variável da seguinte forma (substitua `y` conforme necessário):

```sh
RHDH_IMAGE=quay.io/rhdh/rhdh-hub-rhel9:1.y
```

## Testando o RHDH em uma configuração simulada de proxy corporativo

Se quiser testar como o RHDH se comportaria se implantado em um ambiente de proxy corporativo,
você pode executar `podman-compose` ou `docker-compose` mesclando os arquivos [`compose.yaml`](./compose.yaml) e [`compose-with-corporate-proxy.yaml`](./compose-with-corporate-proxy.yaml).

Exemplo com `podman-compose` (observe que a ordem dos arquivos YAML é importante):

```sh
podman-compose \
   -f compose.yaml \
   -f compose-with-corporate-proxy.yaml \
   up -d
```

O arquivo [`compose-with-corporate-proxy.yaml`](compose-with-corporate-proxy.yaml) inclui um contêiner proxy específico baseado em [Squid](https://www.squid-cache.org/), bem como uma rede isolada, de forma que:

1. somente o contêiner proxy tem acesso ao exterior
2. todos os contêineres que fazem parte da rede interna precisam se comunicar através do contêiner proxy para alcançar o exterior. Isso pode ser feito com as variáveis de ambiente `HTTP(S)_PROXY` e `NO_PROXY`.

## Limpeza

Para redefinir o RHDH Local, você pode usar o seguinte comando. Isso limpará todos os volumes anexados ao contêiner, mas as alterações de configuração feitas nos seus arquivos YAML `rhdh-local` permanecerão.

```sh
podman-compose down --volumes
```

Para redefinir tudo no repositório `rhdh-local` clonado, incluindo quaisquer alterações de configuração feitas nos seus arquivos YAML, tente:

```sh
git reset --hard
```

Para remover completamente os contêineres RHDH do seu sistema (após executar `compose down`):

```sh
docker system prune --volumes # For rhdh-local running on docker
podman system prune --volumes # For rhdh-local running on podman
```

### Problemas conhecidos ao usar o Podman Compose

Funciona com `podman-compose` apenas com imagens que incluem a seguinte correção: https://github.com/redhat-developer/rhdh/pull/1585

Imagens mais antigas não funcionam em combinação com `podman-compose`.
Isso ocorre devido a https://issues.redhat.com/browse/RHIDP-3939. As imagens RHDH atualmente preenchem o diretório dynamic-plugins-root com todos os plugins empacotados dentro da imagem.
Antes de o podman montar o volume sobre o diretório `dynamic-plugins-root`, ele copia todos os arquivos existentes para o volume. Quando os plugins são instalados usando o script `install-dynamic-plugins.sh`, ele cria instalações duplicadas de alguns plugins, o que impede o Backstage de iniciar.

Isso também não funciona com `podman compose` ao usar `docker-compose` como provedor externo de composição no macOS.

Falha com

```
install-dynamic-plugins-1  | Traceback (most recent call last):
install-dynamic-plugins-1  |   File "/opt/app-root/src/install-dynamic-plugins.py", line 429, in <module>
install-dynamic-plugins-1  |     main()
install-dynamic-plugins-1  |   File "/opt/app-root/src/install-dynamic-plugins.py", line 206, in main
install-dynamic-plugins-1  |     with open(dynamicPluginsFile, 'r') as file:
install-dynamic-plugins-1  | PermissionError: [Errno 13] Permission denied: 'dynamic-plugins.yaml'
```

Parece que `docker-compose` quando usado com podman não propaga corretamente o rótulo `Z` do SElinux.

## Usando o banco de dados PostgreSQL

> **ATENÇÃO**: Por padrão, o banco de dados em **memória** é usado.
Se você quiser usar o PostgreSQL com o RHDH, aqui estão os passos:

> **NOTA**: Você precisa ter o [Red Hat Login](https://access.redhat.com/RegistryAuthentication#getting-a-red-hat-login-2) para usar a imagem `postgresql`.

1. Efetue login no registro do contêiner com as credenciais do *Red Hat Login* para usar a imagem `postgresql`

   ```sh
   podman login registry.redhat.io
   ```

   Se preferir `docker`, basta substituir `podman` por `docker`

   ```sh
   docker login registry.redhat.io
   ```

2. Descomente o bloco de serviço `db` no arquivo [compose.yaml](compose.yaml)

   ```yaml
   db:
     image: "registry.redhat.io/rhel8/postgresql-16:latest"
     volumes:
       - "/var/lib/pgsql/data"
     env_file:
       - path: "./default.env"
         required: true
       - path: "./.env"
         required: false
     environment:
       - POSTGRESQL_ADMIN_PASSWORD=${POSTGRES_PASSWORD}
     healthcheck:
       test: ["CMD", "pg_isready", "-U", "postgres"]
       interval: 5s
       timeout: 5s
       retries: 5
   ```

3. Descomente a seção `db` na seção `depends_on` do serviço `rhdh` em [compose.yaml](compose.yaml)

   ```yaml
   depends_on:
     install-dynamic-plugins:
       condition: service_completed_successfully
     db:
       condition: service_healthy
   ```

4. Comente a configuração do SQLite na memória em [`app-config.local.yaml`](configs/app-config.local.yaml)

   ```yaml
   # database:
   #   client: better-sqlite3
   #   connection: ':memory:'
   ```

## Desenvolvedores: Usando o VSCode para depurar plugins de backend

Você pode usar o RHDH-local com um depurador para depurar seus plugins de backend no VSCode. Veja como:

1. Inicie o RHDH-local com o arquivo compose "debug".

   ```sh
   # in rhdh-local directory
   podman-compose up -f compose.yaml -f compose-debug.yaml
   ```

2. Abra o código-fonte do seu plugin no VSCode
3. Exporte o plugin para um plugin RHDH "dinâmico"

   ```sh
   # in plugin source code directory
   npx @janus-idp/cli@latest package export-dynamic-plugin
   ```

4. Copie o pacote de plugin derivado exportado para o diretório `dynamic-plugins-root` no contêiner `rhdh`.

   ```sh
   # in plugin source code directory
   podman cp dist-dynamic rhdh:/opt/app-root/src/dynamic-plugins-root/<your-plugin-name>
   ```

5. Se o seu plugin exigir configuração, adicione-o ao arquivo `app-config.local.yaml` no diretório `rhdh-local` clonado.

6. Reinicie o contêiner `rhdh`

   ```sh
   # in rhdh-local directory
   podman-compose stop rhdh
   podman-compose start rhdh
   ```

7. Configure o depurador VSCode para anexar ao contêiner `rhdh`.

   `.vscode/launch.json` example:

   ```json
   {
      "version": "0.2.0",
      "configurations": [
         {
            "name": "Attach to Process",
            "type": "node",
            "request": "attach",
            "port": 9229,
            "localRoot": "${workspaceFolder}",
            "remoteRoot": "/opt/app-root/src/dynamic-plugins-root/<your-plugin-name>",
         }
      ]
   }
   ```

8. Agora, você pode começar a depurar seu plugin usando o depurador VSCode. O mapeamento de origem deve funcionar e você deve conseguir inserir pontos de interrupção nos seus arquivos TypeScript. Se não funcionar, provavelmente você precisará ajustar os caminhos `localRoot` e `remoteRoot` em `launch.json`.

   Sempre que fizer alterações no código-fonte do seu plugin, repita os passos 3 a 6.

## Configurando Credenciais de Registro

Coloque suas credenciais de registro em `./configs/extra-files` e, em seguida, referencie o arquivo de autenticação em `.env`:

```bash
REGISTRY_AUTH_FILE=/opt/app-root/src/configs/extra-files/auth.json
```

Isso permite que o RHDH-local extraia artefatos OCI de registros como registry.redhat.io sem erros de autenticação.

## Contribuindo e relatando problemas

Para relatar problemas neste repositório, use [JIRA](https://issues.redhat.com/browse/RHIDP) com o Componente: **RHDH Local**

Para navegar pelos problemas existentes, você pode usar este [Query](https://issues.redhat.com/issues/?filter=-4&jql=project%20%3D%20%22Red%20Hat%20Internal%20Developer%20Platform%22%20%20AND%20component%20%3D%20%22RHDH%20Local%22%20AND%20resolution%20%3D%20Unresolved%20ORDER%20BY%20status%2C%20priority%2C%20updated%20%20%20%20DESC).

Contribuições são bem-vindas!

## Licença

```txt
Copyright Red Hat

Licenciado sob a Licença Apache, Versão 2.0 (a "Licença");
você não pode usar este arquivo, exceto em conformidade com a Licença.
Você pode obter uma cópia da Licença em

http://www.apache.org/licenses/LICENSE-2.0

A menos que exigido pela lei aplicável ou acordado por escrito, o software
distribuído sob a Licença é distribuído "NO ESTADO EM QUE SE ENCONTRA",
SEM GARANTIAS OU CONDIÇÕES DE QUALQUER TIPO, expressas ou implícitas.
Consulte a Licença para o texto específico que rege as permissões e
limitações sob a Licença.
```
