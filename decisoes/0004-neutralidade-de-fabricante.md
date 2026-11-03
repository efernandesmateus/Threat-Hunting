# 0004 — Neutralidade de fabricante

- **Status:** Aceita
- **Data:** 2026-11-03

## Contexto

As primeiras versões do projeto citavam plataformas comerciais específicas como plataformas iniciais. Isso dá a impressão de que a metodologia depende dessas ferramentas, pode ser lido como recomendação ou associação comercial e tende a envelhecer rápido, já que produtos mudam de nome, de interface e de fabricante.

## Decisão

O projeto **não cita fabricantes nem produtos comerciais** de segurança ou de tecnologia. Capacidades são descritas pelo **tipo** (SIEM, EDR, XDR, firewall, proxy, serviço de diretório, servidor de e-mail) e o conteúdo técnico é expresso em termos de comportamento, fontes de dados e técnicas MITRE ATT&CK.

Podem ser citados:

- frameworks, modelos e projetos abertos de referência, como MITRE ATT&CK, o próprio OTHF, o TaHiTI, o Hunting Maturity Model e a Pirâmide da Dor;
- organizações sem fins lucrativos e órgãos públicos, como fontes de inteligência;
- a plataforma onde o repositório está hospedado, apenas em links.

## Consequências

- A metodologia pode ser adotada por qualquer time, independentemente das ferramentas que usa.
- O resumo do OTHF em `referencias/OTHF/` generaliza as menções a produtos feitas no material original.
- As consultas dos hunts precisam de um formato que respeite essa regra. O formato das consultas na biblioteca é definido no [ADR 0005](0005-formato-das-consultas.md).
