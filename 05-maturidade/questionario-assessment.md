# Questionário de assessment

Roteiro para conduzir o assessment de maturidade. Cada critério do [modelo](modelo-de-maturidade.md) tem uma pergunta e a evidência esperada.

## Como conduzir

1. **Prepare:** envie o questionário ao ponto focal com antecedência e peça as evidências disponíveis.
2. **Entreviste:** percorra os eixos com as pessoas responsáveis por cada tema (segurança, infraestrutura, inteligência, gestão).
3. **Verifique as evidências:** um critério só é atendido com evidência. Declarações sem evidência contam como **não atendido**.
4. **Avalie as duas visões:** para cada critério, registre se é atendido **pelo cliente** e se é atendido **com o serviço**.
5. **Consolide** no [template de relatório](template-relatorio-maturidade.md).

**Duração de referência:** de 2 a 4 horas de entrevistas, mais a análise das evidências.

**Respostas:** **Sim** (atendido, com evidência) · **Não** · **Parcial** (conta como não atendido, mas ajuda a orientar a recomendação) · **N/A** (não se aplica ao escopo; justifique).

As respostas preenchidas contêm informações do cliente e ficam fora deste repositório ([ADR 0003](../decisoes/0003-sem-dados-reais-no-repositorio.md)).

## Pessoas (PES)

| ID | Nível | Pergunta | Evidência esperada | Cliente | Com o serviço | Observações |
|----|:-----:|----------|--------------------|:-------:|:-------------:|-------------|
| PES-1.1 | 1 | Quem procura ameaças que não geraram alerta? Com que frequência? | Exemplos de hunts ou investigações proativas realizadas | |  | |
| PES-1.2 | 1 | Quais competências o time considera necessárias para hunting? | Lista ou documento de competências, mesmo que simples | |  | |
| PES-2.1 | 2 | Os papéis de hunting estão descritos? O time os conhece? | Descrição de papéis; matriz de responsabilidades | |  | |
| PES-2.2 | 2 | Como se define o que cada papel precisa saber? | Matriz de competências com níveis por papel | |  | |
| PES-2.3 | 2 | Quando foi a última avaliação das competências do time? | Registro da avaliação | |  | |
| PES-3.1 | 3 | O que acontece quando uma lacuna de competência é identificada? | Plano de desenvolvimento; registros de mentoria | |  | |
| PES-3.2 | 3 | Como cada nível de hunter é treinado? | Plano de treinamento; registros de participação | |  | |
| PES-3.3 | 3 | Como o time planeja contratações para hunting? | Plano de recrutamento ou de dimensionamento | |  | |
| PES-4.1 | 4 | Com que frequência as competências são avaliadas, e por quem? | Calendário e registros das avaliações e autoavaliações | |  | |
| PES-4.2 | 4 | O time de hunting treina e é treinado por outras áreas? | Registros de treinamentos cruzados | |  | |

## Processo (PRO)

| ID | Nível | Pergunta | Evidência esperada | Cliente | Com o serviço | Observações |
|----|:-----:|----------|--------------------|:-------:|:-------------:|-------------|
| PRO-1.1 | 1 | Em que situações o hunting acontece hoje? | Exemplos recentes de hunts e seus gatilhos | |  | |
| PRO-1.2 | 1 | Existe algum roteiro de como um hunt é feito? | Roteiro, anotações ou procedimento informal | |  | |
| PRO-1.3 | 1 | Que parte do ambiente já foi objeto de hunting? | Lista de ambientes ou ativos cobertos | |  | |
| PRO-2.1 | 2 | O processo de hunting está formalizado? Onde? | Processo documentado (ex.: equivalente a `01-processo/`) | |  | |
| PRO-2.2 | 2 | Há hunts planejados mesmo sem incidente em curso? | Plano de ciclos; registros de execução de hunts planejados | |  | |
| PRO-2.3 | 2 | Que percentual dos ativos está no escopo do hunting? | Escopo documentado; inventário de ativos cobertos | |  | |
| PRO-3.1 | 3 | Qual é a cadência dos hunts? Ela é cumprida? | Planos e registros dos últimos ciclos | |  | |
| PRO-3.2 | 3 | Quem recebe os resultados dos hunts e como eles são discutidos? | Relatórios periódicos; atas de reuniões de resultados | |  | |
| PRO-3.3 | 3 | Os hunts são documentados e revisados? Avalia-se se viram regra? | Hunts documentados com revisão; avaliação de automação registrada | |  | |
| PRO-3.4 | 3 | Que percentual dos ativos está no escopo do hunting? | Escopo documentado; inventário de ativos cobertos | |  | |
| PRO-4.1 | 4 | Quando o processo de hunting foi revisado pela última vez? | Registro da revisão e das mudanças decorrentes | |  | |
| PRO-4.2 | 4 | Qual é a missão do hunting? O time sabe dizê-la? | Declaração de missão; entrevista com o time | |  | |
| PRO-4.3 | 4 | Os destinatários dos resultados avaliam o hunting? Como? | Pesquisa de satisfação ou registros de retorno | | (igual ao cliente) | |
| PRO-4.4 | 4 | Quantos hunts viraram detecções nos últimos 12 meses? | Lista de regras derivadas de hunts | |  | |
| PRO-4.5 | 4 | O time compartilha hunts com a comunidade? | Contribuições públicas ou em comunidades de compartilhamento | |  | |
| PRO-4.6 | 4 | Que percentual dos ativos está no escopo do hunting? | Escopo documentado; inventário de ativos cobertos | |  | |

## Fontes de dados (DAD)

| ID | Nível | Pergunta | Evidência esperada | Cliente | Com o serviço | Observações |
|----|:-----:|----------|--------------------|:-------:|:-------------:|-------------|
| DAD-1.1 | 1 | Que fontes de dados existem hoje? | Lista de fontes, mesmo que informal | |  | |
| DAD-1.2 | 1 | Que dados são coletados e onde ficam? | Plataformas de coleta em operação | | (igual ao cliente) | |
| DAD-1.3 | 1 | Que parte do ambiente gera dados coletados? | Inventário ou estimativa de cobertura | | (igual ao cliente) | |
| DAD-2.1 | 2 | Existe um inventário formal das fontes? | Inventário de fontes de dados (ex.: `04-onboarding/`) | |  | |
| DAD-2.2 | 2 | Alguém avalia se os dados estão completos e atualizados? | Registros de avaliação ou de problemas de qualidade | |  | |
| DAD-2.3 | 2 | Quem mantém a coleta e como falhas são tratadas? | Responsável definido; registros de manutenção da coleta | | (igual ao cliente) | |
| DAD-2.4 | 2 | Que percentual dos ativos gera dados para hunting? | Inventário com cobertura por fonte | | (igual ao cliente) | |
| DAD-3.1 | 3 | Há critérios formais de qualidade de dados? São aplicados? | Inventário com avaliação de qualidade por critério | |  | |
| DAD-3.2 | 3 | Houve interrupções de coleta nos últimos meses? Como foram tratadas? | Monitoramento da coleta; registros de interrupções | | (igual ao cliente) | |
| DAD-3.3 | 3 | O hunting vai além de buscas simples? Usa análise estatística? | Hunts com stacking, baseline ou análise estatística | |  | |
| DAD-3.4 | 3 | Os ativos críticos geram dados para hunting? | Inventário com cobertura dos ativos críticos | | (igual ao cliente) | |
| DAD-4.1 | 4 | Existe uma política ou padrão de logging para toda a organização? | Política de logging | | (igual ao cliente) | |
| DAD-4.2 | 4 | Os dados chegam padronizados ou dependem de tratamento manual? | Pipeline de normalização em operação | | (igual ao cliente) | |
| DAD-4.3 | 4 | Técnicas de análise de dados são usadas em todo ciclo? | Registros de execução dos últimos ciclos | |  | |
| DAD-4.4 | 4 | Que percentual dos ativos é coberto em rede e em endpoint? | Inventário com cobertura por categoria | | (igual ao cliente) | |

## Threat intel (INT)

| ID | Nível | Pergunta | Evidência esperada | Cliente | Com o serviço | Observações |
|----|:-----:|----------|--------------------|:-------:|:-------------:|-------------|
| INT-1.1 | 1 | De onde vem a inteligência usada hoje? Como circula? | Exemplos de inteligência recebida e repassada | |  | |
| INT-1.2 | 1 | A inteligência considera ameaças específicas do ambiente? | Relatórios ou alertas de inteligência | |  | |
| INT-1.3 | 1 | A inteligência alimenta alguma ferramenta (SIEM, firewall, proxy, IDS/IPS)? | Integração em operação | |  | |
| INT-2.1 | 2 | Quem é responsável pela inteligência? O que se espera dessa função? | Definição da função e das entregas | |  | |
| INT-2.2 | 2 | Quais ameaças do setor estão sendo acompanhadas? | Avaliação de ameaças por setor | |  | |
| INT-2.3 | 2 | A inteligência gera hipóteses de hunting? | Hunts com origem em inteligência | |  | |
| INT-2.4 | 2 | Como os indicadores são gerenciados e distribuídos? | Plataforma de inteligência e integrações | |  | |
| INT-3.1 | 3 | A inteligência considera ameaças da região? | Fontes e relatórios regionais | |  | |
| INT-3.2 | 3 | De quais comunidades de compartilhamento a organização participa? | Comprovação de participação | |  | |
| INT-3.3 | 3 | Que tarefas de inteligência são automatizadas? | Automações em operação | |  | |
| INT-3.4 | 3 | A resposta a incidentes recebe contexto de inteligência automaticamente? | Integrações em operação | |  | |
| INT-3.5 | 3 | Em incidentes recentes, a inteligência ajudou a entender o adversário? | Relatórios de incidentes com contexto de inteligência | |  | |
| INT-4.1 | 4 | A liderança usa a inteligência para decidir? Em que situações? | Exemplos de decisões apoiadas por inteligência | | (igual ao cliente) | |
| INT-4.2 | 4 | O time produz inteligência própria, além de consumir a de terceiros? | Relatórios de inteligência produzidos internamente | |  | |
| INT-4.3 | 4 | Que ferramentas ou processos de inteligência foram desenvolvidos pelo time? | Ferramentas ou processos próprios | |  | |
| INT-4.4 | 4 | Que parte do ciclo de inteligência depende de trabalho manual? | Automações e orquestrações em operação | |  | |

## Métricas (MET)

| ID | Nível | Pergunta | Evidência esperada | Cliente | Com o serviço | Observações |
|----|:-----:|----------|--------------------|:-------:|:-------------:|-------------|
| MET-1.1 | 1 | Que métricas de hunting existem hoje? | Lista de métricas e fonte dos dados | |  | |
| MET-1.2 | 1 | Quem recebe as métricas e quando? | Exemplos de relatórios com métricas | |  | |
| MET-2.1 | 2 | Quais são as metas do hunting? A gestão as conhece? | Metas documentadas (ex.: GOST); comunicação à gestão | |  | |
| MET-3.1 | 3 | Como e com que frequência as métricas são revisadas? | Registros de revisão das métricas | |  | |
| MET-3.2 | 3 | Com que frequência a gestão recebe os resultados? | Relatórios periódicos dos últimos ciclos | |  | |
| MET-4.1 | 4 | O que acontece quando uma meta não é atingida? | Plano de ação ligado às metas | |  | |
| MET-4.2 | 4 | As métricas dependem de compilação manual? | Painel ou relatório gerado automaticamente | |  | |
| MET-4.3 | 4 | A gestão de riscos considera os resultados do hunting? | Avaliação de riscos que cita resultados de hunts | | (igual ao cliente) | |

Na coluna **Com o serviço**, critérios marcados no modelo como "Cliente" repetem a resposta da visão do cliente, porque só ele pode atendê-los.

---

Adaptado de: OTHF, módulo 6 — Operational (Maturity e Assessment Criteria). Licença MIT, ver [`referencias/OTHF/`](../referencias/OTHF/).
