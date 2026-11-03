# Objetivos e capacidade

## 1. Modelo GOST

O OTHF recomenda o modelo **GOST** para dar clareza ao que o time quer alcançar e a como pretende fazer isso:

| Componente | Pergunta | Natureza |
|------------|----------|----------|
| **Goals** (metas) | O que queremos alcançar? | Qualitativa, ampla |
| **Objectives** (objetivos) | Como saberemos que alcançamos? | Quantitativa, mensurável |
| **Strategy** (estratégia) | Qual é o plano geral? | Direção |
| **Tactics** (táticas) | Quais ações concretas vamos executar? | Execução |

## 2. GOST de referência do serviço

Os objetivos abaixo são **exemplos de referência**. Cada time deve definir os próprios valores no início do ano ou do contrato, a partir da sua realidade, e revisá-los periodicamente.

### Meta 1 — Encontrar adversários que escaparam das detecções

| Componente | Referência |
|------------|------------|
| **Objetivos** | Executar 100% dos hunts planejados no ciclo para cada cliente. Notificar 100% dos achados maliciosos dentro do prazo de referência. |
| **Estratégia** | Hunts priorizados pelas técnicas mais usadas pelos adversários do setor de cada cliente. |
| **Táticas** | Manter backlog priorizado por cliente; executar os hunts conforme o [processo](../01-processo/); revisar a priorização a cada ciclo. |

### Meta 2 — Melhorar continuamente as detecções

| Componente | Referência |
|------------|------------|
| **Objetivos** | Avaliar o potencial de automação de 100% dos hunts ativos. Converter em detecção os hunts aprovados para automação. |
| **Estratégia** | Abordagem "hunt once": um padrão encontrado manualmente não deve depender de nova execução manual para ser detectado. |
| **Táticas** | Preencher a seção de potencial de automação de cada hunt; registrar as regras criadas; acompanhar o ruído das novas detecções. |

### Meta 3 — Aumentar a visibilidade dos clientes

| Componente | Referência |
|------------|------------|
| **Objetivos** | Reportar 100% das lacunas de visibilidade encontradas. Aumentar, a cada ciclo, a quantidade de hunts viáveis por cliente. |
| **Estratégia** | Usar a matriz de cobertura do onboarding para mostrar o que cada nova fonte de dados viabiliza. |
| **Táticas** | Registrar a fonte ausente sempre que um hunt não puder ser executado; incluir recomendações no relatório periódico. |

### Meta 4 — Expandir e manter a biblioteca

| Componente | Referência |
|------------|------------|
| **Objetivos** | Adicionar novos hunts ativos a cada trimestre. Revisar cada hunt ativo pelo menos uma vez por ano. |
| **Estratégia** | Priorizar técnicas de alto impacto e alta frequência, cobrindo as táticas do MITRE ATT&CK de forma progressiva. |
| **Táticas** | Manter um backlog de hunts a desenvolver; aplicar o ciclo de vida e a revisão por pares do [template](../02-templates/README.md). |

As métricas detalhadas para acompanhar esses objetivos fazem parte do template de relatório e métricas, em [`02-templates/`](../02-templates/).

## 3. Modelo de dimensionamento

O modelo estima quantas horas o serviço exige e quantos clientes um time comporta. Ele usa variáveis, não valores fixos: cada time deve preencher com os próprios números e recalibrar com o esforço real registrado nas execuções.

### Variáveis

| Variável | Significado | Fonte |
|----------|-------------|-------|
| `H` | Quantidade de hunts por cliente por ciclo | Nível contratado e backlog |
| `E` | Esforço médio por execução de hunt (horas) | Campo "Esforço estimado por execução" dos hunts; depois, média real |
| `T` | Horas de triagem e investigação de achados por ciclo | Histórico do cliente |
| `R` | Horas de relatório e reuniões por ciclo | Nível contratado |
| `O` | Horas de onboarding (somente no início) | Complexidade do ambiente |
| `D` | Horas por analista por ciclo dedicadas a hunting | Escala do time |
| `F` | Fator de eficiência (0 a 1) | Desconta interrupções, treinamento, reuniões internas. Referência: 0,7 |

### Fórmulas

Esforço por cliente, por ciclo:

```text
Esforço_cliente = (H × E) + T + R
```

Capacidade do time, por ciclo:

```text
Capacidade = Nº_analistas × D × F
```

Quantidade de clientes que o time comporta:

```text
Clientes ≈ Capacidade ÷ média(Esforço_cliente)
```

O onboarding (`O`) deve ser somado à parte nos ciclos em que houver clientes novos.

### Exemplo ilustrativo

Valores fictícios, apenas para mostrar o cálculo:

| Variável | Valor |
|----------|-------|
| H | 4 hunts por mês |
| E | 2 h |
| T | 3 h |
| R | 3 h |
| Analistas | 2 |
| D | 80 h por mês (metade da jornada dedicada a hunting) |
| F | 0,7 |

```text
Esforço_cliente = (4 × 2) + 3 + 3 = 14 h por mês
Capacidade      = 2 × 80 × 0,7   = 112 h por mês
Clientes        ≈ 112 ÷ 14       = 8 clientes
```

### Referências do OTHF para a dedicação ao hunting

O OTHF sugere, para times internos:

- organizações pequenas: começar com **4 a 6 horas por analista por semana** e ampliar conforme os resultados forem documentados;
- organizações grandes: dedicar de **10% a 20%** do time de SOC/CIRT a hunting.

Em um MSSP, o hunting é um serviço vendido, então a dedicação deve vir do dimensionamento acima, e não de um percentual fixo.

---

Baseado em: OTHF, módulo 2 — Foundational (estratégia e GOST) e módulo 3 — Structure (dimensionamento). Licença MIT, ver [`referencias/OTHF/`](../referencias/OTHF/).
