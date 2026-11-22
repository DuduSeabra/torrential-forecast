# 0005 — Começar apenas com os pluviômetros CEMADEN

Data: 2026-11-22 · Status: aceita · Pesquisa: [003](../research/003-inventario-de-dados.md) (Opção A)

## Decisão
A Fase 2 (EDA, catálogo de eventos) e os primeiros baselines usam apenas a rede de pluviômetros
CEMADEN de Petrópolis, obtida pela plataforma PED (ped.cemaden.gov.br). GOES-16/19 será baixado
depois, apenas para as janelas definidas pelo catálogo de eventos (+ amostra de dias normais).

## Consequências
- O usuário precisa criar conta no PED e solicitar o histórico (assíncrono; link válido por 30 dias).
- Todos os usos devem citar: "Dados da Rede Observacional do CEMADEN/MCTI".
- Radar fica fora por ora (Morin: histórico curto; INEA/DECEA: acesso incerto).
