# Trabalho 5 — Pipeline CI mínimo e reprodutível

- **Disciplina:** Cloud Computing e DevOps Avançado — Prof. Ranghetti
- **Equipe:** Hendreu Satoshi (240061), André Luiz (14623), Gabriel Friedrichsen (14552)
- **Ferramenta:** GitHub Actions
- **Arquivo do pipeline:** `.github/workflows/ci.yml`

## Gatilho
O pipeline é executado automaticamente em:
- `push` na branch `master`;
- abertura ou atualização de `pull request` para a `master`;
- execução manual pela aba Actions (`workflow_dispatch`).

## Etapas
1. **Checkout** — obtém a versão exata do código do commit.
2. **Configuração do Java 21** (Temurin), com cache das dependências do Maven.
3. **Build e testes** — `./mvnw clean test`. Se qualquer teste falhar, o pipeline é interrompido.
4. **Empacotamento** — `./mvnw package` gera o arquivo JAR.
5. **Publicação do artefato** — o JAR é salvo com o nome `biblioteca-<hash do commit>`.
6. **Relatório de testes** — salvo sempre, inclusive quando há falha, como evidência.

## Dependências
- Java 21 e Maven (via Maven Wrapper `mvnw`, versionado no repositório).
- Dependências da aplicação declaradas no `pom.xml`.

## Artefato e rastreabilidade
O artefato recebe o hash do commit no nome (ex.: `biblioteca-a1b2c3d...`). Assim, é possível saber exatamente qual código gerou cada JAR, qual execução o produziu e quais testes foram executados.

## Como reproduzir
- **No GitHub:** aba Actions → "CI Biblioteca" → "Run workflow".
- **Localmente:** `./mvnw clean test` e depois `./mvnw package`. O JAR fica em `target/`.
Cada execução usa uma máquina virtual nova e limpa, então o resultado não depende do computador de ninguém.

## Como investigar falhas
1. Abrir a aba Actions e clicar na execução com ❌.
2. Identificar a etapa em vermelho e ler o log.
3. Baixar o artefato `relatorio-testes-<hash>` para ver o detalhe dos testes.
4. Corrigir o código ou o teste e enviar um novo commit.

## Evidências

Histórico completo: aba **Actions** do repositório.

| Execução | Resultado | O que demonstra |
|---|---|---|
| [#2](https://github.com/Hendreu04/biblioteca/actions/runs/38620503410) | ✅ Sucesso | Build, testes e artefatos `biblioteca-<hash>` e `relatorio-testes-<hash>` gerados |
| [#4](https://github.com/Hendreu04/biblioteca/actions/runs/38621983601) | ❌ Falha | Teste com falha proposital: o pipeline parou em "Compilar e executar testes" e o JAR **não** foi gerado |
| [#5](https://github.com/Hendreu04/biblioteca/actions/runs/38623146313) | ✅ Sucesso | Correção do teste: o pipeline voltou ao normal |

**Trecho do log da falha (#4):**
- `Tests run: 1, Failures: 1` → um teste executado, um falhou
- `BibliotecaApplicationTests.contextLoads:11 Falha proposital para testar o pipeline` → qual teste, em que linha e por quê
- `BUILD FAILURE` → o build foi interrompido

[arraste aqui os 4 prints]
## Segurança e segredos
- O pipeline usa `permissions: contents: read` (menor privilégio).
- Nenhuma senha ou token é usado ou exibido nos logs.
- **Ponto de melhoria identificado:** o segredo do JWT (`api.security.token.secret`) está escrito no `application.properties`. Em um ambiente real, ele deveria ser lido de uma variável de ambiente, armazenada nos **GitHub Secrets** ou no gerenciador de segredos do provedor, e nunca ficar no código.
