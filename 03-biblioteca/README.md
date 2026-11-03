# 03 — Biblioteca de hunts

Hunts reutilizáveis, executáveis em qualquer cliente que tenha as fontes de dados necessárias.

> **Antes de usar:** os hunts desta biblioteca são publicados como **Proposto**, ou seja, completos e revisados, mas **não validados em plataforma real** pelo projeto. Valide cada hunt no seu ambiente antes de usá-lo em clientes ([ADR 0006](../decisoes/0006-validacao-pelo-adotante.md)). Validou algum? [Contribua com o resultado](../CONTRIBUTING.md#6-contribuindo-com-validações).

## Organização

Uma subpasta por tática do MITRE ATT&CK. IDs, nomes de arquivo e siglas seguem o [CONTRIBUTING.md](../CONTRIBUTING.md#4-hunts-da-biblioteca).

## Regras

- Todo hunt segue o [template](../02-templates/template-hunt.md) e, quando houver, traz a regra Sigma em um `.yml` com o mesmo nome base (ver [ADR 0005](../decisoes/0005-formato-das-consultas.md)).
- Consultas não testadas em plataforma real ficam marcadas como **Não validada**.
- Nenhum dado real de cliente.

## Índice

| ID | Hunt | Tática | Status | Versão |
|----|------|--------|--------|--------|
| [TH-INI-001](initial-access/th-ini-001-acesso-remoto-origem-incomum.md) | Acesso remoto externo de origem incomum | Initial Access | Proposto | 1.0 |
| [TH-EXE-001](execution/th-exe-001-interpretadores-comandos-ofuscados.md) | Interpretadores com comandos codificados ou ofuscados | Execution | Proposto | 1.0 |
| [TH-PER-001](persistence/th-per-001-tarefas-servicos-fora-do-padrao.md) | Tarefas agendadas e serviços fora do padrão | Persistence | Proposto | 1.0 |
| [TH-STL-001](stealth/th-stl-001-utilitarios-legitimos-executando-codigo.md) | Utilitários legítimos do sistema usados para executar código | Stealth | Proposto | 1.0 |
| [TH-DIM-001](defense-impairment/th-dim-001-desativacao-ferramentas-logs.md) | Desativação ou alteração de ferramentas de segurança e de logs | Defense Impairment | Proposto | 1.0 |
| [TH-CRD-001](credential-access/th-crd-001-acesso-memoria-processo-autenticacao.md) | Acesso à memória do processo de autenticação do sistema | Credential Access | Proposto | 1.0 |
| [TH-DIS-001](discovery/th-dis-001-enumeracao-diretorio-em-rajada.md) | Enumeração do diretório e de grupos privilegiados em rajada | Discovery | Proposto | 1.0 |
| [TH-LAT-001](lateral-movement/th-lat-001-movimentacao-lateral-contas-administrativas.md) | Movimentação lateral com contas administrativas entre hosts | Lateral Movement | Proposto | 1.0 |
| [TH-COL-001](collection/th-col-001-compactacao-grandes-volumes.md) | Compactação de grandes volumes de arquivos antes da saída | Collection | Proposto | 1.0 |
| [TH-EXF-001](exfiltration/th-exf-001-exfiltracao-via-dns.md) | Exfiltração de dados via DNS | Exfiltration | Proposto | 1.0 |

## Hunts planejados

A biblioteca inicial prevê **12 hunts**, escolhidos pela relevância das técnicas e pela disponibilidade de telemetria nos ambientes, e não para cobrir todas as táticas. Eles são publicados em blocos de três por versão. Já publicados: 10. Planejados:

| ID | Hipótese | Tática |
|----|----------|--------|
| TH-C2-001 | Beaconing: conexões periódicas para destinos raros | Command and Control |
| TH-IMP-001 | Preparação para ransomware: exclusão de backups e cópias de recuperação | Impact |

**Backlog futuro:**

| Hipótese | Tática | Origem |
|----------|--------|--------|
| Exfiltração com uso de serviços de IA (técnica incluída no ATT&CK v20) | Exfiltration | Inteligência |
| Binários confiáveis usados para executar código em Linux e macOS (complemento do TH-STL-001) | Stealth | Inteligência |
| Acesso a credenciais em memória e em arquivos no Linux (complemento do TH-CRD-001) | Credential Access | Inteligência |

### Táticas sem hunt previsto

| Tática | Motivo |
|--------|--------|
| Reconnaissance | Acontece quase toda fora do ambiente do cliente; há pouca telemetria interna para caçar |
| Resource Development | Preparação de infraestrutura do adversário, fora do ambiente do cliente |
| Privilege Escalation | Muitas técnicas são compartilhadas com Persistence e Execution no próprio ATT&CK e ficam cobertas de forma indireta por TH-PER-001 e TH-EXE-001 |

Para sugerir um hunt, abra uma issue no repositório.

## Origem no OTHF

Módulos 10 (Hunt Data Sources) e 11 (Hunting Techniques), complementados por MITRE ATT&CK e pela documentação das plataformas.
