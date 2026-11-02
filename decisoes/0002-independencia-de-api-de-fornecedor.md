# 0002 — Independência de API de fornecedor

- **Status:** Aceita
- **Data:** 2026-11-02

## Contexto

Uma forma comum de estruturar hunting como serviço é automatizar a execução pelas APIs das plataformas de segurança. Isso cria dependência: a disponibilidade, os limites e o escopo das APIs variam por fornecedor, por licença e ao longo do tempo, e nem toda plataforma expõe pela API as buscas necessárias para hunting.

## Decisão

A metodologia é desenhada para funcionar **sem depender de API de fornecedor**. A execução é feita por analistas, com base nos templates e na biblioteca de hunts, usando as interfaces de busca de cada plataforma.

Automação via API pode ser adotada como **melhoria**, nunca como pré-requisito para o serviço funcionar.

## Consequências

- A metodologia pode ser aplicada em clientes de diferentes plataformas.
- Cada hunt traz consultas por plataforma, em seções separadas, e uma estratégia de busca independente de sintaxe.
- A execução é mais manual no início; o esforço por cliente deve ser considerado no dimensionamento e na precificação.
- Hunts maduros podem ser convertidos em regras de detecção nas plataformas, seguindo a abordagem "hunt once" descrita no OTHF.
