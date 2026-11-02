# 0001 — OTHF como base metodológica

- **Status:** Aceita
- **Data:** 2026-11-02

## Contexto

Estruturar um serviço de threat hunting sem uma referência aumenta o risco de lacunas em processo, métricas, maturidade e comunicação, e torna o serviço mais difícil de explicar e defender.

O Open Threat Hunting Framework (OTHF), da TactiKoolSec, cobre os componentes organizacionais, operacionais e técnicos de um programa de hunting, é agnóstico de ferramenta e é distribuído sob licença MIT.

## Decisão

Usar o OTHF como base metodológica do projeto, **adaptando-o ao modelo de serviço (MSSP)**:

- o OTHF pressupõe um time interno de uma única organização; este projeto atende o cenário de vários clientes, com escopo contratado;
- conceitos internos (apoio executivo, posicionamento do time, staffing) viram definição de oferta, papéis e divisão de responsabilidades entre SOC e cliente;
- o processo de hunting, a cadência, as métricas e o modelo de maturidade são aproveitados com adaptação.

## Consequências

- O projeto herda uma estrutura reconhecível e defensável.
- É preciso manter a atribuição e a licença MIT do OTHF em `referencias/OTHF/`.
- O OTHF não recebe atualizações desde maio de 2023 e seu conteúdo técnico é centrado em artefatos de endpoint Windows. A biblioteca de hunts precisa de fontes complementares (ex.: MITRE ATT&CK, documentação das plataformas).
- O OTHF não traz detecções prontas; a biblioteca de hunts é desenvolvida neste projeto.
