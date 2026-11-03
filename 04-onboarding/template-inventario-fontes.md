<!--
TEMPLATE DE INVENTÁRIO DE FONTES DE DADOS — v1.0
Preenchido na Fase 3 do checklist de onboarding e atualizado a cada mudança de fonte.
Este arquivo é só o MODELO. O inventário preenchido descreve o ambiente do cliente e fica
fora deste repositório (ADR 0003).
Descreva as fontes pelo TIPO, sem fabricantes nem produtos (ADR 0004).
-->

# Inventário de fontes de dados — <Cliente>

**Data do levantamento:** AAAA-MM-DD
**Responsável:** <analista>

## Fontes

| Categoria | Fonte | Onde está disponível | Cobertura de ativos | Identifica o host de origem real? | Retenção pesquisável | Completude | Atualidade | Consistência | Precisão | Qualidade geral | Observações |
|-----------|-------|----------------------|---------------------|-----------------------------------|----------------------|------------|------------|--------------|----------|-----------------|-------------|
| <ver categorias abaixo> | <ex.: logs do resolvedor DNS interno> | <SIEM / EDR / XDR / outra> | <ex.: todos os hosts / só servidores / 80% das estações> | <Sim / Não / Parcial> | <dias> | <A / P / I> | <A / P / I> | <A / P / I> | <A / P / I> | <Adequada / Parcial / Inadequada> | |

**Legenda:** A = Adequada · P = Parcial · I = Inadequada

## Campos-chave por fonte

Para cada fonte, registre os campos usados pelos hunts e se estão presentes.

| Fonte | Campo | Presente? | Observação |
|-------|-------|-----------|------------|
| <ex.: logs do resolvedor DNS> | <ex.: nome consultado completo> | <Sim / Não> | |

## Categorias de fonte

| Categoria | Exemplos de fonte |
|-----------|-------------------|
| **DNS** | Logs do resolvedor DNS interno |
| **Proxy e firewall** | Logs de conexões de saída, com destino, porta, protocolo e volume |
| **Endpoint (EDR)** | Telemetria de processos, linha de comando, arquivos, conexões, acesso a processos |
| **Logs do sistema operacional** | Eventos de auditoria, criação de tarefas e serviços, limpeza de logs |
| **Autenticação e diretório** | Logons, emissão de tíquetes, alterações em contas e grupos, consultas ao diretório |
| **Acesso remoto** | Logs de serviços de acesso remoto externo (VPN, gateways) |
| **E-mail** | Logs de transporte e de entrega de mensagens |
| **Nuvem** | Logs de auditoria e de atividade de serviços de nuvem |

## Critérios de qualidade

Cada fonte é avaliada em quatro critérios, com três níveis.

| Critério | Pergunta | Adequada | Parcial | Inadequada |
|----------|----------|----------|---------|------------|
| **Completude** | Os eventos e campos necessários estão presentes para todos os ativos do escopo? | Todos os ativos e campos | Faltam alguns ativos ou campos não essenciais | Faltam ativos relevantes ou campos essenciais |
| **Atualidade** | Os eventos chegam a tempo e sem atrasos? | Chegam em minutos | Atrasos ocasionais | Atrasos frequentes ou interrupções |
| **Consistência** | O formato e os valores são estáveis ao longo do tempo e entre ativos? | Estáveis | Variações pontuais | Variações que impedem a análise |
| **Precisão** | Os valores refletem a realidade (horário, origem, identidade)? | Precisos | Erros pontuais (ex.: relógio desajustado em alguns ativos) | Erros sistemáticos (ex.: origem sempre registrada como o intermediário) |

**Qualidade geral da fonte:**

- **Adequada:** os quatro critérios adequados.
- **Parcial:** nenhum critério inadequado, mas pelo menos um parcial.
- **Inadequada:** pelo menos um critério inadequado.

## Lacunas encontradas

| Lacuna | Fonte afetada | Impacto nos hunts | Recomendação |
|--------|---------------|-------------------|--------------|
| | | | |

---

Critérios de qualidade adaptados de: OTHF, módulo 4 — Preparation. Licença MIT, ver [`referencias/OTHF/`](../referencias/OTHF/).
