# 0003 — Nenhum dado real no repositório

- **Status:** Aceita
- **Data:** 2026-11-02

## Contexto

Este é um repositório público. Qualquer dado real publicado aqui (logs, endereços, nomes, evidências) exporia ambientes, lacunas de visibilidade e incidentes de terceiros, violaria obrigações de confidencialidade e de privacidade (LGPD) e poderia ajudar atacantes. O mesmo vale para quem fizer um fork e usar o projeto internamente.

## Decisão

O repositório guarda **somente metodologia**: processos, templates, hunts genéricos e documentação. Dados operacionais (logs, IPs, hostnames, usuários, nomes de organizações, evidências e resultados de hunts) ficam nos sistemas de atendimento de quem executa o serviço.

## Consequências

- Exemplos usam apenas dados fictícios, conforme o `CONTRIBUTING.md`.
- O `.gitignore` bloqueia extensões típicas de evidências, capturas e segredos.
- Todo pull request deve ser revisado para essa regra antes do merge.
- Contribuições baseadas em material interno de empresas só são aceitas se forem reescritas de forma genérica e o contribuidor tiver o direito de publicá-las.
