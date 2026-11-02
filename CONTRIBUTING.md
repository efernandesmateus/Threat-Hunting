# Como contribuir

Obrigado pelo interesse em contribuir! Este guia reúne as convenções que todo conteúdo deste repositório deve seguir. Mudanças nestas regras devem ser registradas como uma decisão em [`decisoes/`](decisoes/).

Ao enviar uma contribuição, você concorda que ela seja distribuída sob a [licença MIT](LICENSE) do projeto.

## 1. Regras de segurança (obrigatórias)

Este é um repositório **público**. Nunca inclua:

- nomes de organizações, clientes ou empregadores, ou qualquer informação que os identifique;
- logs, eventos, exports ou capturas reais;
- IPs, hostnames, domínios, URLs internas, nomes de usuário ou e-mails reais;
- credenciais, tokens, chaves de API ou segredos de qualquer tipo;
- evidências ou relatórios de incidentes e hunts executados;
- material interno de empresas (playbooks, procedimentos, relatórios) que você não tenha direito de publicar. Se a ideia for útil, reescreva-a de forma genérica a partir de fontes públicas.

Em exemplos, use sempre dados fictícios e reservados para documentação:

| Tipo        | Use                                                   |
|-------------|-------------------------------------------------------|
| IPv4        | `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24` (RFC 5737) |
| Domínio     | `example.com`, `cliente-exemplo.local`                |
| Host        | `HOST-EXEMPLO-01`, `SRV-EXEMPLO-01`                   |
| Usuário     | `usuario.exemplo`                                     |

Antes de cada commit, revise o diff procurando qualquer item da lista acima.

## 2. Idioma e escrita

- Português do Brasil.
- Termos técnicos consagrados podem ficar em inglês: hunt, hunting, IOC, TTP, stacking, baseline, false positive.
- Texto objetivo, voltado a um analista que vai executar o procedimento.

## 3. Formato e nomes

- Conteúdo em Markdown. Planilhas apenas quando uma tabela não couber bem em Markdown.
- Nomes de arquivo em minúsculas, sem acento, palavras separadas por hífen. Ex.: `processo-de-hunting.md`.
- Cada pasta tem um `README.md` explicando o que ela contém.

## 4. Hunts da biblioteca

- Todo hunt segue o [template de hunt](02-templates/template-hunt.md) e o ciclo de vida descrito em [`02-templates/README.md`](02-templates/README.md).
- **ID:** `TH-<TÁTICA>-<NNN>`, onde `<TÁTICA>` é uma das siglas abaixo e `<NNN>` é sequencial dentro da tática. Ex.: `TH-EXF-001`.
- **Arquivo:** `03-biblioteca/<tatica>/<id-em-minusculas>-<descricao-curta>.md`. Ex.: `03-biblioteca/exfiltration/th-exf-001-exfiltracao-via-dns.md`.

| Sigla | Tática MITRE ATT&CK  | Pasta                   |
|-------|----------------------|-------------------------|
| REC   | Reconnaissance       | `reconnaissance`        |
| RES   | Resource Development | `resource-development`  |
| INI   | Initial Access       | `initial-access`        |
| EXE   | Execution            | `execution`             |
| PER   | Persistence          | `persistence`           |
| PRV   | Privilege Escalation | `privilege-escalation`  |
| DEF   | Defense Evasion      | `defense-evasion`       |
| CRD   | Credential Access    | `credential-access`     |
| DIS   | Discovery            | `discovery`             |
| LAT   | Lateral Movement     | `lateral-movement`      |
| COL   | Collection           | `collection`            |
| C2    | Command and Control  | `command-and-control`   |
| EXF   | Exfiltration         | `exfiltration`          |
| IMP   | Impact               | `impact`                |

Quando um hunt cobrir várias táticas, classifique-o pela tática principal e liste as demais no próprio documento.

## 5. Consultas

- Só inclua sintaxe de consulta validada na plataforma.
- Consulta não validada deve ser marcada com `TODO: validar` e não pode ser usada em cliente até a validação.
- Cada consulta indica a plataforma e, quando relevante, a versão ou o módulo (ex.: USM Anywhere — Events; CrowdStrike — Advanced Event Search).

## 6. Fluxo de trabalho no Git

- A branch `main` contém apenas conteúdo revisado.
- Trabalhe em branches com prefixo pelo tipo: `docs/`, `hunt/`, `template/`, `fix/`. Ex.: `hunt/th-exf-001`.
- Uma entrega por pull request, com descrição do que foi criado e do que ficou pendente.
- Para mudanças grandes, abra uma issue antes para discutir a proposta.
- Mensagens de commit no imperativo e em português. Ex.: `Adiciona template de hunt`.
- Atualize o [CHANGELOG](CHANGELOG.md) a cada entrega.

## 7. Conteúdo derivado do OTHF

- Traduza e adapte para o contexto MSSP; evite copiar trechos longos.
- Mantenha `referencias/OTHF/LICENSE` intacto.
- Ao adaptar uma seção, cite o módulo de origem no fim do documento. Ex.: `Baseado em: OTHF, módulo 6 — Operational`.
