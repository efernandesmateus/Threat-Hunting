<!--
TEMPLATE DE RELATÓRIO PERIÓDICO AO CLIENTE — v1.0
Um relatório por cliente, por ciclo. Entregue até 5 dias úteis após o fim do ciclo (valor de referência, ver 00-servico/escopo.md).
Duas camadas: PARTE A (executiva, uma página, para o patrocinador) e PARTE B (técnica, para o ponto focal).
O relatório preenchido contém dados do cliente e fica fora deste repositório (ADR 0003).
Fonte dos dados: registros de execução do ciclo (02-templates/template-registro-execucao.md).
Apague todos os comentários HTML antes de enviar.
-->

# Relatório de Threat Hunting — <Cliente> — <Ciclo AAAA-MM>

| Campo | Valor |
|-------|-------|
| **Cliente** | <nome> |
| **Ciclo** | <AAAA-MM> (período analisado: AAAA-MM-DD a AAAA-MM-DD) |
| **Nível do serviço** | <Essencial / Avançado / Estratégico> |
| **Elaborado por** | <Hunter Sênior> |
| **Revisado por** | <Líder de Hunting> |
| **Data de emissão** | AAAA-MM-DD |
| **Classificação** | Confidencial — uso restrito ao cliente |

---

# PARTE A — Resumo executivo

<!--
Uma página. Linguagem de negócio, sem jargão técnico desnecessário.
Responda: o que foi feito, o que foi encontrado, o que exige atenção, o que melhorou.
-->

## Visão geral do ciclo

| Indicador | Este ciclo | Ciclo anterior |
|-----------|-----------:|---------------:|
| Hunts executados | | |
| Achados maliciosos (Crítica/Alta) | | |
| Outros achados (Média/Baixa) | | |
| Lacunas de visibilidade encontradas | | |
| Recomendações anteriores atendidas | | |

<!-- No primeiro ciclo, deixe a coluna "Ciclo anterior" com "—". -->

## Principais pontos

1. <o achado ou conclusão mais importante do ciclo, em uma ou duas frases>
2. <segundo ponto>
3. <terceiro ponto>

## Ações recomendadas

| Prioridade | Ação | Por quê |
|------------|------|---------|
| Alta | | |
| Média | | |

## Sobre os resultados

Threat hunting reduz o risco, mas não prova a ausência de comprometimento. Uma hipótese **refutada** indica que a atividade procurada não foi encontrada nos dados disponíveis, no período analisado. Os resultados dependem das fontes de dados disponíveis; as lacunas encontradas estão listadas na seção B.5.

---

# PARTE B — Detalhamento técnico

## B.1 Escopo do ciclo

| Item | Detalhe |
|------|---------|
| **Período analisado** | AAAA-MM-DD a AAAA-MM-DD |
| **Hunts planejados** | <quantidade> |
| **Hunts executados** | <quantidade> |
| **Hunts extraordinários** | <quantidade e motivo, se houver> |
| **Hunts não executados** | <quantidade e motivo: fonte indisponível, retenção, acesso> |

### Fontes de dados utilizadas

| Fonte | Situação no ciclo |
|-------|-------------------|
| | <disponível / com lacunas / indisponível> |

## B.2 Resultados por hunt

| Hunt | Técnicas ATT&CK | Resultado | Achados | Observação |
|------|-----------------|-----------|---------|------------|
| <ID — título> | <IDs e nomes> | <Confirmada / Refutada / Inconclusiva> | <quantidade> | |

<!--
Confirmada: a atividade da hipótese foi encontrada (maliciosa ou não).
Refutada: os dados, com qualidade suficiente, não mostram a atividade no período.
Inconclusiva: não foi possível concluir (dados incompletos, retenção, pendência). Explique o motivo.
-->

## B.3 Achados

<!-- Achados Crítica/Alta já foram notificados durante o ciclo. Aqui se registra o consolidado e o status atual. -->

| # | Descrição | Severidade | Técnicas | Data da notificação | Status |
|---|-----------|------------|----------|---------------------|--------|
| 1 | | <Crítica / Alta> | | AAAA-MM-DD | <em tratamento / contido / encerrado> |

Para cada achado, um resumo:

### Achado 1 — <título>

- **O que foi encontrado:**
- **Ativos envolvidos:**
- **Período:**
- **Ações executadas:**
- **Recomendações pendentes:**

## B.4 Outros achados e recomendações

| # | Achado | Severidade | Recomendação | Hunt de origem |
|---|--------|------------|--------------|----------------|
| 1 | | <Média / Baixa> | | |

## B.5 Lacunas de visibilidade

| Lacuna | Impacto | O que a correção viabilizaria |
|--------|---------|-------------------------------|
| <ex.: logs DNS sem host de origem> | <hunts afetados ou inconclusivos> | <ex.: hunts de exfiltração e C2 via DNS> |

## B.6 Pendências com o cliente

| # | Pendência | Enviada em | Impacto |
|---|-----------|------------|---------|
| 1 | | | <ex.: resultado inconclusivo do hunt X> |

## B.7 Evolução

<!-- A partir do segundo ciclo. Mostre tendência, não só números do mês. -->

| Indicador | Ciclo -2 | Ciclo -1 | Este ciclo | Tendência |
|-----------|---------:|---------:|-----------:|-----------|
| Achados maliciosos | | | | |
| Lacunas de visibilidade abertas | | | | |
| Hunts viáveis no ambiente | | | | |
| Recomendações atendidas (acumulado) | | | | |
| Detecções derivadas de hunts (acumulado) | | | | |

### Técnicas caçadas

<!--
Liste as técnicas ATT&CK já investigadas no cliente. NÃO apresente como "% de cobertura":
ter caçado uma técnica não significa estar protegido contra ela.
-->

| Técnica | Último hunt | Resultado mais recente |
|---------|-------------|------------------------|
| | | |

## B.8 Próximo ciclo

- **Hunts previstos:** <lista ou quantidade>
- **Dependências do cliente:** <acessos, fontes de dados, respostas pendentes>

## Anexo — Detalhes técnicos

<!-- Opcional. Consultas executadas em alto nível, exemplos anonimizados, referências. Sem dados desnecessários. -->

---

Estrutura baseada em: OTHF, módulo 7 — Measurement e módulo 8 — Communication. Licença MIT, ver [`referencias/OTHF/`](../referencias/OTHF/).
