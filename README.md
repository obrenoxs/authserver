# Auth Server

Servidor de autorização **OAuth2 / OpenID Connect** construído com **Spring Boot** e **Spring Authorization Server**, com usuários, clientes e chaves mantidos em memória. Projeto de estudo desenvolvido durante o curso **Java Spring Tópicos Avançados** (Coleção Java Spring), do professor Nelio Alves.

## Sobre o projeto

O objetivo é entender na prática como funciona o fluxo **Authorization Code** com OIDC, emitindo tokens JWT assinados com RSA para uma aplicação cliente (Next.js com Auth.js). Por ser um projeto didático, tudo é configurado em memória, sem banco de dados.

## Funcionalidades

- Fluxo **Authorization Code** com suporte a **Refresh Token**
- **OpenID Connect** habilitado (ID Token, `/userinfo` e discovery)
- Tokens **JWT** assinados com chave **RSA 2048** gerada na inicialização
- Tela de login padrão (form login) com redirecionamento automático para navegadores
- Cliente registrado para uma aplicação **Next.js (Auth.js)**
- Senhas dos usuários codificadas com `DelegatingPasswordEncoder`
- Perfil da aplicação configurável via variável de ambiente

## Tecnologias

| Tecnologia | Versão |
|---|---|
| Java | 25 |
| Spring Boot | 4.1.2 |
| Spring Security | incluso no Boot |
| Spring Authorization Server | via `spring-boot-starter-security-oauth2-authorization-server` |
| Maven | wrapper do projeto |

## Arquitetura da configuração

A classe `AuthorizationServerConfig` concentra toda a configuração:

| Componente | Responsabilidade |
|---|---|
| `authorizationServerSecurityFilterChain` (`@Order(1)`) | Protege os endpoints do Authorization Server e habilita OIDC |
| `defaultSecurityFilterChain` (`@Order(2)`) | Protege as demais rotas e fornece o formulário de login |
| `userDetailsService` | Usuários em memória |
| `registeredClientRepository` | Clientes OAuth2 registrados em memória |
| `jwkSource` / `jwtDecoder` | Chave RSA e decodificação dos JWTs |
| `authorizationServerSettings` | Define o issuer do servidor |

## Como executar

### Pré-requisitos

- JDK 25
- Maven (ou o wrapper `mvnw` do projeto)

### Passos

```bash
# Clone o repositório
git clone https://github.com/obrenoxs/authserver.git
cd authserver

# Execute a aplicação
./mvnw spring-boot:run
```

O servidor sobe em **http://localhost:9000**.

### Configuração

`application.properties`:

```properties
spring.profiles.active=${APP_PROFILE:dev}
server.port=9000
```

| Variável | Padrão | Descrição |
|---|---|---|
| `APP_PROFILE` | `dev` | Perfil ativo do Spring |

## Credenciais de desenvolvimento

> **Atenção:** credenciais fixas apenas para estudo. Nunca use em produção.

**Usuários**

| Usuário | Senha |
|---|---|
| `maria@example.com` | `12345678` |
| `alex@example.com` | `12345678` |

**Cliente OAuth2**

| Campo | Valor |
|---|---|
| Client ID | `nextjs-client` |
| Client Secret | `nextjs-secret` |
| Autenticação do cliente | `client_secret_basic` e `client_secret_post` |
| Grant types | `authorization_code`, `refresh_token` |
| Scope | `openid` |
| Consentimento | não exigido |
| Redirect URIs | `http://localhost:3000/api/auth/callback/spring`, `http://localhost:9000/authorized` |
| Post logout redirect | `http://localhost:3000` |

## Endpoints principais

| Endpoint | Descrição |
|---|---|
| `GET /.well-known/openid-configuration` | Metadados OIDC (discovery) |
| `GET /oauth2/authorize` | Início do fluxo Authorization Code |
| `POST /oauth2/token` | Troca do código por tokens / refresh |
| `GET /oauth2/jwks` | Chaves públicas para validar os JWTs |
| `GET /userinfo` | Informações do usuário autenticado |
| `GET /login` | Tela de login |

## Testando o fluxo manualmente

1. Abra no navegador:

```
http://localhost:9000/oauth2/authorize?response_type=code&client_id=nextjs-client&scope=openid&redirect_uri=http://localhost:9000/authorized
```

2. Faça login com um dos usuários de desenvolvimento.
3. Copie o parâmetro `code` da URL de redirecionamento.
4. Troque o código por tokens:

```bash
curl -X POST http://localhost:9000/oauth2/token \
  -u nextjs-client:nextjs-secret \
  -d "grant_type=authorization_code" \
  -d "code=SEU_CODE_AQUI" \
  -d "redirect_uri=http://localhost:9000/authorized"
```

A resposta traz o `access_token`, o `id_token` (por causa do scope `openid`) e o `refresh_token`.

## Integração com Next.js (Auth.js)

Exemplo de provider OIDC apontando para este servidor:

```ts
{
  id: "spring",
  name: "Spring Auth Server",
  type: "oidc",
  issuer: "http://localhost:9000",
  clientId: "nextjs-client",
  clientSecret: "nextjs-secret",
}
```

## Limitações conhecidas

- A **chave RSA é regenerada a cada inicialização**, o que invalida os tokens emitidos antes de um restart.
- Usuários, clientes e autorizações ficam **em memória** e são perdidos ao reiniciar.
- Segredos e senhas estão no código-fonte, apenas para fins didáticos.

## Próximos passos

- Persistir usuários e clientes em banco de dados (JPA)
- Persistir autorizações e consentimentos
- Carregar a chave RSA de um keystore ou de variáveis de ambiente
- Externalizar segredos e usar `{bcrypt}` para o secret do cliente
- Adicionar roles/authorities e claims customizadas ao token
- Escrever testes de integração do fluxo OAuth2

## Autor

**Breno Oliveira de Souza**
GitHub: [@obrenoxs](https://github.com/obrenoxs)

Projeto desenvolvido junto com o professor **Nelio Alves** no curso Java Spring Tópicos Avançados (DevSuperior).
