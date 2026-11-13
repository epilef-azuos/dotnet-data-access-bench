# Spec 00 — Fundação

**Projeto:** dotnet-data-access-bench · **Branch:** feat/fase-0-fundacao · **Status:** Aprovada

## 1. Contexto

O repositório ainda não existe. Antes de escrever qualquer entidade ou consulta, a solução precisa
de uma base que imponha os mesmos padrões a todos os projetos (versão do .NET, nullable, analisadores,
versões centralizadas de pacotes), um banco local reproduzível e um CI que impeça código quebrado de
chegar à `main`.

## 2. Objetivo

Ter a solução completa compilando em Release localmente e no GitHub Actions, com a `main` protegida
e a documentação inicial publicada na tag `v0.1.0`.

## 3. Escopo

- **Inclui:** repositório no GitHub; proteção da `main`; solução `.slnx` com os 8 projetos e suas
  referências; `global.json`; `Directory.Build.props`; `Directory.Packages.props` (CPM);
  `.editorconfig`; `.gitignore`; `.gitattributes`; `docker-compose.yml` com SQL Server 2022 e
  `.env.example`; template de PR e de issue; `CHANGELOG.md`; `ci.yml` (restore + build);
  README inicial; spec 00; ADR 0001.
- **Não inclui:** entidades, DbContext, migrations, seed (Fase 1); pacotes NuGet e testes (Fases 1 e 2);
  `dotnet format`, testes e cobertura no CI (Fase 5); ADR de roadmap (registrado ao fim da Fase 4).

## 4. Regras de negócio

Não se aplica (fase de infraestrutura). Regras do repositório:

- RN-01: a `main` só recebe mudanças por Pull Request com o check `build` verde.
- RN-02: nenhum `.csproj` declara `Version` em `PackageReference`; versões ficam em `Directory.Packages.props`.
- RN-03: nenhum segredo é versionado; o `.env` fica fora do Git e o `.env.example` traz só valores locais.
- RN-04: commits seguem Conventional Commits com escopo.

## 5. Contratos

- **API:** não se aplica.
- **Eventos:** não se aplica.
- **Dados:** contêiner `bench-sqlserver` (SQL Server 2022 Developer), porta `1433` (configurável por
  `MSSQL_PORT`), volume `sqlserver-data`. O banco `DataAccessBench` é criado na Fase 1.

Dependências entre projetos:

| Projeto | Referencia |
|---|---|
| Bench.Domain | — |
| Bench.Data.Contracts | Domain |
| Bench.Data.Ef | Domain, Contracts |
| Bench.Data.Dapper | Domain, Contracts |
| Bench.Seed | Domain, Data.Ef |
| Bench.Runner | Contracts, Data.Ef, Data.Dapper |
| Bench.Tests.Unit | Domain, Seed |
| Bench.Tests.Integration | Contracts, Data.Ef, Data.Dapper, Seed |

`Bench.Data.Dapper` não referencia `Bench.Data.Ef`: as implementações são independentes e só
compartilham o contrato.

## 6. Requisitos não funcionais

- Build determinístico (`Deterministic`, `ContinuousIntegrationBuild` no CI).
- Warnings tratados como erro desde o primeiro commit.
- CI em até 15 minutos, com cancelamento de execuções antigas da mesma branch.
- Permissões mínimas no workflow (`contents: read`).

## 7. Critérios de aceite

- **Dado** o repositório clonado e o .NET 10 instalado, **quando** executo
  `dotnet build dotnet-data-access-bench.slnx -c Release`, **então** o build termina com 0 erros e 0 warnings.
- **Dado** o `.env` criado a partir do `.env.example`, **quando** executo `docker compose up -d`,
  **então** `docker compose ps` mostra o serviço `sqlserver` como `healthy`.
- **Dado** um PR aberto para a `main`, **quando** o CI termina, **então** o check `build` aparece verde
  e o merge só é liberado nessa condição.
- **Dado** um push direto na `main`, **quando** o Git envia, **então** o GitHub rejeita.
- **Dado** o merge do PR da fase, **quando** crio a tag `v0.1.0`, **então** ela aparece no GitHub
  apontando para o commit de merge.

## 8. Plano de testes

Sem testes automatizados nesta fase (não há código de domínio). A verificação é o build local, o
healthcheck do contêiner e o CI. Os projetos de teste já existem para que as referências fiquem
corretas desde o início; os pacotes de teste entram com o primeiro teste real, na Fase 1.

## 9. Tarefas

- [ ] Criar repositório no GitHub e commit inicial na `main`
- [ ] Criar a branch `feat/fase-0-fundacao`
- [ ] `global.json`, `Directory.Build.props`, `Directory.Packages.props`
- [ ] `.editorconfig`, `.gitignore`, `.gitattributes`
- [ ] Solução `.slnx` e os 8 projetos com referências
- [ ] `docker-compose.yml` e `.env.example`
- [ ] Templates de PR e issue
- [ ] `ci.yml` (restore + build)
- [ ] README, CHANGELOG, spec 00 e ADR 0001
- [ ] Abrir PR, configurar o ruleset da `main` exigindo o check `build`, fazer merge
- [ ] Criar a tag `v0.1.0`

## 10. Riscos e decisões

| Risco | Mitigação |
|---|---|
| `.slnx` não ser aberto por alguma ferramenta antiga | VS 2026 e o SDK do .NET 10 suportam; `dotnet sln migrate` gera um `.sln` se necessário |
| `TreatWarningsAsErrors` travar o avanço nas próximas fases | Corrigir o warning em vez de suprimir; suprimir só com justificativa no `.editorconfig` |
| Porta 1433 ocupada por um SQL Server instalado no Windows | Mudar `MSSQL_PORT` no `.env` |
| Senha do SA fraca recusada pelo SQL Server | `.env.example` já atende à política de complexidade |

Decisões: [ADR 0001 — Objetivo e escopo](../adr/0001-objetivo-e-escopo.md).

## 11. Instruções para o agente de IA

- Pode alterar apenas arquivos da raiz, `.github/`, `docs/` e os `.csproj`/`Program.cs` mínimos.
- Não adicionar pacotes NuGet, entidades, DbContext ou testes nesta fase.
- Nenhum `PackageReference` com `Version`; nenhum segredo real em arquivo versionado.
- Validar com `dotnet build dotnet-data-access-bench.slnx -c Release` (0 warnings) e `docker compose up -d`.
- Pronto = critérios de aceite da seção 7 atendidos e CI verde no PR.
