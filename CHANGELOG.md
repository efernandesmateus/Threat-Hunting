# Changelog

Todas as mudanças relevantes deste repositório são registradas aqui.

O formato segue o [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/) e as versões seguem o [Versionamento Semântico](https://semver.org/lang/pt-BR/):

- **MAJOR:** mudança que altera o escopo da metodologia ou quebra templates existentes;
- **MINOR:** novo documento, template ou hunt;
- **PATCH:** correção ou ajuste em conteúdo existente.

## [Não lançado]

## [1.3.0] - 2026-11-03

Terceiro bloco de hunts da biblioteca.

### Adicionado

- `03-biblioteca/discovery/th-dis-001-enumeracao-diretorio-em-rajada.md` (v1.0, Proposto) e regras Sigma (4 regras e 1 correlação de rajada): enumeração de contas, grupos privilegiados, confianças e controladores do domínio, incluindo AdFind (S0552) e BloodHound (S0521).
- `03-biblioteca/lateral-movement/th-lat-001-movimentacao-lateral-contas-administrativas.md` (v1.0, Proposto) e regras Sigma (4 regras e 1 correlação de espalhamento): pares conta → host nunca vistos, espalhamento, logons entre estações, pass the hash e execução remota.
- `03-biblioteca/collection/th-col-001-compactacao-grandes-volumes.md` (v1.0, Proposto) e regras Sigma (5 regras, Windows e Linux): compactação com senha, divisão em volumes e diretórios de preparação, detectadas pelo comportamento, sem depender do nome do compactador.

### Alterado

- `03-biblioteca/README.md`: índice com os novos hunts e lista de planejados atualizada.
- `04-onboarding/matriz-de-cobertura.md`: status dos novos hunts, fontes alternativas do TH-DIS-001 e nota sobre a telemetria de endpoint.
- `README.md`: status e roadmap.

## [1.2.0] - 2026-11-03

Segundo bloco de hunts da biblioteca.

### Adicionado

- `03-biblioteca/stealth/th-stl-001-utilitarios-legitimos-executando-codigo.md` (v1.0, Proposto) e regras Sigma (5 regras): utilitários nativos do Windows usados para executar ou buscar código (rundll32, regsvr32, mshta, msiexec, certutil), incluindo cópias renomeadas.
- `03-biblioteca/defense-impairment/th-dim-001-desativacao-ferramentas-logs.md` (v1.0, Proposto) e regras Sigma (6 regras, Windows e Linux): limpeza de logs, desativação de auditoria, do firewall e de agentes, supressão de histórico e **silêncio de telemetria**.
- `03-biblioteca/credential-access/th-crd-001-acesso-memoria-processo-autenticacao.md` (v1.0, Proposto) e regras Sigma (5 regras): acesso à memória do LSASS, dumps e técnicas conhecidas de extração de credenciais.
- Dois hunts futuros no backlog: complementos para Linux e macOS do TH-STL-001 e do TH-CRD-001.

### Alterado

- `03-biblioteca/README.md`: índice com os novos hunts e lista de planejados atualizada.
- `04-onboarding/matriz-de-cobertura.md`: status dos novos hunts.
- `README.md`: status e roadmap.

## [1.1.0] - 2026-11-03

Primeiro bloco de hunts da biblioteca.

### Adicionado

- `03-biblioteca/initial-access/th-ini-001-acesso-remoto-origem-incomum.md` (v1.0, Proposto): autenticações bem-sucedidas no acesso remoto externo com atributos raros para a conta, com pontuação por indicadores. Sem regra Sigma (lógica baseada em histórico e enriquecimento).
- `03-biblioteca/execution/th-exe-001-interpretadores-comandos-ofuscados.md` (v1.0, Proposto) e regras Sigma (5 regras, Windows e Linux): interpretadores com comandos codificados, ofuscados ou de download e execução.
- `03-biblioteca/persistence/th-per-001-tarefas-servicos-fora-do-padrao.md` (v1.0, Proposto) e regras Sigma (4 regras, Windows e Linux): tarefas agendadas e serviços raros ou com ações suspeitas.
- ADR 0007: exceção técnica que permite citar sistemas operacionais e seus componentes nativos, mantendo a proibição para produtos de segurança.

### Alterado

- ADR 0004: status indica que foi complementado pelo ADR 0007.
- `CONTRIBUTING.md` e `README.md`: regra de neutralidade com a exceção técnica.
- `referencias/OTHF/README.md`: menções a artefatos Windows restauradas, conforme o ADR 0007.
- `03-biblioteca/README.md`: índice com os novos hunts e lista de planejados atualizada.
- `04-onboarding/matriz-de-cobertura.md`: status dos novos hunts e fonte complementar do TH-PER-001.

## [1.0.0] - 2026-11-03

Primeira versão completa da metodologia: definição do serviço, processo operacional, templates, biblioteca de referência com o primeiro hunt, onboarding de cliente e assessment de maturidade.

### Adicionado

- `README.md`: seção "Por onde começar", com o ponto de entrada para cada objetivo.

### Alterado

- Revisão geral de consistência entre os documentos, alinhando o uso dos dois níveis de status (biblioteca e operacional, ADR 0006):
  - `01-processo/README.md`: tabela dos dois processos e diagrama do ciclo completo, que passa a incluir a validação no ambiente do time;
  - `01-processo/desenvolvimento-de-hunts.md`: tabela de etapas e etapa 7 (publicação como Proposto na biblioteca; Ativo apenas no ambiente do time);
  - `01-processo/automacao-e-melhoria.md`, `01-processo/backlog-e-priorizacao.md` e `01-processo/identificacao-de-hunts.md`: referências ao status operacional;
  - `00-servico/objetivos-e-capacidade.md`: objetivos das metas 2 e 4.
- `README.md`: status da versão e descrição das pastas de onboarding e maturidade.

## [0.7.0] - 2026-11-03

### Adicionado

- `05-maturidade/modelo-de-maturidade.md`: modelo em cinco eixos e cinco níveis (0 a 4), com 65 critérios verificáveis e cumulativos, regras de avaliação, duas visões (cliente e com o serviço) e indicação de quem pode atender cada critério.
- `05-maturidade/questionario-assessment.md`: roteiro de condução, com pergunta e evidência esperada para cada critério.
- `05-maturidade/template-relatorio-maturidade.md`: modelo do relatório de assessment, com perfil por eixo, lacunas, recomendações priorizadas e evolução.

### Alterado

- `05-maturidade/README.md`: índice dos documentos e periodicidade de aplicação.

### Corrigido

- `referencias/OTHF/README.md` e `05-maturidade/README.md`: o modelo de maturidade do OTHF tem cinco níveis (0 a 4), e não quatro (0 a 3), como registrado anteriormente.

## [0.6.0] - 2026-11-03

### Adicionado

- `04-onboarding/checklist-onboarding.md`: roteiro de onboarding em seis fases (kick-off, acessos, dados, contexto, planejamento e aceite), com responsáveis, regras de acesso do SOC e manutenção após o onboarding.
- `04-onboarding/template-ficha-cliente.md`: modelo da ficha com os acordos do onboarding.
- `04-onboarding/template-inventario-fontes.md`: modelo do inventário de fontes de dados, com oito categorias de fonte e critérios de qualidade em três níveis.
- `04-onboarding/matriz-de-cobertura.md`: método de viabilidade dos hunts (Viável, Parcial, Inviável), modelo da matriz, priorização das recomendações de visibilidade e mapa de referência das fontes por hunt.

### Alterado

- `04-onboarding/README.md`: índice dos documentos.

## [0.5.0] - 2026-11-03

### Adicionado

- `02-templates/template-registro-execucao.md`: modelo do registro de cada execução de hunt em um cliente.
- `02-templates/template-relatorio-cliente.md`: modelo do relatório periódico ao cliente, com parte executiva de uma página e parte técnica, incluindo evolução entre ciclos e técnicas caçadas.
- `02-templates/metricas.md`: catálogo de métricas de operação, valor, redução da superfície de ataque e biblioteca, com fórmula, frequência, público e regras de uso.

### Alterado

- `02-templates/README.md`: índice dos novos templates.
- `01-processo/execucao-de-hunts.md` e `00-servico/objetivos-e-capacidade.md`: referências aos novos templates.

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
