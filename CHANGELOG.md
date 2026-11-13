# Changelog

Todas as mudanças relevantes deste projeto são registradas aqui.

O formato segue o [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/)
e o projeto adota o [Versionamento Semântico](https://semver.org/lang/pt-BR/).

## [Unreleased]

## [0.1.0] - 2026-11-13

Fase 0 — Fundação.

### Added

- Solução `.slnx` com os projetos `Bench.Domain`, `Bench.Data.Contracts`, `Bench.Data.Ef`,
  `Bench.Data.Dapper`, `Bench.Seed`, `Bench.Runner`, `Bench.Tests.Unit` e `Bench.Tests.Integration`.
- `Directory.Build.props` com .NET 10, nullable, implicit usings, analisadores e warnings como erro.
- Central Package Management (`Directory.Packages.props`).
- `global.json` fixando o SDK do .NET 10.
- `.editorconfig`, `.gitignore` e `.gitattributes`.
- `docker-compose.yml` com SQL Server 2022 e `.env.example`.
- Workflow `ci.yml` com restore e build em Release.
- Template de Pull Request e template de issue por fase.
- README inicial, spec 00 (fundação) e ADR 0001 (objetivo e escopo).

[Unreleased]: https://github.com/epilef-azuos/dotnet-data-access-bench/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/epilef-azuos/dotnet-data-access-bench/releases/tag/v0.1.0
