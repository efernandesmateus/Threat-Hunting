# Modelo de maturidade

Modelo para avaliar a maturidade de threat hunting de um cliente, em **cinco eixos** e **cinco níveis (0 a 4)**. É uma adaptação do modelo de maturidade do OTHF ao contexto de serviço: os critérios foram reescritos como requisitos verificáveis, as inconsistências do original foram corrigidas e cada critério indica quem pode atendê-lo.

Use junto com o [questionário](questionario-assessment.md) e o [template de relatório](template-relatorio-maturidade.md).

## 1. Níveis

| Nível | Nome | Ideia geral |
|-------|------|-------------|
| 0 | Inexistente | Hunting não existe ou é desconhecido |
| 1 | Inicial | Hunting acontece de forma informal e pontual |
| 2 | Definido | Hunting é formalizado, documentado e proativo |
| 3 | Gerenciado | Hunting é executado com regularidade, medido e integrado às demais áreas |
| 4 | Otimizado | Hunting é revisado continuamente, automatizado onde possível e orienta decisões do negócio |

Os nomes dos níveis são deste projeto; o OTHF os identifica apenas por número.

## 2. Regras de avaliação

1. **Critérios cumulativos.** Um eixo está no nível **N** quando **todos** os critérios dos níveis 1 a N estão atendidos. Atender parte dos critérios de um nível não conta: o eixo permanece no nível anterior. O nível 0 não tem critérios: é o ponto de partida.
2. **Evidência obrigatória.** Todo critério atendido precisa de evidência (documento, registro ou demonstração), conforme o [questionário](questionario-assessment.md). Sem evidência, o critério conta como **não atendido**.
3. **Sem nota geral.** O resultado é o **perfil** dos cinco eixos. Não se calcula média nem nota única: a fraqueza de um eixo limita os demais e ficaria escondida em um número.
4. **Percentuais de cobertura.** Os percentuais de Processo e de Fontes de dados são os de referência do OTHF e se aplicam aos ativos **do escopo** acordado com o cliente.

## 3. Duas visões

Quando o cliente contrata o serviço, parte da capacidade de hunting passa a ser fornecida pelo SOC. Por isso, o assessment produz duas visões:

| Visão | O que avalia | Quando usar |
|-------|--------------|-------------|
| **Cliente** | Apenas a capacidade própria do cliente, sem o serviço | Diagnóstico inicial; proposta comercial; clientes que só contratam o assessment |
| **Com o serviço** | A capacidade combinada do cliente e do SOC | Acompanhamento ao longo do contrato |

A coluna **Quem pode atender** indica, para cada critério:

- **Cliente ou SOC:** na visão com o serviço, o critério pode ser atendido pelo SOC (por exemplo, um processo de hunting documentado e executado em cadência).
- **Cliente:** só o cliente pode atender, em qualquer visão (por exemplo, coleta de dados no próprio ambiente ou decisões do negócio).

A diferença entre as duas visões mostra, de forma objetiva, o que o serviço acrescenta. Critérios marcados como **Cliente** e não atendidos são as recomendações que dependem do cliente para evoluir.

## Pessoas (PES)

Quem faz o hunting e com que preparo.

| ID | Nível | Critério | Quem pode atender |
|----|:-----:|----------|-------------------|
| PES-1.1 | 1 | Alguém executa atividades de hunting, mesmo sem papel formal | Cliente ou SOC |
| PES-1.2 | 1 | Existe uma lista, ainda que informal, das competências necessárias | Cliente ou SOC |
| PES-2.1 | 2 | Os papéis de hunting são dedicados, com responsabilidades documentadas e comunicadas | Cliente ou SOC |
| PES-2.2 | 2 | O catálogo de competências tem critérios e requisitos definidos por papel | Cliente ou SOC |
| PES-2.3 | 2 | As competências do time são avaliadas, ao menos ocasionalmente | Cliente ou SOC |
| PES-3.1 | 3 | Existe um plano formal para cobrir lacunas de competência, com mentoria | Cliente ou SOC |
| PES-3.2 | 3 | Existe programa de treinamento para todos os níveis de hunter | Cliente ou SOC |
| PES-3.3 | 3 | Existe plano formal de recrutamento para hunting | Cliente ou SOC |
| PES-4.1 | 4 | As competências são avaliadas regularmente, pela liderança e por autoavaliação | Cliente ou SOC |
| PES-4.2 | 4 | Existe treinamento cruzado com outras áreas (monitoramento, resposta a incidentes, inteligência) | Cliente ou SOC |

## Processo (PRO)

Como o hunting é planejado, executado e melhorado.

| ID | Nível | Critério | Quem pode atender |
|----|:-----:|----------|-------------------|
| PRO-1.1 | 1 | Hunting acontece, ao menos de forma reativa (por exemplo, motivado por incidentes) | Cliente ou SOC |
| PRO-1.2 | 1 | O método de hunting está documentado, ainda que de forma informal | Cliente ou SOC |
| PRO-1.3 | 1 | O hunting cobre parte do ambiente | Cliente ou SOC |
| PRO-2.1 | 2 | O método de hunting é formal e documentado | Cliente ou SOC |
| PRO-2.2 | 2 | Hunting é proativo e contínuo, independentemente de incidentes | Cliente ou SOC |
| PRO-2.3 | 2 | O hunting cobre pelo menos 50% do ambiente | Cliente ou SOC |
| PRO-3.1 | 3 | O processo é executado regularmente, em cadência definida | Cliente ou SOC |
| PRO-3.2 | 3 | Os resultados são discutidos de forma consistente com as partes afetadas | Cliente ou SOC |
| PRO-3.3 | 3 | Todo hunt é documentado, revisado e avaliado para conversão em detecção | Cliente ou SOC |
| PRO-3.4 | 3 | O hunting cobre pelo menos 75% do ambiente | Cliente ou SOC |
| PRO-4.1 | 4 | O processo é revisado regularmente quanto à eficiência | Cliente ou SOC |
| PRO-4.2 | 4 | A missão de hunting está definida e é compreendida pelo time | Cliente ou SOC |
| PRO-4.3 | 4 | O retorno das partes interessadas confirma que os resultados atendem às expectativas | Cliente |
| PRO-4.4 | 4 | Hunts maduros são convertidos em detecções de forma sistemática | Cliente ou SOC |
| PRO-4.5 | 4 | Hunts desenvolvidos são compartilhados com a comunidade de segurança | Cliente ou SOC |
| PRO-4.6 | 4 | O hunting cobre pelo menos 90% do ambiente | Cliente ou SOC |

## Fontes de dados (DAD)

O que pode ser visto, com que qualidade e cobertura.

| ID | Nível | Critério | Quem pode atender |
|----|:-----:|----------|-------------------|
| DAD-1.1 | 1 | A visibilidade das fontes de dados é ao menos parcialmente conhecida | Cliente ou SOC |
| DAD-1.2 | 1 | Existe coleta de dados de segurança | Cliente |
| DAD-1.3 | 1 | As fontes cobrem parte do ambiente | Cliente |
| DAD-2.1 | 2 | As fontes disponíveis para hunting estão formalmente documentadas | Cliente ou SOC |
| DAD-2.2 | 2 | A visibilidade e a qualidade das fontes são avaliadas, ainda que informalmente | Cliente ou SOC |
| DAD-2.3 | 2 | A coleta é ativa e mantida como parte do programa de hunting | Cliente |
| DAD-2.4 | 2 | As fontes cobrem pelo menos 50% do ambiente | Cliente |
| DAD-3.1 | 3 | A visibilidade e a qualidade são medidas formalmente, com critérios definidos | Cliente ou SOC |
| DAD-3.2 | 3 | A coleta é executada de forma consistente, sem interrupções relevantes | Cliente |
| DAD-3.3 | 3 | O hunting usa técnicas de análise de dados (estatística, detecção de anomalias) | Cliente ou SOC |
| DAD-3.4 | 3 | As fontes cobrem pelo menos 75% do ambiente, incluindo os ativos críticos | Cliente |
| DAD-4.1 | 4 | Existe um padrão corporativo de registro de logs, documentado | Cliente |
| DAD-4.2 | 4 | A padronização e a normalização das fontes são automatizadas | Cliente |
| DAD-4.3 | 4 | A análise de dados é parte rotineira da operação de hunting | Cliente ou SOC |
| DAD-4.4 | 4 | As fontes cobrem pelo menos 90% do ambiente, em rede e em endpoint | Cliente |

## Threat intel (INT)

Como a inteligência de ameaças orienta o hunting.

| ID | Nível | Critério | Quem pode atender |
|----|:-----:|----------|-------------------|
| INT-1.1 | 1 | Inteligência de ameaças é coletada e compartilhada, ao menos de forma ad hoc | Cliente ou SOC |
| INT-1.2 | 1 | O escopo da inteligência inclui ameaças globais e específicas da organização | Cliente ou SOC |
| INT-1.3 | 1 | Existem fontes de inteligência atualizadas, integradas a pelo menos uma ferramenta de defesa | Cliente ou SOC |
| INT-2.1 | 2 | Inteligência é uma função própria, com expectativas formalmente documentadas | Cliente ou SOC |
| INT-2.2 | 2 | O escopo inclui ameaças do setor | Cliente ou SOC |
| INT-2.3 | 2 | Dados internos e externos são combinados para orientar hunts proativos | Cliente ou SOC |
| INT-2.4 | 2 | Existe plataforma de inteligência com gestão de indicadores, em integração com as ferramentas de defesa | Cliente ou SOC |
| INT-3.1 | 3 | O escopo inclui ameaças regionais | Cliente ou SOC |
| INT-3.2 | 3 | A organização participa de comunidades de compartilhamento (ISACs, ISAOs) | Cliente ou SOC |
| INT-3.3 | 3 | Parte da análise de inteligência é automatizada | Cliente ou SOC |
| INT-3.4 | 3 | A inteligência está integrada às ferramentas de defesa e ao sistema de resposta a incidentes | Cliente ou SOC |
| INT-3.5 | 3 | A inteligência apoia a resposta a incidentes com conhecimento sobre os adversários | Cliente ou SOC |
| INT-4.1 | 4 | A inteligência orienta decisões operacionais e estratégicas do negócio | Cliente |
| INT-4.2 | 4 | O time produz inteligência tática e estratégica própria | Cliente ou SOC |
| INT-4.3 | 4 | O time desenvolve ferramentas e processos próprios de inteligência | Cliente ou SOC |
| INT-4.4 | 4 | A maior parte do ciclo de inteligência é automatizada, com análise avançada e orquestração | Cliente ou SOC |

## Métricas (MET)

Como o hunting é medido e como os resultados são usados.

| ID | Nível | Critério | Quem pode atender |
|----|:-----:|----------|-------------------|
| MET-1.1 | 1 | As métricas-chave estão identificadas e são medidas com precisão | Cliente ou SOC |
| MET-1.2 | 1 | As métricas são reportadas, ao menos de forma ad hoc | Cliente ou SOC |
| MET-2.1 | 2 | Metas de desempenho e indicadores estão definidos e são comunicados à gestão | Cliente ou SOC |
| MET-3.1 | 3 | As métricas são acompanhadas formalmente e revisadas | Cliente ou SOC |
| MET-3.2 | 3 | Os resultados são reportados à gestão em periodicidade definida | Cliente ou SOC |
| MET-4.1 | 4 | Melhorias são priorizadas onde o desempenho não atinge as metas | Cliente ou SOC |
| MET-4.2 | 4 | As métricas operacionais são atualizadas automaticamente | Cliente ou SOC |
| MET-4.3 | 4 | Os resultados dos hunts são incorporados à avaliação de riscos da organização | Cliente |

## Diferenças em relação ao original

- Os critérios foram reescritos como **requisitos positivos e cumulativos**. No original, alguns níveis descrevem ausências (por exemplo, "não há programa de treinamento"), o que impede somar os níveis.
- Foram removidas inconsistências do original: no nível 3 de Processo, a afirmação de que a missão "ainda não foi definida"; no nível 4 de Pessoas, a ressalva herdada do nível anterior sobre avaliações ocasionais.
- Referências a categorias específicas de ferramentas foram generalizadas, conforme o [ADR 0004](../decisoes/0004-neutralidade-de-fabricante.md).
- Foram acrescentadas as **duas visões** e a coluna **Quem pode atender**, para o contexto de serviço.

---

Adaptado de: OTHF, módulo 6 — Operational (Maturity e Assessment Criteria). Licença MIT, ver [`referencias/OTHF/`](../referencias/OTHF/).
