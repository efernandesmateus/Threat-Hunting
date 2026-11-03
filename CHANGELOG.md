# Changelog

Todas as mudanças relevantes deste repositório são registradas aqui.

O formato segue o [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/) e as versões seguem o [Versionamento Semântico](https://semver.org/lang/pt-BR/):

- **MAJOR:** mudança que altera o escopo da metodologia ou quebra templates existentes;
- **MINOR:** novo documento, template ou hunt;
- **PATCH:** correção ou ajuste em conteúdo existente.

## [Não lançado]

## [0.4.0] - 2026-11-03

### Adicionado

- `03-biblioteca/exfiltration/th-exf-001-exfiltracao-via-dns.md` (v1.0): primeiro hunt da biblioteca, exfiltração de dados via DNS (T1048.003), com status **Proposto**.
- `03-biblioteca/exfiltration/th-exf-001-exfiltracao-via-dns.yml`: regras Sigma de apoio (rótulo longo e correlação por host), com sintaxe verificada.
- ADR 0005: formato das consultas (pseudoconsulta obrigatória, regra Sigma recomendada, traduções por plataforma fora do repositório) e regra para citar ferramentas de adversários.
- ADR 0006: validação dos hunts pelo adotante; a biblioteca passa a ser de referência.
- Seção "Contribuindo com validações" no `CONTRIBUTING.md`.
- Índice da biblioteca, com os 11 hunts planejados, o backlog futuro e as táticas sem hunt previsto (com justificativa), em `03-biblioteca/README.md`.
- Roadmap do `README.md` dividido entre metodologia (até a v1.0.0) e biblioteca (a partir da v1.1.0).

### Alterado

- `02-templates/template-hunt.md` (v1.1): seção 8 reorganizada em pseudoconsulta, regra Sigma e implementações por plataforma; seção 9 dividida em orientação de validação e validações registradas; campo de versão do ATT&CK no cabeçalho.
- `02-templates/README.md`: dois níveis de status (na biblioteca: Rascunho, Proposto, Validado pela comunidade, Descontinuado; operacional, no ambiente de quem adota: Em validação, Ativo, Automatizado) e nova regra de versionamento.
- `01-processo/desenvolvimento-de-hunts.md`: diferença entre a publicação na biblioteca e a validação pelo adotante.
- `README.md`: aviso sobre o status dos hunts.
- `CONTRIBUTING.md`: seção de consultas alinhada ao ADR 0005.
- `01-processo/escalonamento.md`: rótulo encurtado no diagrama do modo somente notificação.

## [0.3.0] - 2026-11-03

### Adicionado

- `01-processo/identificacao-de-hunts.md`: fontes de hipóteses, gatilhos e avaliação de ameaças por cliente.
- `01-processo/desenvolvimento-de-hunts.md`: as 7 etapas do desenvolvimento de um hunt, de Rascunho a Ativo, com revisão por pares.
- `01-processo/backlog-e-priorizacao.md`: backlogs da biblioteca e por cliente, viabilidade dos dados como critério eliminatório, pontuação de prioridade, Pirâmide da Dor, cadência, rotação e hunts extraordinários.
- `01-processo/execucao-de-hunts.md`: ciclo mensal em 6 fases, checklist de preparação, campos mínimos do registro de execução e tratamento de hunts não executáveis.
- `01-processo/escalonamento.md`: severidade em 4 níveis, escalonamento interno, fluxos por modo de resposta, conteúdo da notificação e preservação de evidências.
- `01-processo/automacao-e-melhoria.md`: critérios e fluxo do "hunt once", lições aprendidas e melhoria contínua.
- Diagramas Mermaid dos principais fluxos.
- ADR 0004: neutralidade de fabricante.

### Alterado

- `01-processo/README.md`: índice, diagrama do ciclo completo e diferença entre desenvolvimento e execução.
- `02-templates/README.md`: exceção no ciclo de vida para hunts extraordinários em validação.
- Removidas todas as menções a fabricantes e produtos comerciais (README, CONTRIBUTING, template de hunt, resumo do OTHF e demais documentos). Capacidades passam a ser descritas pelo tipo (SIEM, EDR, XDR).
- `02-templates/template-hunt.md`: seção 8 passa a ser organizada por tipo de plataforma.
- `CONTRIBUTING.md`: tabela de táticas atualizada para o MITRE ATT&CK Enterprise v20 (15 táticas). Defense Evasion (`DEF`), aposentada no ATT&CK v19, foi substituída por Stealth (`STL`, TA0005) e Defense Impairment (`DIM`, TA0112). Incluídos os IDs das táticas e a regra de atualização do ATT&CK.

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
