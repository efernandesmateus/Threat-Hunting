# Ofertas do serviço

O serviço é oferecido em três níveis contínuos e em serviços avulsos. Os valores de cadência e quantidade abaixo são **referências**: cada contrato pode ajustá-los, desde que o ajuste fique registrado no escopo do cliente.

Preços não fazem parte desta metodologia. O esforço de cada nível pode ser estimado pelo modelo em [objetivos e capacidade](objetivos-e-capacidade.md#3-modelo-de-dimensionamento).

## 1. Níveis contínuos

### Essencial

Para clientes que querem iniciar hunting de forma estruturada, com foco nas técnicas mais comuns.

- Hunts da biblioteca, escolhidos de acordo com as fontes de dados do cliente.
- Cadência mensal.
- Relatório consolidado mensal.
- Escalonamento de achados maliciosos conforme o [escopo](escopo.md).
- Recomendações básicas de visibilidade: indicação das fontes de dados ausentes que impediram a execução de hunts.

### Avançado

Para clientes que querem hunts alinhados às ameaças atuais e evolução contínua das detecções.

Inclui tudo do **Essencial**, mais:

- Hunts orientados por inteligência: novas campanhas, vulnerabilidades exploradas ativamente, técnicas em alta.
- Hunts extraordinários quando surge uma ameaça crítica relevante para o cliente.
- Recomendação de conversão de hunts em regras de detecção ("hunt once").
- Recomendações de melhoria de visibilidade (fontes de dados, retenção, configuração de logs).

### Estratégico

Para clientes com maior maturidade ou maior exposição a ameaças direcionadas.

Inclui tudo do **Avançado**, mais:

- Hunts personalizados, desenvolvidos a partir das ameaças do setor e dos ativos críticos do cliente.
- Assessment de maturidade de threat hunting no início do contrato e reavaliação anual.
- Reuniões periódicas de resultados com o cliente.
- Implementação das detecções derivadas dos hunts, quando o SOC tiver acesso administrativo às plataformas.

## 2. Comparativo

| Item | Essencial | Avançado | Estratégico |
|------|:---------:|:--------:|:-----------:|
| Hunts da biblioteca | ✔ | ✔ | ✔ |
| Cadência de referência | Mensal | Mensal | Mensal + sob demanda |
| Relatório consolidado | Mensal | Mensal | Mensal |
| Escalonamento de achados maliciosos | ✔ | ✔ | ✔ |
| Hunts orientados por inteligência | — | ✔ | ✔ |
| Hunts extraordinários para ameaças críticas | — | ✔ | ✔ |
| Recomendação de novas detecções | — | ✔ | ✔ |
| Recomendações de visibilidade | Básicas | ✔ | ✔ |
| Hunts personalizados por setor e ativos críticos | — | — | ✔ |
| Assessment de maturidade | Avulso | Avulso | ✔ Incluído |
| Reuniões periódicas de resultados | — | — | ✔ |
| Implementação de detecções | — | — | ✔ (se houver acesso) |

## 3. Serviços avulsos

| Serviço | Descrição | Quando usar |
|---------|-----------|-------------|
| **Assessment de maturidade** | Avaliação da maturidade de threat hunting do cliente (ver [`05-maturidade/`](../05-maturidade/)). | Antes de contratar um nível contínuo, ou para clientes que querem só o diagnóstico. |
| **Hunt pontual** | Execução de um ou mais hunts específicos, fora de um contrato contínuo. | Uma nova ameaça relevante, uma vulnerabilidade crítica divulgada, uma suspeita específica. |

> **Atenção:** o hunt pontual **não substitui** um compromise assessment ou uma resposta a incidentes. Se houver indício concreto de comprometimento, o caso segue para resposta a incidentes (ver [escopo](escopo.md#3-exclusões)).

## 4. Pré-requisito comum a todos os níveis

Todo cliente passa pelo **onboarding** (ver [`04-onboarding/`](../04-onboarding/)) antes do primeiro hunt. Sem o inventário de fontes de dados, não é possível saber quais hunts são viáveis.
