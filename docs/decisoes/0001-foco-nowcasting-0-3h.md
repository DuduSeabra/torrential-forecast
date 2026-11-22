# 0001 — Foco em nowcasting de 0–3 h

Data: 2026-11-22 · Status: aceita

## Contexto
O objetivo é alertar sobre chuvas destrutivas em Petrópolis. Previsões de dias (6–72 h) são
dominadas por modelos numéricos e modelos globais de IA; no curtíssimo prazo (0–3 h), métodos de
deep learning sobre satélite, radar e estações superam a extrapolação clássica, principalmente em
limiares altos (ex.: TUPANN, arXiv 2511.05471; survey An et al., arXiv 2406.04867).

## Decisão
O projeto foca em nowcasting com horizonte de 0–3 h. Previsões de maior prazo ficam fora de escopo por ora.

## Consequências
- As fontes de dados precisam ter alta frequência temporal (≤ 10–15 min): satélite geoestacionário,
  radar, pluviômetros automáticos.
- Fontes como ERA5 (horária, com atraso) entram, no máximo, como contexto ou variáveis auxiliares.
