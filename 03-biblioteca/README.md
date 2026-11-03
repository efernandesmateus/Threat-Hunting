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
| [TH-EXF-001](exfiltration/th-exf-001-exfiltracao-via-dns.md) | Exfiltração de dados via DNS | Exfiltration | Proposto | 1.0 |

## Hunts planejados

A biblioteca inicial prevê **12 hunts**, escolhidos pela relevância das técnicas e pela disponibilidade de telemetria nos ambientes, e não para cobrir todas as táticas. Eles serão publicados em blocos de três por versão.

| ID | Hipótese | Tática |
|----|----------|--------|
| TH-INI-001 | Autenticação em acesso remoto externo de origem incomum | Initial Access |
| TH-EXE-001 | Interpretadores de script com comandos codificados ou ofuscados | Execution |
| TH-PER-001 | Tarefas agendadas e serviços criados fora do padrão | Persistence |
| TH-STL-001 | Utilitários legítimos do sistema usados para executar código | Stealth |
| TH-DIM-001 | Desativação ou alteração de ferramentas de segurança e de logs | Defense Impairment |
| TH-CRD-001 | Acesso à memória do processo de autenticação do sistema | Credential Access |
| TH-DIS-001 | Enumeração do diretório e de grupos privilegiados em rajada | Discovery |
| TH-LAT-001 | Movimentação lateral com contas administrativas entre hosts | Lateral Movement |
| TH-COL-001 | Compactação de grandes volumes de arquivos antes da saída | Collection |
| TH-C2-001 | Beaconing: conexões periódicas para destinos raros | Command and Control |
| TH-IMP-001 | Preparação para ransomware: exclusão de backups e cópias de recuperação | Impact |

**Backlog futuro:**

| Hipótese | Tática | Origem |
|----------|--------|--------|
| Exfiltração com uso de serviços de IA (técnica incluída no ATT&CK v20) | Exfiltration | Inteligência |

### Táticas sem hunt previsto

| Tática | Motivo |
|--------|--------|
| Reconnaissance | Acontece quase toda fora do ambiente do cliente; há pouca telemetria interna para caçar |
| Resource Development | Preparação de infraestrutura do adversário, fora do ambiente do cliente |
| Privilege Escalation | Muitas técnicas são compartilhadas com Persistence e Execution no próprio ATT&CK e ficam cobertas de forma indireta por TH-PER-001 e TH-EXE-001 |

Para sugerir um hunt, abra uma issue no repositório.

## Origem no OTHF

Módulos 10 (Hunt Data Sources) e 11 (Hunting Techniques), complementados por MITRE ATT&CK e pela documentação das plataformas.
