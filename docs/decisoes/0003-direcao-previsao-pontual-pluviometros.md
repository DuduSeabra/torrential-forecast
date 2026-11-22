# 0003 — Direção: previsão probabilística nos pluviômetros, com satélite como contexto

Data: 2026-11-22 · Status: aceita · Pesquisa: [001](../research/001-estado-da-arte-nowcasting.md) (Opção B)

## Contexto
O estado da arte em nowcasting de chuva extrema (NowcastNet, TUPANN, CasCast) gera mapas de chuva
a partir de radar ou satélite. Para Petrópolis: (1) a cobertura de radar é incerta; (2) produtos de
satélite usados como alvo (RRQPE) tendem a subestimar chuva orográfica; (3) os pluviômetros
CEMADEN (33 na cidade, 10 min) são a única medição direta e densa, e é neles que se decide um alerta.
Modelos treinados em uma única estação generalizam mal (Cereto et al. 2025).

## Decisão
O alvo do projeto é a **probabilidade de a chuva exceder limiares** em cada pluviômetro de
Petrópolis, para horizontes de +30, +60, +120 e +180 min.
- Entradas: histórico da rede de pluviômetros (multiestação) + contexto de satélite GOES
  (e radar, se a Fase 2 mostrar que é viável).
- TUPANN (código aberto, treinado no Rio) entra como baseline e como possível feature.
- Evolução em degraus: persistência/climatologia → modelos tabulares (GBM) → deep learning.
- Limiares e acumulados exatos serão definidos na Fase 2 (com base na Defesa Civil/CEMADEN e na
  quantidade de eventos disponíveis).

## Consequências
- Fase 2 prioriza: dados CEMADEN de Petrópolis (qualidade, lacunas, nº de eventos extremos),
  definição operacional de "chuva destrutiva" e inventário de radar/satélite.
- Métricas: Brier score, confiabilidade (calibração), PR-AUC, POD/FAR/CSI por limiar e avaliação
  por evento; mapas (FSS/CSI com pooling) apenas para o baseline TUPANN.
- Split sempre temporal; evento de 15/02/2022 reservado para teste.
- Não gera mapa contínuo de chuva; se necessário, fica para trabalho futuro (Opção A/C).
