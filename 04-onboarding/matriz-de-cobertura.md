# Matriz de cobertura

A matriz de cobertura cruza as **fontes de dados que cada hunt exige** (seção 5 do hunt) com o **inventário de fontes do cliente**. O resultado responde a duas perguntas:

- quais hunts da biblioteca podem ser executados neste cliente;
- quais fontes de dados, se corrigidas ou incluídas, viabilizariam mais hunts.

A matriz preenchida descreve o ambiente do cliente e fica **fora deste repositório** ([ADR 0003](../decisoes/0003-sem-dados-reais-no-repositorio.md)).

## 1. Viabilidade de um hunt

| Estado | Critério | Tratamento |
|--------|----------|------------|
| **Viável** | Todas as fontes **obrigatórias** do hunt existem, com retenção suficiente e qualidade **Adequada** | Entra no backlog do cliente |
| **Parcial** | Todas as fontes obrigatórias existem, mas pelo menos uma tem qualidade **Parcial** ou retenção menor que a exigida | Entra no backlog do cliente, com a limitação registrada; resultados "refutados" devem ser lidos com cautela |
| **Inviável** | Falta uma fonte obrigatória, ou ela tem qualidade **Inadequada** | Não entra no backlog; vira **recomendação de visibilidade** |

Fontes **complementares** não mudam a viabilidade. Elas melhoram a triagem e devem ser registradas na matriz como "complementar ausente" quando faltarem.

## 2. Como montar

1. Liste os hunts da biblioteca que podem ser aplicados ao escopo do cliente.
2. Para cada hunt, copie as fontes obrigatórias e complementares da seção 5.
3. Para cada fonte, consulte o [inventário do cliente](template-inventario-fontes.md): existe? qual a qualidade? a retenção atende?
4. Classifique o hunt como Viável, Parcial ou Inviável, conforme a seção 1.
5. Para cada fonte ausente ou inadequada, conte quantos hunts ela viabilizaria. Essa contagem ordena as recomendações de visibilidade.

## 3. Template

### 3.1 Viabilidade por hunt

| Hunt | Fontes obrigatórias | Situação das obrigatórias | Complementares ausentes | Viabilidade | Observações |
|------|---------------------|---------------------------|-------------------------|-------------|-------------|
| TH-EXF-001 | Logs do resolvedor DNS interno | <existe, qualidade, retenção> | | <Viável / Parcial / Inviável> | |
| | | | | | |

### 3.2 Recomendações de visibilidade

| Fonte ausente ou inadequada | Problema | Hunts que viabilizaria | Prioridade |
|-----------------------------|----------|------------------------|------------|
| | <ausente / qualidade inadequada / retenção insuficiente / sem host de origem> | <lista e quantidade> | <Alta / Média / Baixa> |

A prioridade considera a quantidade de hunts viabilizados e a relevância das técnicas desses hunts para o cliente (ver a [avaliação de ameaças](../01-processo/identificacao-de-hunts.md#3-avaliação-de-ameaças-por-cliente)).

## 4. Mapa de referência: fontes por hunt

Categorias de fonte que cada hunt da biblioteca exige. Use como ponto de partida para estimar a cobertura **antes** do inventário detalhado, por exemplo, numa proposta comercial.

**Legenda:** **O** = obrigatória · **O\*** = obrigatória, atendida por qualquer uma das fontes marcadas com O\* na linha · **C** = complementar · — = não usada

| Hunt | Status | DNS | Proxy e firewall | Endpoint (EDR) | Logs do SO | Autenticação e diretório | Acesso remoto |
|------|--------|:---:|:----------------:|:--------------:|:----------:|:------------------------:|:-------------:|
| TH-EXF-001 — Exfiltração via DNS | Proposto | O | C | C | — | — | — |
| TH-INI-001 — Acesso remoto externo de origem incomum | Planejado | — | — | — | — | C | O |
| TH-EXE-001 — Interpretadores com comandos codificados | Planejado | — | — | O\* | O\* | — | — |
| TH-PER-001 — Tarefas e serviços fora do padrão | Planejado | — | — | O\* | O\* | — | — |
| TH-STL-001 — Utilitários legítimos executando código | Planejado | — | C | O | — | — | — |
| TH-DIM-001 — Desativação de ferramentas e logs | Planejado | — | — | O\* | O\* | — | — |
| TH-CRD-001 — Acesso à memória do processo de autenticação | Planejado | — | — | O | — | — | — |
| TH-DIS-001 — Enumeração do diretório em rajada | Planejado | — | — | C | — | O | — |
| TH-LAT-001 — Movimentação lateral com contas administrativas | Planejado | — | — | C | — | O | — |
| TH-COL-001 — Compactação de grandes volumes de arquivos | Planejado | — | C | O | — | — | — |
| TH-C2-001 — Beaconing para destinos raros | Planejado | C | O | C | — | — | — |
| TH-IMP-001 — Preparação para ransomware | Planejado | — | — | O\* | O\* | — | — |

Notas:

- Para hunts **Planejados**, o mapa é uma previsão. As fontes definitivas são as da seção 5 de cada hunt, quando ele for publicado; atualize o mapa nesse momento.
- As categorias **E-mail** e **Nuvem** ainda não são usadas pelos hunts planejados, mas fazem parte do inventário porque serão necessárias para hunts futuros.
- A telemetria de **endpoint** é a fonte mais requisitada da biblioteca: aparece em 11 dos 12 hunts e é obrigatória, sozinha ou como alternativa aos logs do sistema operacional, em 7. Sem ela, 3 hunts ficam inviáveis (TH-STL-001, TH-CRD-001 e TH-COL-001) e outros 4 passam a depender inteiramente dos logs do sistema operacional.

---

Baseado em: OTHF, módulo 4 — Preparation (dados e dicionário de dados). Licença MIT, ver [`referencias/OTHF/`](../referencias/OTHF/).
