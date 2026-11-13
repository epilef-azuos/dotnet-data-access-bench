# results

Relatórios exportados pelo BenchmarkDotNet (Markdown e JSON) de execuções **locais**.

Cada execução fica numa pasta própria (`2026-11-13_<descricao>/`) com os metadados do ambiente
(CPU, RAM, .NET, SQL Server, versões do EF Core e do Dapper). O formato do JSON é definido na Fase 3
e serve de base para a futura Bench.Api.

Resultados do GitHub Actions **não** são guardados aqui: o runner compartilhado distorce a medição.
