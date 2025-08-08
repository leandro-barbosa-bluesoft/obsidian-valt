Editar settings.xml da pasta .m2 do usuário:

<?xml version="1.0" encoding="UTF-8"?>
`<settings>`
   `<servers>`
      `<server>`
         `<id>nexus.releases</id>`
         `<username>admin</username>`
         `<password>admin123</password>`
      `</server>`
      `<server>`
         `<id>nexus.snapshots</id>`
         `<username>admin</username>`
         `<password>admin123</password>`
      `</server>`
      `<server>`
         `<id>nexus.snapshot</id>`
         `<username>admin</username>`
         `<password>admin123</password>`
      `</server>`
      `<server>`
         `<id>archetype</id>`
         `<username>admin</username>`
         `<password>admin123</password>`
      `</server>`
   `</servers>`
   `<mirrors>`
      `<mirror>`
         `<id>nexus</id>`
         `<url>https://nexus.bluesoft.com.br/content/groups/public/</url>`
         `<mirrorOf>*</mirrorOf>`
      `</mirror>`
   `</mirrors>`
   `<profiles>`
      `<profile>`
         `<id>nexus</id>`
         `<!--Enable snapshots for the built in central repo to direct -->`
         `<!--all requests to nexus via the mirror -->`
         `<repositories>`
            `<repository>`
               `<id>central</id>`
               `<url>http://central</url>`
               `<releases>`
                  `<enabled>true</enabled>`
               `</releases>`
               `<snapshots>`
                  `<enabled>true</enabled>`
               `</snapshots>`
            `</repository>`
         `</repositories>`
         `<pluginRepositories>`
            `<pluginRepository>`
               `<id>central</id>`
               `<url>http://central</url>`
               `<releases>`
                  `<enabled>true</enabled>`
               `</releases>`
               `<snapshots>`
                  `<enabled>true</enabled>`
               `</snapshots>`
            `</pluginRepository>`
         `</pluginRepositories>`
      `</profile>`
   `</profiles>`
   `<activeProfiles>`
      `<!--make the profile active all the time -->`
      `<activeProfile>nexus</activeProfile>`
   `</activeProfiles>`
`</settings>`

#### 1. `<servers>`

Esta seção é usada para configurar credenciais (usuário e senha) para servidores de repositório. É uma boa prática de segurança, pois evita que senhas fiquem expostas diretamente nos arquivos `pom.xml` do projeto, que são compartilhados com toda a equipe.

`<servers>`
   `<server>`
      `<id>nexus.releases</id>`
      `<username>{ommited}</username>`
      `<password>{ommited}</password>`
   `</server>`
`</servers>`

- **`<server>`**: Define as credenciais para um servidor específico.
  
- **`<id>`**: Este é o identificador único do servidor. **É crucial** que este `id` seja exatamente o mesmo que o `id` do repositório (`<repository>`) definido no seu `pom.xml` (geralmente na seção `<distributionManagement>`) para o qual você precisa se autenticar para fazer deploy (envio) de artefatos.
  
- **No seu caso**: Você tem credenciais para quatro repositórios no Nexus:
    - `nexus.releases`: Para publicar versões de lançamento (releases).
    - `nexus.snapshots` / `nexus.snapshot`: Para publicar versões de desenvolvimento (snapshots).
    - `archetype`: Para um repositório de archetypes (modelos de projeto).

#### 2. `<mirrors>`

A seção de "espelhos" (mirrors) é uma das mais importantes da sua configuração. Ela redireciona os pedidos de download de dependências do Maven.

`<mirrors>`
   `<mirror>`
      `<id>nexus</id>`
      `<url>https://nexus.bluesoft.com.br/content/groups/public/</url>`
      `<mirrorOf>*</mirrorOf>`
   `</mirror>`
`</mirrors>`

- **`<mirror>`**: Define um espelho.
- **`<id>`**: Um nome para o espelho.
- **`<url>`**: A URL para onde todas as requisições serão redirecionadas. No seu caso, é o Nexus da Bluesoft.
    
- **`<mirrorOf>`**: Esta é a tag mais poderosa aqui. Ela define _quais_ repositórios este espelho irá substituir.
    
    - O valor **`*`** (asterisco) significa que este espelho irá substituir **TODOS** os repositórios. Qualquer dependência que o Maven tentar baixar (seja do repositório central padrão, do JBoss, etc.) será interceptada e o pedido será feito para a URL do seu Nexus (`https://nexus.bluesoft.com.br/...`).
        

**Objetivo disso:** Forçar todo o tráfego de dependências a passar pelo Nexus da empresa. Isso traz vantagens como:

1. **Cache:** O Nexus armazena uma cópia local (cache) das dependências baixadas. O próximo desenvolvedor que precisar da mesma dependência a baixará diretamente do Nexus (muito mais rápido) em vez da internet.

2. **Controle:** A empresa pode controlar e auditar quais bibliotecas de terceiros são usadas nos projetos.

3. **Disponibilidade:** Se o repositório central do Maven ficar fora do ar, o Nexus da empresa ainda terá as dependências em cache, permitindo que o desenvolvimento continue.


#### 3. `<profiles>`

Perfis (`profiles`) permitem que você customize a configuração do build para diferentes ambientes (desenvolvimento, teste, produção).

`<profiles>`
   `<profile>`
      `<id>nexus</id>`
      `<repositories>`
               `</repositories>`
      `<pluginRepositories>`
               `</pluginRepositories>`
   `</profile>`
`</profiles>`

Você tem um perfil com `id="nexus"`. Dentro dele, há uma configuração um pouco sutil, mas inteligente:

- As seções `<repositories>` e `<pluginRepositories>` estão redefinindo o repositório com `id="central"`. Por padrão, o repositório `central` do Maven vem configurado para permitir apenas o download de versões `release`. A sua configuração habilita também as versões `snapshot` (`<enabled>true</enabled>`).

**Por que isso é feito?** Para que o seu mirror (`<mirrorOf>*</mirrorOf>`) funcione corretamente para **todas** as requisições. Se um projeto pedir uma dependência `SNAPSHOT` do repositório `central`, e o `central` não tiver snapshots habilitados por padrão, o Maven nem tentaria fazer o download. Ao habilitar snapshots aqui, você garante que o Maven tente o download, permitindo que o seu _mirror_ intercepte a requisição e a redirecione para o Nexus.

#### 4. `<activeProfiles>`

Esta seção simplesmente ativa um ou mais perfis por padrão, sem que você precise especificá-los na linha de comando (com a flag `-P`).

`<activeProfiles>`
   `<activeProfile>nexus</activeProfile>`
`</activeProfiles>`

- **`<activeProfile>nexus</activeProfile>`**: Isso garante que o perfil com `id="nexus"` esteja **sempre ativo** para qualquer build que você rodar. Isso torna a configuração do repositório e do mirror transparente para o desenvolvedor.


---

### Diferença para a Configuração Padrão/Normal

Uma instalação "limpa" do Maven, sem um `settings.xml` de usuário, se comporta da seguinte forma:

1. **Sem Mirror Centralizado:** Todas as dependências são baixadas diretamente do repositório central oficial do Maven (`https://repo.maven.apache.org/maven2/`). Não há um `<mirrorOf>*</mirrorOf>`.

2. **Downloads Mais Lentos:** Cada desenvolvedor baixa as dependências da internet. Não há um cache centralizado para a equipe, o que torna os builds iniciais mais lentos.

3. **Sem Credenciais Pré-configuradas:** A seção `<servers>` estaria vazia. Se você precisasse fazer deploy para um repositório privado, teria que adicionar as credenciais manualmente.

4. **Sem Perfis Ativos por Padrão:** A seção `<activeProfiles>` estaria vazia.

5. **Dependência da Internet:** Se o repositório central do Maven estiver lento ou indisponível, seus builds podem falhar ou demorar muito.