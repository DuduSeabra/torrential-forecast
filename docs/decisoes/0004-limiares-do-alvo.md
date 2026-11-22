# 0004 — Limiares do alvo: 10, 20 e 40 mm em 1 h

Data: 2026-11-22 · Status: aceita (provisória até a EDA) · Pesquisa: [002](../research/002-definicao-chuva-destrutiva.md) (Opção A)

## Decisão
O modelo prevê, para cada pluviômetro CEMADEN de Petrópolis e cada horizonte (+30/+60/+120/+180 min),
a probabilidade de o acumulado de 1 h exceder:
- **10 mm** — impacto urbano (transbordamento de rios, segundo a Defesa Civil);
- **20 mm** — chuva forte;
- **40 mm** — índice crítico da Região Serrana (DRM-RJ).

Acumulados observados de 24/72/96 h entram como features (contexto de solo encharcado), não como alvo.

## Consequências
- A EDA deve contar eventos por limiar/estação; se ≥40 mm/h for raro demais, avaliar agrupar estações
  ou usar acumulado de 3 h para o nível mais alto (e atualizar este ADR).
- Janela de 1 h (móvel ou hora cheia) será definida na EDA.
- Pedir à Defesa Civil de Petrópolis os limiares do plano de contingência para alinhar.
