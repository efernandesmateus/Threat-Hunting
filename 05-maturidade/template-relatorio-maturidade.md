<!--
TEMPLATE DE RELATÓRIO DE MATURIDADE — v1.0
Resultado do assessment, conforme o modelo de maturidade e o questionário.
O relatório preenchido contém dados do cliente e fica fora deste repositório (ADR 0003).
Apague todos os comentários HTML antes de enviar.
-->

# Assessment de maturidade de threat hunting — <Cliente>

| Campo | Valor |
|-------|-------|
| **Cliente** | <nome> |
| **Data do assessment** | AAAA-MM-DD |
| **Tipo** | <Inicial / Reavaliação / Avulso> |
| **Escopo avaliado** | <ambientes e ativos considerados> |
| **Avaliadores** | <nomes> |
| **Participantes do cliente** | <papéis> |
| **Assessment anterior** | <data ou "não há"> |
| **Classificação** | Confidencial — uso restrito ao cliente |

## 1. Resumo executivo

<!-- Meia página. O perfil, os pontos fortes, a principal limitação e as três recomendações mais importantes. -->

**Principais pontos:**

1.
2.
3.

> O resultado é um **perfil por eixo**, sem nota geral: a fraqueza de um eixo limita os demais e ficaria escondida em um número único.

## 2. Perfil de maturidade

Níveis: 0 Inexistente · 1 Inicial · 2 Definido · 3 Gerenciado · 4 Otimizado

| Eixo | Visão do cliente | Com o serviço | Assessment anterior (com o serviço) |
|------|:----------------:|:-------------:|:-----------------------------------:|
| Pessoas | | | |
| Processo | | | |
| Fontes de dados | | | |
| Threat intel | | | |
| Métricas | | | |

<!--
Visão do cliente: apenas a capacidade própria do cliente.
Com o serviço: capacidade combinada do cliente e do SOC.
A diferença entre as colunas mostra o que o serviço acrescenta.
No assessment avulso (sem contrato de serviço), preencha apenas a visão do cliente.
-->

## 3. Detalhamento por eixo

<!-- Repita esta seção para cada eixo. -->

### <Eixo>

- **Nível na visão do cliente:** <N> — <nome do nível>
- **Nível com o serviço:** <N> — <nome do nível>
- **Pontos fortes:** <critérios atendidos que merecem destaque>
- **Critérios que impedem o próximo nível:**

| ID | Critério | Quem pode atender | Situação | Recomendação |
|----|----------|-------------------|----------|--------------|
| | | <Cliente / Cliente ou SOC> | <Não / Parcial> | |

## 4. Recomendações priorizadas

<!--
Priorize primeiro os eixos que limitam os outros. Em geral, Fontes de dados vem antes:
sem dados, os demais eixos não evoluem na prática.
-->

| Prioridade | Recomendação | Eixo e critérios | Depende de | Prazo sugerido |
|------------|--------------|------------------|------------|----------------|
| Alta | | | <Cliente / SOC / Ambos> | |
| Média | | | | |
| Baixa | | | | |

## 5. Evolução

<!-- A partir da segunda avaliação. -->

| Eixo | Anterior | Atual | Variação | Principais mudanças |
|------|:--------:|:-----:|:--------:|---------------------|
| Pessoas | | | | |
| Processo | | | | |
| Fontes de dados | | | | |
| Threat intel | | | | |
| Métricas | | | | |

## 6. Próxima avaliação

- **Data prevista:** AAAA-MM-DD
- **Metas de nível até lá:** <eixo: nível pretendido>

## Anexo — Respostas e evidências

<!-- Referência ao questionário preenchido e ao local das evidências. Não anexe evidências sensíveis a este relatório. -->

---

Estrutura baseada em: OTHF, módulo 6 — Operational (Maturity) e módulo 8 — Communication. Licença MIT, ver [`referencias/OTHF/`](../referencias/OTHF/).
