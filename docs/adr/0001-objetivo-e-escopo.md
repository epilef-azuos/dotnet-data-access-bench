# ADR 0001 — Comparar EF Core e Dapper com cenários equivalentes, testados e medidos localmente

**Status:** Aceito · **Data:** 2026-11-12

## Contexto

A pergunta "EF Core ou Dapper?" aparece em entrevistas e em decisões reais de projeto, e quase
sempre é respondida com opinião. Benchmarks publicados costumam comparar consultas que não são
equivalentes (colunas diferentes, tracking ligado de um lado e desligado do outro, índices
diferentes), o que invalida a conclusão. O objetivo deste repositório é produzir uma resposta
reproduzível e honesta, servindo também de portfólio para vaga de Desenvolvedor .NET Pleno.
Restrições: projeto curto, executado num notebook (Windows 11, i5 11ª geração, 32 GB), sem custo
de nuvem nesta etapa.

## Decisão

1. **Mesmas condições:** EF Core e Dapper rodam sobre o mesmo SQL Server 2022, o mesmo schema
   (criado pelas migrations do EF Core), os mesmos índices e os mesmos dados (Bogus com seed fixa).
2. **Mesmo contrato:** as duas implementações seguem as interfaces de `Bench.Data.Contracts`.
3. **Equivalência antes de velocidade:** um cenário só entra no benchmark depois que um teste de
   integração (Testcontainers) prova que EF Core e Dapper retornam o mesmo resultado.
4. **Medição:** BenchmarkDotNet com `[MemoryDiagnoser]`, Dapper como `Baseline`, sempre em Release,
   fora do debugger, e resultados exportados em Markdown e JSON em `results/`.
5. **Escopo:** seis cenários (busca por Id, paginação, join, agregação, inserção em massa,
   atualização em lote). Um único banco (SQL Server).
6. **Fora do escopo agora:** API, frontend, deploy, outros bancos, outros ORMs. Ficam para um ADR
   de roadmap e só começam depois da `v1.0.0`.
7. **Números do CI não são resultado:** o runner compartilhado do GitHub Actions serve apenas para
   verificar que os benchmarks compilam e executam.

## Alternativas consideradas

| Alternativa | Prós | Contras |
|---|---|---|
| **Cenários equivalentes + testes de equivalência + BenchmarkDotNet local** (escolhida) | Comparação justa e reproduzível; demonstra testes, método e senso crítico | Mais trabalho antes do primeiro número; resultados válidos só para o hardware local |
| Microbenchmark simples com `Stopwatch` | Rápido de escrever | Sem aquecimento, sem estatística, sem medição de memória; números pouco confiáveis |
| Teste de carga numa API (k6, NBomber) | Mais próximo de produção | Mistura ORM com HTTP, serialização e rede; não isola a diferença entre ORMs |
| Usar banco em memória ou SQLite | Setup mais simples | Não representa o SQL Server que usei em produção; planos de execução diferentes |
| Rodar os benchmarks no GitHub Actions | Automatizado | Hardware compartilhado e variável distorce a medição |

## Consequências

- **Positivas:** resultados defensáveis numa entrevista ("como você garantiu que comparou a mesma
  coisa?"); o repositório demonstra testes de integração, CI, ADRs e Spec-Driven Development.
- **Negativas / custos:** os números valem para um notebook com banco local; em produção, a latência
  de rede e o plano de execução costumam pesar mais do que a diferença entre ORMs, e o README precisa
  deixar isso claro. O SQL Server em Docker consome cerca de 2 GB de RAM.
- **O que faria mudar esta decisão:** necessidade de comparar bancos (incluir PostgreSQL), de medir
  sob concorrência (teste de carga separado) ou de rodar benchmarks em hardware dedicado.

## Referências

- [Spec 00 — Fundação](../specs/00-fundacao.md)
- [BenchmarkDotNet — documentação](https://benchmarkdotnet.org/articles/overview.html)
- [EF Core — performance](https://learn.microsoft.com/ef/core/performance/)
- [Dapper — repositório oficial](https://github.com/DapperLib/Dapper)
- [Testcontainers para .NET](https://dotnet.testcontainers.org/)
