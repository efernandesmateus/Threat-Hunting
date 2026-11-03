# Threat Hunting — metodologia aberta para SOCs e MSSPs

Metodologia aberta para estruturar e operar um **serviço de threat hunting** em SOCs e provedores de serviços gerenciados de segurança (MSSPs): definição do serviço, processo, templates, biblioteca de hunts, onboarding de clientes e avaliação de maturidade.

> **Status:** em construção (v0.4.0). Veja o [CHANGELOG](CHANGELOG.md) e o [roadmap](#roadmap).

## O que é threat hunting neste projeto

Threat hunting é a busca **proativa**, **orientada por hipóteses** e **repetível** por sinais de atividade maliciosa que **não geraram alertas** nas ferramentas de segurança.

Ele complementa o monitoramento contínuo, sem substituí-lo:

| Monitoramento (SOC)                     | Threat hunting                                        |
|-----------------------------------------|-------------------------------------------------------|
| Reativo: parte de um alerta             | Proativo: parte de uma hipótese                       |
| Responde ao que as regras já detectam   | Procura o que as regras ainda não detectam            |
| Resultado: triagem e resposta ao alerta | Resultado: achados, lacunas de visibilidade e novas detecções |

## Para quem é

- **MSSPs e MDRs** que querem oferecer threat hunting como serviço a vários clientes.
- **SOCs internos** que querem sair do hunting ad hoc para um processo estruturado.
- **Analistas** que querem uma base prática para formular, executar e documentar hunts.

## Base metodológica

O projeto parte do **Open Threat Hunting Framework (OTHF)**, da TactiKoolSec, licenciado sob MIT. O OTHF foi escrito para um time interno de uma única organização; aqui ele é **adaptado para o modelo de serviço**, com vários clientes, escopo contratado e relatórios ao cliente.

Resumo de cada módulo do OTHF e como ele foi aproveitado: [`referencias/OTHF/`](referencias/OTHF/).

## Estrutura do repositório

| Pasta | Conteúdo | Origem no OTHF |
|-------|----------|----------------|
| [`00-servico/`](00-servico/) | Definição do serviço, missão, ofertas, papéis, divisão SOC x cliente | 2 Foundational, 3 Structure |
| [`01-processo/`](01-processo/) | Processo operacional de hunting, cadência, priorização, melhoria contínua | 6 Operational |
| [`02-templates/`](02-templates/) | Template de hunt, template de relatório ao cliente | 6 Operational, 7 Measurement, 8 Communication |
| [`03-biblioteca/`](03-biblioteca/) | Hunts prontos e reutilizáveis, organizados por tática MITRE ATT&CK | 10 Data Sources, 11 Techniques |
| [`04-onboarding/`](04-onboarding/) | Checklist de entrada do cliente e inventário de fontes de dados | 4 Preparation |
| [`05-maturidade/`](05-maturidade/) | Assessment de maturidade de threat hunting do cliente | 6 Operational (Maturity) |
| [`decisoes/`](decisoes/) | Registro das decisões de design do projeto (ADRs) | — |
| [`referencias/`](referencias/) | Material de referência externo e licenças | — |

## Neutralidade de fabricante

A metodologia é **agnóstica de ferramenta** e **não cita fabricantes nem produtos**. Ela descreve o que buscar e por quê, em termos de comportamento, fontes de dados e técnicas MITRE ATT&CK, para que possa ser aplicada em qualquer plataforma de SIEM, EDR ou XDR (ver [ADR 0004](decisoes/0004-neutralidade-de-fabricante.md)).

O projeto também **não depende de API de fornecedor** para funcionar (ver [ADR 0002](decisoes/0002-independencia-de-api-de-fornecedor.md)).

## Sobre os hunts da biblioteca

Os hunts são publicados como **Proposto**: completos, com consultas de sintaxe verificada, mas **não validados em plataforma real** pelo projeto. Eles são pontos de partida e devem ser validados no ambiente de quem os adota antes do uso em produção (ver [ADR 0006](decisoes/0006-validacao-pelo-adotante.md)). Validações contribuídas pela comunidade são bem-vindas.

## Regra de ouro: nenhum dado real

Este repositório guarda **metodologia**, nunca dados operacionais. Não inclua nomes de organizações, logs reais, IPs, hostnames, domínios, usuários, credenciais ou evidências. Detalhes em [CONTRIBUTING.md](CONTRIBUTING.md) e no [ADR 0003](decisoes/0003-sem-dados-reais-no-repositorio.md).

## Roadmap

**Metodologia (até a v1.0.0)**

- [x] Fundação: estrutura, convenções, decisões iniciais, referência ao OTHF
- [x] Template de hunt
- [x] Definição do serviço
- [x] Processo operacional
- [x] Primeiro hunt da biblioteca (exfiltração via DNS)
- [ ] Template de relatório e métricas
- [ ] Onboarding de cliente
- [ ] Assessment de maturidade

**Biblioteca (a partir da v1.1.0)**

- [ ] 11 hunts planejados, em blocos de três por versão (ver [`03-biblioteca/`](03-biblioteca/README.md#hunts-planejados))

## Como contribuir

Sugestões, correções e novos hunts são bem-vindos via issues e pull requests. Leia o [CONTRIBUTING.md](CONTRIBUTING.md) antes de enviar.

## Autor

**Eduardo Fernandes Mateus** — [github.com/efernandesmateus](https://github.com/efernandesmateus)

## Licença

Distribuído sob a licença MIT. Veja [LICENSE](LICENSE).

Partes deste projeto são adaptadas do [Open Threat Hunting Framework (OTHF)](https://github.com/TactiKoolSec/OTHF), © 2022 TactiKoolSec, também sob licença MIT. A licença original está em [`referencias/OTHF/LICENSE`](referencias/OTHF/LICENSE).

## Aviso

Este material é fornecido "como está", para fins educacionais e de apoio a operações de segurança. Valide consultas e procedimentos no seu ambiente antes de usá-los em produção.
