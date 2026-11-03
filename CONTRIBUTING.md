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
- Não cite fabricantes nem produtos de segurança e de defesa. Descreva a capacidade pelo tipo: "o EDR", "o SIEM", "o firewall", "o serviço de diretório". Frameworks e projetos abertos de referência (MITRE ATT&CK, OTHF) podem ser citados.
- **Exceção técnica ([ADR 0007](decisoes/0007-excecao-tecnica-sistemas-operacionais.md)):** sistemas operacionais e seus componentes nativos (interpretadores, agendadores, serviços, registros de eventos, caminhos) podem ser citados quando necessários para descrever a técnica com precisão.
- Texto objetivo, voltado a um analista que vai executar o procedimento.

## 3. Formato e nomes

- Conteúdo em Markdown. Planilhas apenas quando uma tabela não couber bem em Markdown.
- Nomes de arquivo em minúsculas, sem acento, palavras separadas por hífen. Ex.: `processo-de-hunting.md`.
- Cada pasta tem um `README.md` explicando o que ela contém.

## 4. Hunts da biblioteca

- Todo hunt segue o [template de hunt](02-templates/template-hunt.md) e o ciclo de vida descrito em [`02-templates/README.md`](02-templates/README.md).
- **ID:** `TH-<TÁTICA>-<NNN>`, onde `<TÁTICA>` é uma das siglas abaixo e `<NNN>` é sequencial dentro da tática. Ex.: `TH-EXF-001`.
- **Arquivo:** `03-biblioteca/<tatica>/<id-em-minusculas>-<descricao-curta>.md`. Ex.: `03-biblioteca/exfiltration/th-exf-001-exfiltracao-via-dns.md`.

As siglas seguem as táticas da matriz **MITRE ATT&CK Enterprise v20** (15 táticas):

| Sigla | Tática MITRE ATT&CK  | ID     | Pasta                   |
|-------|----------------------|--------|-------------------------|
| REC   | Reconnaissance       | TA0043 | `reconnaissance`        |
| RES   | Resource Development | TA0042 | `resource-development`  |
| INI   | Initial Access       | TA0001 | `initial-access`        |
| EXE   | Execution            | TA0002 | `execution`             |
| PER   | Persistence          | TA0003 | `persistence`           |
| PRV   | Privilege Escalation | TA0004 | `privilege-escalation`  |
| STL   | Stealth              | TA0005 | `stealth`               |
| DIM   | Defense Impairment   | TA0112 | `defense-impairment`    |
| CRD   | Credential Access    | TA0006 | `credential-access`     |
| DIS   | Discovery            | TA0007 | `discovery`             |
| LAT   | Lateral Movement     | TA0008 | `lateral-movement`      |
| COL   | Collection           | TA0009 | `collection`            |
| C2    | Command and Control  | TA0011 | `command-and-control`   |
| EXF   | Exfiltration         | TA0010 | `exfiltration`          |
| IMP   | Impact               | TA0040 | `impact`                |

> **Atenção à mudança da v19:** a tática **Defense Evasion** foi aposentada no ATT&CK v19 (abril de 2026) e substituída por **Stealth**, que reaproveita o ID TA0005, e **Defense Impairment** (TA0112). Ao usar fontes anteriores a essa versão, remapeie a técnica pela intenção do adversário: **esconder-se** (Stealth) ou **quebrar ou desativar defesas** (Defense Impairment). Como o ID TA0005 mudou de significado, sempre use o **nome** da tática junto com o ID.

### Atualização do ATT&CK

- A versão do ATT&CK de referência é a indicada acima. Ao sair uma nova versão, verifique as notas de lançamento e atualize esta tabela e os mapeamentos dos hunts afetados.
- Hunts mapeados para técnicas revogadas ou movidas de tática devem ser revisados, conforme a regra de [versionamento](02-templates/README.md#versionamento-do-hunt).

Quando um hunt cobrir várias táticas, classifique-o pela tática principal e liste as demais no próprio documento.

## 5. Consultas

- Formato conforme o [ADR 0005](decisoes/0005-formato-das-consultas.md): **pseudoconsulta** obrigatória e **regra Sigma** recomendada, em arquivo `.yml` com o mesmo nome base do hunt.
- Traduções para plataformas específicas não entram no repositório.
- Só inclua sintaxe de consulta validada na plataforma.
- Consulta não testada em plataforma real deve ser marcada como **Não validada**. Quem adota o hunt deve validá-la no próprio ambiente antes de usá-la em clientes ([ADR 0006](decisoes/0006-validacao-pelo-adotante.md)).
- Cada consulta indica o **tipo** de plataforma (SIEM, EDR, XDR) e onde ela deve ser executada, sem citar fabricantes ou produtos (ver [ADR 0004](decisoes/0004-neutralidade-de-fabricante.md)).

## 6. Contribuindo com validações

Se você validou um hunt da biblioteca no seu ambiente, contribua com o resultado: ele pode levar o hunt ao status **Validado pela comunidade**.

1. Abra uma issue ou um pull request com o ID e a versão do hunt.
2. Informe: **tipo** de plataforma (SIEM, EDR, XDR), data da validação, como os dados de teste foram gerados, período histórico analisado, ruído observado e ajustes necessários na lógica ou nos limiares.
3. Não cite fabricantes nem produtos, e não inclua dados reais (consultas traduzidas para a sua plataforma ficam com você).

As informações entram na seção 9 do hunt.

## 7. Fluxo de trabalho no Git

- A branch `main` contém apenas conteúdo revisado.
- Trabalhe em branches com prefixo pelo tipo: `docs/`, `hunt/`, `template/`, `fix/`. Ex.: `hunt/th-exf-001`.
- Uma entrega por pull request, com descrição do que foi criado e do que ficou pendente.
- Para mudanças grandes, abra uma issue antes para discutir a proposta.
- Mensagens de commit no imperativo e em português. Ex.: `Adiciona template de hunt`.
- Atualize o [CHANGELOG](CHANGELOG.md) a cada entrega.

## 8. Conteúdo derivado do OTHF

- Traduza e adapte para o contexto MSSP; evite copiar trechos longos.
- Mantenha `referencias/OTHF/LICENSE` intacto.
- Ao adaptar uma seção, cite o módulo de origem no fim do documento. Ex.: `Baseado em: OTHF, módulo 6 — Operational`.
