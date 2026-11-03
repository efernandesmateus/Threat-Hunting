# Backlog e priorização

O time sempre terá mais ideias de hunts do que tempo para executá-las. Este documento define como organizar e priorizar o trabalho.

## 1. Os dois backlogs

| Backlog | Contém | Responsável | Onde fica |
|---------|--------|-------------|-----------|
| **Da biblioteca** | Hunts a **desenvolver** ou revisar | Líder de Hunting | Pode ficar no próprio projeto (ex.: issues do repositório), pois não contém dados de clientes |
| **Por cliente** | Hunts Ativos a **executar** naquele cliente | Hunter Sênior responsável pelo cliente | Nos sistemas de atendimento de quem executa o serviço, **nunca** neste repositório |

Os backlogs se alimentam: uma técnica prioritária para um cliente sem hunt na biblioteca gera um item no backlog da biblioteca; um hunt que fica Ativo pode entrar no backlog de todos os clientes que têm as fontes de dados necessárias.

## 2. Critério eliminatório: viabilidade dos dados

Antes de pontuar, verifica-se se o cliente tem as fontes de dados **obrigatórias** do hunt (seção 5 do template), com a retenção mínima exigida.

- **Viável:** o hunt segue para a pontuação.
- **Inviável:** o hunt não entra no backlog do cliente. A fonte ausente é registrada como **recomendação de visibilidade** no relatório periódico.

## 3. Pontuação de prioridade

Cada item recebe uma nota de 1 a 5 em três critérios:

| Critério | 1 | 3 | 5 |
|----------|---|---|---|
| **Relevância (R)** — a técnica é usada por adversários que atacam o setor do cliente? | Sem registro de uso no setor | Usada por alguns grupos relevantes | Usada por vários grupos relevantes ou em campanha ativa |
| **Impacto (I)** — qual o dano se a técnica for usada com sucesso? | Baixo, localizado | Moderado | Grave: ativos críticos, dados sensíveis, interrupção do negócio |
| **Esforço (E)** — quanto custa executar o hunt? | Muito alto (vários dias, muito ruído) | Moderado | Baixo (poucas horas, pouco ruído) |

> O **esforço** é pontuado de forma invertida: quanto **menor** o esforço, **maior** a nota.

```text
Prioridade = (R × 2) + (I × 2) + E
```

Relevância e impacto têm peso dobrado porque refletem o risco; o esforço serve para desempatar. A prioridade varia de 5 a 25.

| Prioridade | Faixa | Tratamento |
|------------|-------|------------|
| **Alta** | 20 a 25 | Entra no próximo ciclo |
| **Média** | 13 a 19 | Entra conforme a capacidade do ciclo |
| **Baixa** | 5 a 12 | Permanece no backlog e é reavaliada a cada ciclo |

### Ajustes à pontuação

Alguns fatores, citados pelo OTHF, podem elevar a prioridade independentemente da nota:

- **Requisito regulatório** de detecção aplicável ao cliente.
- **Incidente anterior** com a mesma técnica no cliente ou no setor.
- **Caminho comprovado** em pentest ou red team no ambiente do cliente.
- **Técnica que sustenta várias hipóteses** ao mesmo tempo.

Ajustes devem ser justificados no registro do item.

### Pirâmide da Dor

A [Pirâmide da Dor](https://detect-respond.blogspot.com/2013/03/the-pyramid-of-pain.html), de David Bianco, mostra que alguns indicadores custam muito mais para o adversário trocar do que outros:

| Camada (do topo para a base) | Custo para o adversário trocar |
|------------------------------|--------------------------------|
| TTPs | Muito alto |
| Ferramentas | Alto |
| Artefatos de rede e de host | Moderado |
| Domínios | Baixo |
| Endereços IP | Muito baixo |
| Hashes | Trivial |

Hunts baseados nas **três camadas superiores** (TTPs, ferramentas e artefatos) continuam úteis mesmo quando o adversário troca sua infraestrutura. Hunts baseados apenas em IOCs (domínios, IPs, hashes) são rápidos, mas envelhecem rápido. Por isso:

- em empate de pontuação, prefira o hunt das camadas superiores;
- hunts por IOC são válidos, principalmente para inteligência recente, mas não devem dominar o backlog.

### Itens que não sobem

Um item de prioridade baixa **não é removido** do backlog. Técnicas pouco usadas hoje podem voltar a ser usadas, e a ausência de atividade de um grupo não garante que ele não esteja presente. Os itens são reavaliados a cada ciclo.

## 4. Cadência por nível de oferta

| Nível | Hunts por ciclo (referência) | Composição | Hunts extraordinários |
|-------|-----------------------------|------------|-----------------------|
| **Essencial** | Conforme contrato | Hunts da biblioteca, pelos de maior prioridade | Não incluídos |
| **Avançado** | Conforme contrato | Biblioteca + hunts orientados por inteligência recente | Incluídos |
| **Estratégico** | Conforme contrato | Biblioteca + inteligência + hunts personalizados | Incluídos, com prioridade |

A quantidade de hunts por ciclo é definida em contrato, a partir do [modelo de dimensionamento](../00-servico/objetivos-e-capacidade.md#3-modelo-de-dimensionamento).

### Rotação

Para não executar sempre os mesmos hunts:

- hunts já executados no cliente voltam ao backlog com a prioridade recalculada;
- recomenda-se que todo hunt viável seja executado pelo menos uma vez a cada 12 meses em cada cliente, exceto os que já viraram detecção (status **Automatizado**).

### Hunts extraordinários

São hunts executados **fora do planejamento do ciclo**, por um gatilho urgente: vulnerabilidade crítica explorada ativamente, campanha em curso contra o setor, alerta de um CERT.

- Avaliados pelo Líder de Hunting assim que o gatilho surge.
- Se o hunt ainda não existe na biblioteca, desenvolve-se uma **versão mínima** (hipótese, fontes, consulta e triagem), marcada como **Em validação**. Ela é completada depois, pelo processo normal.
- A execução de um hunt **Em validação** é uma **exceção** à regra do [ciclo de vida](../02-templates/README.md#ciclo-de-vida-campo-status), permitida apenas para hunts extraordinários, com autorização do Líder de Hunting registrada na execução. Os resultados devem ser triados com cautela redobrada, já que o ruído do hunt ainda não é conhecido.
- Podem substituir um hunt planejado do ciclo, desde que isso seja registrado.

---

Baseado em: OTHF, módulo 6 — Operational (Hunt Tempo, Prioritization e Scheduling). Licença MIT, ver [`referencias/OTHF/`](../referencias/OTHF/).
