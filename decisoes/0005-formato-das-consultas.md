# 0005 — Formato das consultas na biblioteca

- **Status:** Aceita
- **Data:** 2026-11-03

## Contexto

Consultas de hunt são, por natureza, específicas de cada plataforma: cada SIEM, EDR ou XDR tem a própria linguagem e os próprios nomes de campos. Ao mesmo tempo, o projeto não cita fabricantes nem produtos ([ADR 0004](0004-neutralidade-de-fabricante.md)) e precisa que os hunts sejam executáveis por qualquer time.

## Decisão

Cada hunt da biblioteca publica a consulta em **dois formatos neutros**:

1. **Pseudoconsulta:** a lógica da busca em passos estruturados (fonte, filtros, agregações, limiares e saída), que qualquer analista consegue traduzir para a sua plataforma. É **obrigatória** em todo hunt.
2. **Regra Sigma:** quando a lógica puder ser expressa no formato [Sigma](https://github.com/SigmaHQ/sigma-specification), um padrão aberto mantido pela comunidade e conversível para a linguagem de várias plataformas. Fica em um arquivo `.yml` ao lado do hunt, com o mesmo nome base. É **recomendada**, não obrigatória, porque parte da lógica de hunting (baseline, stacking, entropia) não cabe em uma regra.

As **traduções para plataformas específicas não fazem parte do repositório**. Cada time mantém as suas, validadas no próprio ambiente.

### Ferramentas de adversários

Ferramentas usadas por adversários podem ser citadas pelo nome **quando constarem no catálogo de Software do MITRE ATT&CK**. Elas descrevem o comportamento do adversário e não são produtos de defesa. A neutralidade do ADR 0004 continua valendo para plataformas e produtos de segurança e de tecnologia.

## Consequências

- A seção 8 do template passa a ter pseudoconsulta, regra Sigma e uma nota sobre as implementações por plataforma.
- Regras Sigma usam a taxonomia genérica de campos do Sigma. O mapeamento para os campos de cada plataforma é feito na conversão, por quem executa o hunt.
- O status **Validada** de uma consulta significa que a lógica foi testada em pelo menos uma plataforma real, ainda que a tradução usada não esteja no repositório. O registro informa o tipo de plataforma e a data, sem citar o produto.
