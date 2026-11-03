# 0007 — Exceção técnica para sistemas operacionais

- **Status:** Aceita
- **Data:** 2026-11-03
- **Complementa:** [ADR 0004](0004-neutralidade-de-fabricante.md)

## Contexto

O [ADR 0004](0004-neutralidade-de-fabricante.md) proíbe citar fabricantes e produtos. Ao desenvolver hunts de endpoint, essa regra se mostrou incompatível com a precisão técnica:

- mecanismos como tarefas agendadas, serviços e interpretadores de comando funcionam de forma diferente em cada sistema operacional, com eventos, caminhos e componentes próprios;
- o MITRE ATT&CK organiza técnicas por plataforma (Windows, Linux, macOS) e usa nomes de sistemas e componentes em várias subtécnicas;
- regras Sigma de endpoint precisam declarar o sistema operacional no campo `logsource` para serem convertíveis.

Sem nomear o sistema, os hunts de endpoint ficariam genéricos demais para serem executados.

## Decisão

Podem ser citados, **quando necessários para descrever a técnica com precisão**:

- **sistemas operacionais** (por exemplo, Windows, Linux, macOS);
- **componentes nativos** desses sistemas: interpretadores, agendadores, serviços, registros de eventos e seus identificadores, caminhos e arquivos de configuração;
- **nomes de técnicas e plataformas do MITRE ATT&CK**, exatamente como estão no framework;
- **valores de `logsource`** exigidos pela especificação Sigma.

Continua proibido citar **fabricantes e produtos de segurança e de defesa** (SIEM, EDR, XDR, firewall, proxy, plataformas de inteligência e similares), que é o objetivo central do ADR 0004. Também continua proibido citar produtos comerciais de terceiros quando o tipo for suficiente (por exemplo, "servidor de e-mail" em vez do nome do produto).

## Consequências

- Os hunts de endpoint podem ter seções por sistema operacional.
- O `CONTRIBUTING.md` passa a descrever a exceção.
- O ADR 0004 continua válido para tudo o que não está coberto por esta exceção.
