<!--
TEMPLATE DE FICHA DO CLIENTE — v1.0
Registra os acordos do onboarding. Preenchida na Fase 1 e confirmada na Fase 6 do checklist.
Este arquivo é só o MODELO. A ficha preenchida contém dados do cliente e fica nos
sistemas de atendimento de quem executa o serviço, NUNCA neste repositório (ADR 0003).
Mantenha a ficha atualizada: ela é consultada em todo escalonamento.
-->

# Ficha do cliente — <Cliente>

| Campo | Valor |
|-------|-------|
| **Cliente** | <nome> |
| **Nível do serviço** | <Essencial / Avançado / Estratégico> |
| **Início do contrato** | AAAA-MM-DD |
| **Hunter Sênior responsável** | <nome> |
| **Data do aceite do onboarding** | AAAA-MM-DD |
| **Última atualização da ficha** | AAAA-MM-DD |

## 1. Escopo

| Item | Detalhe |
|------|---------|
| **Ambientes incluídos** | <ex.: rede corporativa, ambiente de nuvem, unidades> |
| **Ambientes excluídos** | |
| **Tipos de ativos incluídos** | <ex.: estações, servidores, controladores de domínio> |
| **Requisitos regulatórios ou contratuais** | |

## 2. Contatos

| Papel | Nome | Canal principal | Canal alternativo | Disponibilidade |
|-------|------|-----------------|-------------------|-----------------|
| Ponto focal técnico | | | | |
| Responsável por decisões em incidentes | | | | <ex.: 24x7 / horário comercial> |
| Contato alternativo para incidentes | | | | |
| Patrocinador | | | | |

## 3. Escalonamento

| Item | Acordo |
|------|--------|
| **Canal para achados Críticos** | <ex.: telefone + registro escrito> |
| **Canal para achados Altos** | |
| **Prazo de notificação de achado malicioso** | <referência: até 2 horas após a confirmação, ou valor do contrato> |
| **Prazo de consulta sobre achado suspeito** | <referência: até 1 dia útil> |
| **Procedimento se o cliente não for alcançado** | <ordem de tentativa dos contatos e regra contratual> |

## 4. Modo de resposta

**Modo contratado:** <Somente notificação / Contenção pelo SOC>

### Ações pré-autorizadas (somente no modo contenção pelo SOC)

| Ação | Condições | Aprovado por | Data |
|------|-----------|--------------|------|
| <ex.: isolar estação de trabalho> | <ex.: achado Crítico confirmado> | | |

### Ativos que nunca podem ser contidos sem aprovação

| Ativo ou grupo de ativos | Motivo | Quem aprova a contenção |
|--------------------------|--------|-------------------------|
| | | |

## 5. Acessos do SOC

| Analista | Plataforma (tipo) | Permissão | Concedido em | Próxima revisão |
|----------|-------------------|-----------|--------------|-----------------|
| | <SIEM / EDR / XDR> | <somente leitura / outra (justificar)> | AAAA-MM-DD | AAAA-MM-DD |

## 6. Janela para atividades ativas

<Quando e com que aviso prévio o SOC pode executar ações ativas no ambiente, se previstas.>

## 7. Histórico de alterações

| Data | Alteração | Responsável |
|------|-----------|-------------|
| | | |
