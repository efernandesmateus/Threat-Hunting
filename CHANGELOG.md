# Changelog

Todas as mudanças relevantes deste repositório são registradas aqui.

O formato segue o [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/) e as versões seguem o [Versionamento Semântico](https://semver.org/lang/pt-BR/):

- **MAJOR:** mudança que altera o escopo da metodologia ou quebra templates existentes;
- **MINOR:** novo documento, template ou hunt;
- **PATCH:** correção ou ajuste em conteúdo existente.

## [Não lançado]

## [0.2.0] - 2026-11-03

### Adicionado

- `00-servico/definicao-e-missao.md`: definição de threat hunting (o que é e o que não é), missão, princípios e proposta de valor.
- `00-servico/ofertas.md`: níveis Essencial, Avançado e Estratégico, comparativo e serviços avulsos (assessment de maturidade e hunt pontual).
- `00-servico/escopo.md`: itens incluídos, premissas, exclusões, modos de resposta (somente notificação ou contenção pelo SOC), limitações e prazos de comunicação de referência.
- `00-servico/papeis-e-responsabilidades.md`: papéis no SOC (Líder de Hunting, Hunter Sênior, Hunter) e no cliente, matriz de competências e matriz RACI.
- `00-servico/objetivos-e-capacidade.md`: objetivos no modelo GOST e modelo de dimensionamento do time.

### Alterado

- `00-servico/README.md`: índice dos documentos e ordem de leitura.

## [0.1.0] - 2026-11-02

Primeira versão pública.

### Adicionado

- Estrutura de pastas do repositório, com um `README.md` por pasta.
- `README.md` com visão geral, público-alvo, estrutura, plataformas, roadmap, autoria e licença.
- `LICENSE` (MIT).
- `CONTRIBUTING.md` com regras de segurança, convenções de nomes, IDs de hunt e fluxo de Git.
- `.gitignore` bloqueando arquivos de evidência e segredos.
- Decisões iniciais: ADR 0001 (OTHF como base), ADR 0002 (independência de API de fornecedor) e ADR 0003 (nenhum dado real no repositório).
- `referencias/OTHF/` com a licença MIT original e um resumo em português de cada módulo do OTHF.
- `02-templates/template-hunt.md` (v1.0): template dos hunts da biblioteca, com 16 seções.
- Guia de preenchimento, ciclo de vida dos hunts e regra de versionamento em `02-templates/README.md`.
