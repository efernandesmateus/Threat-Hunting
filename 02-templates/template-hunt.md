<!--
TEMPLATE DE HUNT — v1.1
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
| **Status** | Rascunho <!-- Rascunho / Proposto / Validado pela comunidade / Descontinuado --> |
| **Tática principal** | <Tática MITRE ATT&CK> |
| **Tipos de plataforma com consulta validada** | <Nenhum / SIEM / EDR / XDR> |
| **Versão do ATT&CK** | <ex.: v20> |
| **Esforço estimado por execução** | <ex.: 1–2 h por cliente> |
| **Autor** | <nome> |
| **Revisor** | <nome> |
| **Criado em** | AAAA-MM-DD |
| **Última revisão** | AAAA-MM-DD |

## 1. Objetivo

<!--
Uma frase SMART: o que o hunt quer descobrir, onde e em qual período.
Ex.: "Identificar, nos últimos 30 dias, estações de trabalho que enviaram volumes anômalos de dados via DNS para domínios externos."
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

**Escopo de ativos:** <ex.: estações de trabalho, servidores, controladores de domínio, ambiente de nuvem>

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

## 8. Consultas

<!--
Formato definido no ADR 0005. Não cite fabricantes nem produtos (ADR 0004).
Status: Validada / Não validada. Consulta não validada deve ser testada no ambiente de quem adota antes do uso em clientes (ADR 0006).
-->

### 8.1 Pseudoconsulta

**Status:** Não validada

```text
FONTE:      <fonte de dados e período>
FILTRO:     <condições de inclusão e exclusão>
AGREGAÇÃO:  <agrupar por / contar / calcular>
LIMIAR:     <condição para destacar o resultado (valor de referência, calibrar pelo baseline)>
SAÍDA:      <campos a apresentar para a triagem>
```

### 8.2 Regra Sigma

**Status:** Não validada
**Arquivo:** `<id-em-minusculas>-<descricao-curta>.yml`, na mesma pasta do hunt. Se a lógica não couber em Sigma, escreva "Não se aplica" e explique.

### 8.3 Implementações por plataforma

As traduções para plataformas específicas ficam fora do repositório. Registre aqui apenas em que **tipos** de plataforma a lógica foi validada, e quando.

| Tipo de plataforma | Data da validação | Observações |
|--------------------|-------------------|-------------|
| | | |

## 9. Validação do hunt

<!--
Os hunts da biblioteca são validados por quem os adota (ADR 0006).
9.1 orienta como validar; 9.2 registra as validações contribuídas.
Não cite fabricantes nem produtos; informe apenas o tipo de plataforma.
-->

### 9.1 Como validar no seu ambiente

**Dados de teste (somente em laboratório ou com autorização formal):**

1. <como reproduzir a técnica para gerar dados de teste>
2. <o que confirmar para garantir que o teste representa a técnica real>

**Validação histórica:** executar o hunt em pelo menos <período> de dados reais, com autorização, para medir o ruído e calibrar os limiares.

### 9.2 Validações registradas

| Tipo de plataforma | Data | Dados de teste | Detectou? | Período histórico | Ruído | Ajustes sugeridos |
|--------------------|------|----------------|-----------|-------------------|-------|-------------------|
| | | | | | | |

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
