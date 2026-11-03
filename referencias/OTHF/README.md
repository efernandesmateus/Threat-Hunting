# OTHF — Open Threat Hunting Framework

| | |
|---|---|
| **Autores** | TactiKoolSec, Grifter, Sameer Korrane |
| **Repositório** | https://github.com/TactiKoolSec/OTHF |
| **Versão de referência** | commit `4f1cc5ff841ed01bfc61479734a236b0d727c3e7` (branch `main`, maio de 2023) |
| **Licença** | MIT — ver [LICENSE](LICENSE) |

Esta pasta contém a licença original e um resumo, em português, de cada módulo do OTHF. O material original **não é copiado** para cá: consulte-o no repositório oficial, na versão de referência acima.

## Visão geral

O OTHF é um guia para criar e amadurecer um programa de threat hunting. Ele não é uma coleção de detecções: foca nas peças organizacionais, operacionais e técnicas que costumam ser negligenciadas e que determinam o sucesso do programa. É independente de fornecedor e modular, podendo ser usado em parte por quem não tem um time dedicado.

## Resumo dos módulos

### 1 — Introduction

Apresenta o propósito do framework: dar base conceitual sobre threat hunting e diretrizes práticas para lançar e amadurecer um programa.

### 2 — Foundational

- **Apoio da liderança:** o programa depende de patrocínio executivo. Sugere argumentos como redução do tempo de detecção, melhoria das detecções automatizadas, validação dos controles e redução de custos de resposta a incidentes. Recomenda começar pequeno e demonstrar valor.
- **Posicionamento:** idealmente o time reporta à liderança de segurança (CISO).
- **Definição de threat hunting:** cada organização deve documentar a sua. O OTHF recomenda que ela diferencie hunting de atividade reativa (buscar o que **não** gerou alerta), descreva um processo repetível e seja baseada em hipóteses.
- **Missão:** uma ou duas frases, revisáveis, que orientam o programa.
- **Estratégia e objetivos:** tipos de metas (curto e longo prazo, quantitativas, qualitativas, de resultado, de processo) e o modelo GOST (Goals, Objectives, Strategy, Tactics).

### 3 — Structure

- **Papéis:** Principal Threat Hunter, Senior Threat Hunter e Threat Hunter, com as competências de cada um.
- **Perfil:** experiência ofensiva, mentalidade investigativa e atenção a detalhes. Hunting é detecção de anomalias, e só se reconhece a anomalia conhecendo o normal.
- **Dimensionamento:** em organizações pequenas, começar com 4 a 6 horas por analista por semana e documentar resultados para ganhar mais tempo; em grandes, dedicar de 10% a 20% do SOC/CIRT a hunting.
- **Comunicação:** relacionamento com TI, SOC/CIRT e threat intel; base de conhecimento compartilhada; qualquer membro pode levantar uma suspeita, que deve ser investigada.
- Inclui uma planilha de matriz de habilidades.

### 4 — Preparation

- **Dados:** sem dados não há hunting. Lista categorias de fontes (rede, endpoints, aplicações, ferramentas de segurança, controles físicos).
- **Dicionário de dados:** inventário padronizado das fontes e seus campos. Cita projetos abertos de dicionário de dados e de análise (como o MITRE CAR) e um framework de gestão de coleta como apoio.
- **Confiabilidade:** critérios de qualidade dos dados (precisão, completude, consistência, atualidade, unicidade, validade).
- **Stack tecnológica** necessária para coletar e analisar os dados.

### 5 — Threat Hunting Labs

Ambientes de laboratório para gerar dados de teste: infraestrutura como código em nuvem pública e máquinas virtuais próprias do OTHF, com um domínio enviando logs para plataformas de SIEM.

> **Atenção:** os laboratórios do OTHF usam credenciais padrão publicadas no próprio repositório. Se forem usados, devem ficar isolados e ter as senhas trocadas.

### 6 — Operational

O módulo central. Resume o framework em cinco passos: conhecer as ameaças aplicáveis, entender seus TTPs, desenvolver um hunt por TTP, automatizar os hunts e revalidar periodicamente.

- **Identificação de hunts:** por inteligência (estratégica e tática, com MITRE ATT&CK e o catálogo de grupos da ETDA), avaliação de ameaças (quem ataca o setor, o que busca, qual o impacto, como faria), vulnerabilidades e exploits, superfície de ataque, missão da organização e observação do ambiente.
- **Processo de hunting do OTHF:** definir um objetivo (critérios SMART) → desenvolver a hipótese (formato "se... então" ou "dado, quando, então") → validar se os dados existem e têm qualidade → gerar dados de teste (ex.: testes abertos de emulação de adversário) → definir a estratégia de busca comparando o normal com o sinal do ataque → validar o hunt em dados históricos e ajustar falsos positivos → documentar os achados. Também apresenta o processo TaHiTI.
- **Cadência:** gatilhos para novos hunts (nova inteligência, incidentes, pentests, tabletop, outros hunts, ativos críticos, modelagem de ameaças, regulação), backlog, priorização (com foco no topo da Pirâmide da Dor) e agendamento.
- **Automação:** abordagem "hunt once" — executar o hunt uma vez e convertê-lo em detecção contínua, com critérios para essa migração.
- **Melhoria contínua:** lições aprendidas após cada hunt.
- **Maturidade:** cita o Hunting Maturity Model (HMM) e propõe o modelo do OTHF, com níveis de 0 a 3 em cinco eixos: Pessoas, Processo, Fontes de dados, Threat intel e Métricas. Inclui um arquivo de apresentação com o modelo.

### 7 — Measurement

Documentar é o que prova o valor do hunting, mesmo quando nada malicioso é encontrado. Propõe:

- **Registro de cada hunt:** hipótese, inteligência usada, fontes de dados, sistemas e protocolos analisados, indicadores encontrados, mitigações criadas, participantes.
- **Métricas do programa:** hunts realizados, horas dedicadas, alertas e relatórios criados, inteligência gerada, playbooks de automação, incidentes originados, pontos cegos encontrados, recomendações feitas.
- **Métricas de redução de superfície de ataque:** TTPs e técnicas ATT&CK investigadas, configurações incorretas, vulnerabilidades, protocolos em texto claro, credenciais expostas, sistemas inseguros, ativos desconhecidos, ferramentas de atacante e malware encontrados.

### 8 — Communication

Estratégia de comunicação (objetivo, mensagens-chave, públicos, métricas) adaptada a cada público: outras áreas, SOC/CIRT, o próprio time de hunting e a liderança.

### 9 — Appendix

Exemplo de avaliação de ameaças para uma empresa de varejo, usando MITRE ATT&CK e ETDA para chegar aos grupos prioritários e às técnicas a investigar. O exemplo de proposta de programa está marcado como "a fazer" no original.

### 10 — Hunt Data Sources

Aprofundamento em artefatos de endpoint: Shimcache, Amcache, Autoruns, Prefetch, processos em execução, conexões de rede ativas, metadados de executáveis, serviços e logs de eventos. Para cada um: campos importantes, valor para hunting, técnicas e triagem.

### 11 — Hunting Techniques

Técnicas de análise: stacking (contagem de frequência para achar o raro), enriquecimento (inteligência, reputação, criticidade do ativo, geolocalização), agrupamento, busca por palavra-chave e busca por IOC.

### Materiais extras

- **Practice and Sample Content:** três cenários para praticar a formulação de objetivo e hipótese (um deles sobre a exploração de uma vulnerabilidade em servidor de e-mail).
- **Sample Scripts:** scripts Python com a biblioteca pyattck para mapear grupos adversários a técnicas e priorizar as mais usadas.

## Como o OTHF é aproveitado neste serviço

| Módulo do OTHF | Destino no serviço | Adaptação principal |
|----------------|--------------------|---------------------|
| 2 Foundational | `00-servico/` | Apoio executivo e posicionamento viram proposta de valor e oferta do serviço |
| 3 Structure | `00-servico/` | Papéis e dimensionamento aplicados ao SOC que atende vários clientes |
| 4 Preparation | `04-onboarding/` | Dicionário de dados vira inventário de fontes **por cliente** |
| 5 Labs | apoio à validação de hunts | Usado internamente para gerar dados de teste |
| 6 Operational | `01-processo/`, `05-maturidade/` | Cadência e priorização por cliente; maturidade como assessment vendável |
| 7 Measurement | `02-templates/` | Métricas reportadas ao cliente e métricas internas do serviço |
| 8 Communication | `02-templates/` | Público principal passa a ser o cliente |
| 10 e 11 | `03-biblioteca/` | Base para hunts; complementado com outras fontes |

## Limitações conhecidas

- Sem atualizações desde maio de 2023.
- Conteúdo técnico centrado em artefatos de endpoint; pouco sobre nuvem, identidade e rede.
- Cita fabricantes e produtos em vários pontos; neste projeto, essas referências foram generalizadas (ver ADR 0004).
- Escrito para um time interno, não para MSSP.
- Algumas seções estão incompletas no original (ex.: exemplo de proposta no apêndice).
