<!--
TEMPLATE DE HUNT — v1.0
Como usar:
1. Copie este arquivo para 03-biblioteca/<tatica>/<id-em-minusculas>-<descricao-curta>.md
2. Preencha todas as seções. Seção que não se aplica: escreva "Não se aplica" e explique em uma linha.
3. Apague todos os comentários HTML (blocos de orientação como este) antes de abrir o pull request.
4. Nenhum dado real de cliente. Use os dados fictícios definidos no CONTRIBUTING.md.
Guia de preenchimento: 02-templates/README.md
-->

# TH-XXX-000 — <Título curto do hunt>

| Campo | Valor |
|-------|-------|
| **ID** | TH-XXX-000 |
| **Versão** | 0.1 |
| **Status** | Rascunho |
| **Tática principal** | <Tática MITRE ATT&CK> |
| **Plataformas com consulta validada** | <Nenhuma / USM Anywhere / CrowdStrike> |
| **Esforço estimado por execução** | <ex.: 1–2 h por cliente> |
| **Autor** | <nome> |
| **Revisor** | <nome> |
| **Criado em** | AAAA-MM-DD |
| **Última revisão** | AAAA-MM-DD |

## 1. Objetivo

<!--
Uma frase SMART: o que o hunt quer descobrir, onde e em qual período.
Ex.: "Identificar, nos últimos 30 dias, hosts Windows que enviaram volumes anômalos de dados via DNS para domínios externos."
-->

## 2. Origem e contexto

<!--
Por que este hunt existe. Marque a origem e explique o contexto em poucas linhas.
Cite relatórios ou fontes de inteligência públicas; nunca informações de clientes.
-->

**Origem:** <Inteligência de ameaças | Vulnerabilidade/exploit | Superfície de ataque | Incidente anterior (anonimizado) | Pentest/red team | Observação do ambiente | Requisito regulatório>

**Contexto:**

**Ameaças/grupos associados:** <ex.: grupos ou famílias de malware conhecidos por usar a técnica, se houver>

## 3. Hipótese

<!--
Formato "Dado / Quando / Então". A hipótese precisa ser testável com os dados listados na seção 5.
-->

- **Dado que** <contexto do ambiente ou do adversário>,
- **quando** <o adversário executa a técnica>,
- **então** <o que deve aparecer nos dados>.

## 4. Mapeamento MITRE ATT&CK

| Tática | Técnica / Subtécnica | ID |
|--------|----------------------|----|
| | | |

## 5. Fontes de dados necessárias

<!--
Liste o que o cliente precisa ter para o hunt ser viável. Isso alimenta a matriz de cobertura do onboarding.
Obrigatória = sem ela o hunt não pode ser executado. Complementar = melhora o resultado.
-->

| Fonte | Eventos / campos relevantes | Retenção mínima | Obrigatória? |
|-------|-----------------------------|-----------------|--------------|
| | | | |

**Escopo de ativos:** <ex.: estações Windows, servidores Linux, controladores de domínio, tenant de nuvem>

## 6. Comportamento normal (baseline)

<!--
O que é esperado e legítimo no ambiente para esta atividade. Sem conhecer o normal, não é possível reconhecer o anômalo.
Inclua como levantar o baseline de um cliente novo.
-->

## 7. Estratégia de busca

<!--
Lógica do hunt, independente de plataforma. Indique a(s) técnica(s) de análise usadas:
stacking, enriquecimento, agrupamento, palavra-chave, IOC, análise temporal.
Descreva em passos o que buscar, como filtrar o normal e o que destacar.
-->

**Técnicas de análise:** <ex.: stacking + enriquecimento>

**Lógica:**

1.
2.
3.

## 8. Consultas por plataforma

<!--
Uma subseção por plataforma. Status: Validada / TODO: validar.
Consulta com "TODO: validar" não pode ser executada em cliente.
Registre a data e a plataforma em que foi validada.
-->

### 8.1 USM Anywhere (LevelBlue)

**Status:** TODO: validar
**Onde executar:** <ex.: Activity > Events>

```text
<consulta ou filtros>
```

**Observações:**

### 8.2 CrowdStrike Falcon / Next-Gen SIEM

**Status:** TODO: validar
**Onde executar:** <ex.: Next-Gen SIEM > Advanced Event Search>

```text
<consulta>
```

**Observações:**

## 9. Validação do hunt

<!--
Como foi comprovado que o hunt encontra o que promete.
Dados de teste: ex. teste do Atomic Red Team, script de laboratório, simulação manual.
Validação histórica: execução em período longo para medir ruído.
-->

| Item | Detalhe |
|------|---------|
| **Dados de teste usados** | <ex.: Atomic Red Team T0000 teste N> |
| **Ambiente de teste** | <ex.: laboratório interno> |
| **Resultado com dados de teste** | <detectou / não detectou> |
| **Período histórico testado** | <ex.: 30 dias> |
| **Ruído observado** | <baixo / médio / alto, e por quê> |
| **Data da validação** | AAAA-MM-DD |

## 10. Triagem dos resultados

<!--
Como o analista classifica cada resultado. Seja concreto: o que olhar, o que perguntar ao cliente.
-->

| Classificação | Critérios |
|---------------|-----------|
| **Benigno** | |
| **Suspeito — investigar** | |
| **Malicioso — escalar** | |

**Perguntas de enriquecimento:**

- <ex.: o processo é assinado? o destino tem reputação ruim? o usuário costuma fazer isso?>

## 11. Falsos positivos conhecidos

| Origem | Como identificar | Tratamento |
|--------|------------------|------------|
| | | |

## 12. Ação ao encontrar atividade maliciosa

<!--
O que fazer quando o resultado for classificado como malicioso.
Deve seguir o processo de escalonamento do SOC (01-processo/) e a matriz de responsabilidades com o cliente.
-->

1. Interromper o hunt e preservar as evidências no sistema de atendimento do SOC (nunca neste repositório).
2. Escalar conforme o processo de escalonamento do SOC.
3. <ações específicas desta técnica, ex.: contenção sugerida, artefatos a coletar>

## 13. Outros achados possíveis

<!--
Hunts frequentemente revelam problemas que não são ataques. Liste o que este hunt pode revelar
e como reportar: configuração incorreta, protocolo inseguro, ativo desconhecido, ponto cego de logs etc.
-->

## 14. Potencial de automação

<!--
Avalie se o hunt pode virar regra de detecção ("hunt once").
-->

| Critério | Avaliação |
|----------|-----------|
| **Candidato a detecção?** | <Sim / Não / Parcialmente> |
| **Ruído permite alerta contínuo?** | |
| **Plataforma(s) alvo** | |
| **Regra criada** | <nome/ID da regra, quando existir> |

## 15. Referências

-

## 16. Histórico de versões

| Versão | Data | Autor | Alteração |
|--------|------|-------|-----------|
| 0.1 | AAAA-MM-DD | | Criação |

---

Estrutura baseada em: OTHF, módulo 6 — Operational (processo de hunting) e módulo 7 — Measurement. Licença MIT, ver `referencias/OTHF/`.
