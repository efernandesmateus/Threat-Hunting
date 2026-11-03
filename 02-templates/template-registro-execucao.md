<!--
TEMPLATE DE REGISTRO DE EXECUÇÃO — v1.0
Um registro por execução de hunt em um cliente.
Este arquivo é só o MODELO. O registro preenchido contém dados do cliente e fica
nos sistemas de atendimento de quem executa o serviço, NUNCA neste repositório (ADR 0003).
Processo: 01-processo/execucao-de-hunts.md
-->

# Registro de execução — <ID da execução>

## 1. Identificação

| Campo | Valor |
|-------|-------|
| **ID da execução** | <código único, ex.: EXE-AAAA-MM-NNN> |
| **Cliente** | <identificação no sistema de atendimento> |
| **Hunt e versão** | <ex.: TH-EXF-001 v1.0> |
| **Status operacional do hunt** | <Ativo / Em validação (somente hunt extraordinário, com autorização)> |
| **Autorização** | <obrigatória se o hunt estiver Em validação: quem autorizou e quando> |
| **Ciclo** | <ex.: AAAA-MM> |
| **Tipo de execução** | <Planejada / Extraordinária / Avulsa> |
| **Analista** | <nome> |
| **Revisor** | <nome> |

## 2. Execução

| Campo | Valor |
|-------|-------|
| **Data da execução** | AAAA-MM-DD |
| **Duração** | <horas> |
| **Esforço estimado no hunt** | <horas, copiado do cabeçalho do hunt> |
| **Período analisado** | AAAA-MM-DD a AAAA-MM-DD |
| **Tipo de plataforma** | <SIEM / EDR / XDR> |
| **Consultas usadas** | <sem alteração / com ajustes (descrever na seção 6)> |

## 3. Fontes de dados verificadas

| Fonte | Disponível? | Cobertura do período | Observações |
|-------|-------------|----------------------|-------------|
| | <Sim / Parcial / Não> | <completa / com lacunas> | |

## 4. Resultado

**Resultado da hipótese:** <Confirmada / Refutada / Inconclusiva>

**Justificativa:** <uma ou duas frases. Se refutada, por que os dados permitiam encontrar. Se inconclusiva, o que impediu a conclusão.>

| Classificação | Quantidade |
|---------------|------------|
| Benigno | |
| Suspeito (pendente) | |
| Malicioso | |

## 5. Achados

| # | Descrição resumida | Classificação | Severidade | Escalonado? | Referência (chamado ou incidente) |
|---|--------------------|---------------|------------|-------------|-----------------------------------|
| 1 | | | <Crítica / Alta / Média / Baixa> | <Sim / Não> | |

### Outros achados

| # | Tipo | Descrição | Severidade | Recomendação |
|---|------|-----------|------------|--------------|
| 1 | <configuração incorreta / lacuna de visibilidade / protocolo inseguro / ativo desconhecido / outro> | | | |

### Pendências com o cliente

| # | Pergunta ou ação pendente | Enviada em | Responsável no cliente |
|---|---------------------------|------------|------------------------|
| 1 | | | |

## 6. Lições aprendidas

| Área | Observação | Ação proposta |
|------|------------|---------------|
| Hunt (consultas, limiares, triagem) | | <ex.: revisão do hunt, nova exceção> |
| Dados | | |
| Processo | | |
| Tempo (real × estimado) | | |

## 7. Evidências

<Referência ao local das evidências no sistema de atendimento. Não anexar evidências a este registro se ele circular fora desse sistema.>
