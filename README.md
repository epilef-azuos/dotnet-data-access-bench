# dotnet-data-access-bench

[![ci](https://github.com/epilef-azuos/dotnet-data-access-bench/actions/workflows/ci.yml/badge.svg)](https://github.com/epilef-azuos/dotnet-data-access-bench/actions/workflows/ci.yml)
![.NET 10](https://img.shields.io/badge/.NET-10-512BD4)
![SQL Server 2022](https://img.shields.io/badge/SQL%20Server-2022-CC2927)

> **EF Core ou Dapper? Quando usar cada um?** Este repositório responde com dados:
> os mesmos cenários, o mesmo banco, os mesmos índices, medidos com BenchmarkDotNet
> e com testes que provam que as duas implementações retornam o mesmo resultado.

**Status:** 🚧 Fase 0 (fundação) — ainda não há resultados de benchmark.

## Por que este projeto

Trabalhei com EF Core e, principalmente, Dapper em produção. A discussão "qual é mais rápido"
costuma ser feita com opinião ou com benchmarks que comparam coisas diferentes (uma consulta
com `Include` contra um SQL escrito à mão que traz menos colunas, por exemplo).
Aqui, cada comparação só vale depois que um teste confirma que as duas versões devolvem
**o mesmo resultado**.

## Cenários

| # | Cenário | EF Core | Dapper (baseline) |
|---|---|---|---|
| 1 | Buscar por Id | Tracking, `AsNoTracking`, consulta compilada | `QuerySingleOrDefaultAsync` |
| 2 | Lista paginada com filtro | Entidade completa × projeção para DTO | `OFFSET/FETCH` |
| 3 | Join reserva + cliente + veículo | `Include`, `AsSplitQuery`, projeção | Multi-mapping |
| 4 | Relatório agregado (categoria/mês) | LINQ `GroupBy` | SQL direto |
| 5 | Inserir 1.000 registros | `AddRange` + `SaveChanges` | Loop de `ExecuteAsync` × `SqlBulkCopy` |
| 6 | Atualização em lote | Carregar e salvar × `ExecuteUpdateAsync` | `UPDATE ... WHERE` |

Domínio: locação de veículos (Cliente, Veiculo, Reserva), com cerca de 10 mil clientes,
2 mil veículos e 100 mil reservas gerados com Bogus a partir de uma seed fixa.

## Resultados

Ainda não executados. A tabela e as conclusões entram na Fase 4, a partir de execuções
locais documentadas em `results/`.

## Como rodar

### Pré-requisitos

- [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0)
- [Docker Desktop](https://docs.docker.com/desktop/) **atualizado** (versões de 2022, com Engine 20.x,
  não conseguem baixar as imagens atuais do SQL Server; veja [Solução de problemas](#solução-de-problemas))
- Cerca de 2 GB de RAM livres para o contêiner do SQL Server

### 1. Banco de dados (SQL Server 2022 no Docker)

```bash
# Linux/macOS: cp | Windows (PowerShell): Copy-Item
cp .env.example .env

docker compose up -d
docker compose ps        # aguarde a coluna STATUS mostrar "(healthy)" — 30 a 60 s na primeira vez
```

Para confirmar que o banco responde (a senha é lida da variável de ambiente dentro do contêiner):

```bash
docker exec bench-sqlserver bash -c '/opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P "$MSSQL_SA_PASSWORD" -C -Q "SELECT @@VERSION"'
```

### 2. Build

```bash
dotnet build dotnet-data-access-bench.slnx -c Release
```

### 3. Desligar o banco

```bash
docker compose stop      # para o contêiner e libera memória (dados preservados)
docker compose down -v   # remove contêiner e volume (banco zerado)
```

## Solução de problemas

| Sintoma | Causa | Correção |
|---|---|---|
| `archive/tar: invalid tar header` no `docker pull` | Docker Desktop antigo (Engine 20.x) | Atualizar o Docker Desktop |
| `Get "https://mcr.microsoft.com/v2/": EOF` | DNS não resolve o registro da Microsoft | Docker Desktop > Settings > Docker Engine: `"dns": ["1.1.1.1", "8.8.8.8"]` > Apply & restart |
| `docker compose up` reclama de `MSSQL_SA_PASSWORD` | Arquivo `.env` não criado | Copiar `.env.example` para `.env` |
| Porta 1433 ocupada | SQL Server instalado na máquina | Mudar `MSSQL_PORT` no `.env` |
| Contêiner reiniciando em loop | Senha fora da política do SQL Server ou pouca memória para o Docker | Senha forte (8+ caracteres, maiúsculas, minúsculas, números e símbolos); ao menos 4 GB para o Docker |

## Estrutura

```
src/
  Bench.Domain/           entidades e DTOs
  Bench.Data.Contracts/   interfaces comuns dos repositórios
  Bench.Data.Ef/          implementação EF Core
  Bench.Data.Dapper/      implementação Dapper
  Bench.Seed/             cria o schema e popula o banco
bench/Bench.Runner/       BenchmarkDotNet (sempre em Release)
tests/                    unitários e integração (Testcontainers)
docs/specs/               specs por fase (Spec-Driven Development)
docs/adr/                 decisões de arquitetura
results/                  relatórios exportados (Markdown + JSON)
```

## Roadmap

| Fase | Entrega | Versão |
|---|---|---|
| 0 | Fundação: solução, padrões, Docker, CI de build | `v0.1.0` |
| 1 | Domínio, schema, índices e seed | `v0.2.0` |
| 2 | Repositórios EF/Dapper e testes de equivalência | `v0.3.0` |
| 3 | Benchmarks e primeira rodada | `v0.4.0` |
| 4 | Análise, resultados e ADR da decisão | `v0.5.0` |
| 5 | CI/CD completo, cobertura e release | `v1.0.0` |

Depois da `v1.0.0`: Bench.Api (Minimal API), dashboard em Angular e deploy no Azure.

## Documentação

- [Spec 00 — Fundação](docs/specs/00-fundacao.md)
- [ADR 0001 — Objetivo e escopo](docs/adr/0001-objetivo-e-escopo.md)
- [CHANGELOG](CHANGELOG.md)

## Autor

Felipe Assis Souza — [GitHub](https://github.com/epilef-azuos) · [LinkedIn](https://www.linkedin.com/in/felipeassissouza/)
