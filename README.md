# 🔐 Petshop Authentication Service (`petshop-auth-service`)

Este repositório contém o **Microsserviço de Autenticação** da arquitetura distribuída do Petshop. Ele é responsável por gerenciar credenciais de acesso, validar usuários e gerar tokens assinados digitalmente.

---

## 🏗️ Papel e Funcionalidade no Ecossistema

O `auth-service` atua de forma isolada na arquitetura:
1. **Segurança Centralizada**: Gera credenciais seguras para todos os microsserviços do ecossistema.
2. **SmallRye JWT / RSA**: Utiliza criptografia de chave pública/privada (RSA com chaves de 2048 bits) para assinar tokens JWT. O Kong API Gateway ou outros serviços usam apenas a chave pública para validar as requisições, sem a necessidade de consultar o microsserviço de autenticação a cada requisição (zero overhead e latência reduzida).
3. **Sem Estado (Stateless)**: O serviço não mantém sessão em memória ou banco de dados, facilitando a escalabilidade horizontal e resiliência a falhas.

---

## 🛠️ Tecnologias Principais

* **Java 21** e **Quarkus Framework**
* **Quarkus SmallRye JWT Build** (Geração segura de tokens JWT)
* **Quarkus Micrometer & Prometheus Registry** (Métricas de JVM e HTTP)
* **Maven** (Gerenciador de build e dependências)

---

## 💻 Como Rodar o Serviço Localmente

### Pré-requisitos
* Java 21 JDK instalado localmente (ou via Docker)
* Maven instalado localmente (ou use o `./mvnw` incluso)

### Executando em Modo de Desenvolvimento (Live Coding)

Para iniciar o Quarkus com recarregamento em tempo real (qualquer alteração no código é refletida instantaneamente):

```bash
./mvnw compile quarkus:dev
```

* **Porta local padrão**: `8081`
* **Painel Dev UI do Quarkus**: `http://localhost:8081/q/dev/`

### Empacotamento e Execução em Produção

Para compilar e gerar o pacote de distribuição otimizado:

```bash
./mvnw package
```
O build produzirá os arquivos compilados no diretório `target/quarkus-app/`. Para iniciar o microsserviço empacotado:

```bash
java -jar target/quarkus-app/quarkus-run.jar
```

---

## 🧪 Testes Automatizados e Cobertura

O projeto possui suíte de testes unitários e de integração utilizando **JUnit 5**, **Mockito** e **RestAssured**:

### Executar Testes Locais
```bash
./mvnw clean verify
```

### Visualizar Cobertura de Código (Jacoco)
Após a execução bem-sucedida do comando acima:
1. Navegue até a pasta `target/jacoco-report/`.
2. Abra o arquivo `index.html` em qualquer navegador web para auditar a cobertura por classe e método (limite mínimo de qualidade de **50%** configurado).

---

## 🎛️ Observabilidade

O serviço expõe telemetria rica em tempo real para monitoramento corporativo:
* **Endpoint de Métricas**: `GET http://localhost:8081/q/metrics`
* **Métricas Expostas**: Latência de requisições, status HTTP, uso de memória Heap JVM, taxa de Garbage Collector e threads ativas.
* **Integração**: Coletado pelo Prometheus e encaminhado ao Grafana Cloud via `remote_write` (conforme detalhado no repositório geral de infraestrutura).

---

## 📖 Documentação da API (Swagger / OpenAPI)

O microsserviço está configurado com suporte nativo ao **Swagger UI** e geração de especificação **OpenAPI** via extensão `quarkus-smallrye-openapi`.

### 🌐 Endpoints de Acesso em Desenvolvimento (DEV)

Em ambiente de desenvolvimento (local ou na nuvem), você pode acessar a documentação diretamente no microsserviço (completamente independente do API Gateway):

* **Swagger UI (Interface Visual)**: `http://localhost:8080/q/swagger-ui/`
  * No Render (DEV): [https://petshop-auth-service-dev.onrender.com/q/swagger-ui/](https://petshop-auth-service-dev.onrender.com/q/swagger-ui/)
* **OpenAPI Spec (Esquema JSON)**: `http://localhost:8080/q/openapi`
  * No Render (DEV): [https://petshop-auth-service-dev.onrender.com/q/openapi](https://petshop-auth-service-dev.onrender.com/q/openapi)

### 🔒 Controle de Ambientes e Segurança

Para alinhar segurança e performance em produção/homologação, a exibição da documentação segue esta estratégia:

1. **Inclusão na Compilação (`Build Time`)**:
   A propriedade `quarkus.swagger-ui.always-include=true` está configurada no arquivo principal `application.properties`. Isso garante que o Quarkus compile e empacote os arquivos estáticos do Swagger no JAR de produção gerado no Dockerfile.
2. **Bloqueio em Homologação/Produção (`Runtime`)**:
   Para evitar a exposição pública indesejada de ferramentas de teste, o Swagger é desativado em tempo de execução no perfil de homologação através da propriedade:
   ```properties
   quarkus.swagger-ui.enable=false
   ```
   Qualquer tentativa de acesso fora do ambiente DEV retornará erro `404 Not Found`.

---

## 🚀 Pipeline de CI/CD (GitHub Actions)

Este repositório possui fluxos totalmente automatizados integrando as melhores práticas DevOps:

1. **Continuous Integration (`ci.yml`)**:
   * Executado a cada push/pull request para as branches `main` e `develop`.
   * Realiza a compilação e validação do código com Java 21.
   * Envia relatórios estatísticos de qualidade para o **SonarCloud** (Project Key: `ocsane-figueira_petshop-auth-service`).
   * Para pushes aprovados em `main`, constrói a imagem Docker oficial multi-stage e envia para o Docker Hub com tags SHA e `main` (`ocsane/petshop-auth-service`).

2. **Automatic Release (`release.yml`)**:
   * Executado na branch `main` pós-CI bem-sucedido.
   * Utiliza **Semantic Release** para analisar os commits convencionais e atualizar o SemVer no GitHub automaticamente.

3. **Continuous Deployment (`cd.yml`)**:
   * O fluxo monitora a conclusão do CI. Caso a validação de testes finalize com sucesso:
     * Branch `develop`: Invoca o webhook do Render para atualizar o ambiente de desenvolvimento (`petshop-auth-service-dev`).
     * Branch `main`: Invoca o webhook do Render para atualizar o ambiente de produção (`petshop-auth-service`).
