# 001 — Estado da arte em nowcasting de chuva extrema (0–3 h)
Data: 2026-11-22 · Status: decidido (ADR 0003 — Opção B)

## Pergunta
Qual combinação de **dados de entrada**, **alvo (verdade de referência)**, **família de modelo** e
**métricas** representa o estado da arte para prever chuva torrencial de 0–3 h numa região
montanhosa e sem radar denso como Petrópolis — e é viável com GPU gratuita (Colab/Kaggle)?

## Por que importa para Petrópolis
- **Chuva convectiva muito localizada.** Em 15/02/2022 o posto São Sebastião registrou ~260 mm em
  3h20 ([relatório Defesa Civil](https://www.petropolis.rj.gov.br/pmp/phocadownload/defesa-civil/relatorios/2025/Relatrio%20-%20%20Eventos%20chuvosos%20de%2015%20de%20fevereiro%20de%202022.pdf)).
  Células convectivas têm poucos km e vivem de minutos a ~1 h — exatamente o que é difícil de prever.
- **Orografia (Serra do Mar).** O relevo força o ar úmido a subir e intensifica a chuva
  (*chuva orográfica*). Produtos de satélite que estimam chuva pela temperatura do topo da nuvem
  tendem a errar nessa situação (ver Hydronn e o README do RRQPE abaixo).
- **Radar.** Os radares do INEA ficam em Guaratiba (cidade do Rio). *Eu infiro* que a cobertura da
  serra é limitada pelo bloqueio do relevo; a disponibilidade real de radar para Petrópolis é uma
  pergunta em aberto (Fase 2).
- **Pluviômetros.** O CEMADEN tem 33 pluviômetros em Petrópolis (24 operando), com leituras a cada
  10 min — é a **única medição direta** de chuva densa na cidade.

### Conceitos rápidos
- **Nowcasting:** previsão de 0–6 h feita principalmente extrapolando observações recentes,
  não integrando equações da atmosfera (isso é NWP — *numerical weather prediction*).
- **Advecção / optical flow:** estimar o vetor de movimento da chuva e "empurrar" o campo para o
  futuro. É o baseline clássico (biblioteca pySTEPS). Não cria nem mata células.
- **Borramento (blurring):** redes treinadas com MSE preveem a *média* dos futuros possíveis, o que
  vira um campo borrado com picos achatados — fatal para chuva extrema. Modelos generativos
  (GAN, difusão) existem principalmente para resolver isso.
- **CSI (Critical Success Index):** acertos / (acertos + falsos alarmes + perdas) para um limiar,
  ex.: chuva ≥ 50 mm/h. **POOL4/16** = calcula após max-pooling 4×4/16×16, tolerando erro de posição.

## Papers analisados
| Ref | Ano | Venue | Dados | Método | Resultado principal | Limitações | Relevância | Lido? |
|---|---|---|---|---|---|---|---|---|
| [TUPANN](https://arxiv.org/abs/2511.05471) (Catão et al., IMPA) | 2025 | arXiv | GOES-16 RRQPE 2 km/10 min; IMERG; Rio, Manaus, Miami, La Paz | Encoder variacional prevê campos de movimento+intensidade (supervisionado por optical flow), MaxViT evolui o estado latente, *warp* diferenciável (de NowcastNet) | Rio, CSI ≥64 mm/h (POOL1/POOL4): **TUPANN 0,072/0,090**, NowcastNet 0,059/0,074, CasCast 0,016/0,027, Earthformer 0,009/0,006, pySTEPS 0,013/0,015. Treino em 1×A100; inferência <2 s; pipeline roda em RTX 3080. Código: [github](https://github.com/acataos/tupann) | Verdade = RRQPE (estimativa de satélite, não medição); sem incerteza; perde skill após 2 h | **alta** | sim |
| [NPM](https://arxiv.org/abs/2412.11480) (Park et al.) | 2024 | AAAI 2025 | GK2A (3 canais IR/WV) + relevo (DEM); alvo radar coreano | 2 estágios: prevê imagens de satélite futuras → traduz satélite→chuva (GAN) | CSI ≥8 mm/h médio 0,05; caso de enchente: obs ~60 mm, IMERG 46,5, NPM 27,5 mm | Limiares baixos; treino em 8×A6000; subestima extremos | média | sim |
| [CasCast](https://arxiv.org/abs/2402.04290) (Gong et al.) | 2024 | ICML | Radar (SEVIR, HKO-7, MeteoNet), 60 min | Cascata: modelo determinístico (Earthformer/SimVP) + **difusão latente** gera detalhes de pequena escala | SEVIR CSI-219 POOL16: CasCast 0,284 vs Earthformer 0,148 (+91,8%) e NowcastNet 0,270 | Treino de 18 h em 4×A100; só radar; retreino por região | média (pesada p/ GPU grátis) | parcial |
| [NowcastNet](https://www.nature.com/articles/s41586-023-06184-4) (Zhang et al.) | 2023 | Nature | Radar EUA+China, até 3 h | Rede de evolução (advecção neural com equação de continuidade) + rede generativa | Segundo fontes secundárias: 62 meteorologistas o classificaram 1º em ~70% dos casos ([Berkeley](https://engineering.berkeley.edu/news/2023/07/nowcasting-model-enables-forecasting-of-extreme-precipitation/)); estudo independente: CSI mediano 0,30 vs 0,04 do HRRR para >16 mm/h ([arXiv 2407.11317](https://arxiv.org/abs/2407.11317)) | Superestima chuva agregada em lead times longos (estudo independente) | alta (base de TUPANN) | **não lido** (bloqueado) |
| [Curcio et al.](https://arxiv.org/abs/2505.19258) (Cefet/RJ, LNCC, AlertaRio) | 2025 | arXiv | 83 pluviômetros Sirenes + 33 AlertaRio + 19 INMET; ERA5 (treino) / GFS (inferência); grade 0,25°, horária | STConvS2S, classificação em 4 classes (fraca/moderada/forte/extrema) | F1 classe forte (≥25 mm/h) a 1 h: **0,20**; extrema: apenas **2,04%** acertados (mas 93,9% vistos como "chuva significativa") | Grade grossa (0,25° ≈ 28 km); forte subestimação de extremos | **alta** (mesma região, mostra a dificuldade) | parcial |
| [Hydronn](https://amt.copernicus.org/articles/15/6907/2022/) (Pfreundschuh et al.) | 2022 | AMT | GOES-16 ABI → alvo GPM radar+radiômetro; América do Sul | Rede neural de *retrieval* (estima chuva atual a partir do satélite), com incerteza | vs HYDRO (operacional INPE): MAE horário −21%/−22%, correlação +138%/+312% (jun/dez 2020) | Média subestima chuva forte; amostras da distribuição recuperam melhor. Código no Zenodo | alta (melhor alvo de satélite para o Brasil) | parcial |
| [Ritvanen et al.](https://gmd.copernicus.org/articles/18/1851/2025/) | 2025 | GMD | Radar suíço 1 km/5 min | Framework de verificação por rastreamento de células convectivas; compara advecção, S-PROG, LINDA, L-CNN | Métricas de campo inteiro (CSI, FSS) são dominadas por chuva estratiforme e escondem o desempenho em células convectivas; L-CNN (DL) reproduziu melhor a existência de células, mas superestima área e subestima intensidade | Parâmetros de rastreamento ajustados à mão; só 1ª hora | alta (métricas) | parcial |
| [Cereto et al.](https://www.editorarealize.com.br/editora/anais/sinageo/2025/TRABALHO_COMPLETO_EV217_ID1537_TB609_26072025091233.pdf) (UFF) | 2025 | SINAGEO | 1 pluviômetro CEMADEN (Quitandinha, 330390617A) + 11.043 atendimentos da Defesa Civil (2015–2024) | MLP/SVM/RF sobre acumulados 24/48/72 h → dia com deslizamento | MLP recall 1,0 / F1 0,80 na estação de treino; em outra estação a 32,6 km: **F1 0,17, precision 0,09** | Split aleatório (não temporal); escala diária, não nowcasting | média (dados e rótulos locais) | sim |

Apoio (só snippets/abstract, não lidos): [survey An et al. 2024](https://arxiv.org/abs/2406.04867);
[DGMR, Ravuri et al. 2021](https://arxiv.org/abs/2104.00954) (CSI alto de U-Net vinha de previsões borradas, visível no espectro de potência);
[LDCast](https://arxiv.org/abs/2304.12891); [README RRQPE NOAA](https://noaasis.noaa.gov/pdf/ps-pvr/goes-16/ABI/Rainfall%20Rate/Full/GOES-16_ABI_L2_RRQPE_Full_README.pdf) (subestimação significativa em certas condições; mais confiável em convecção profunda);
[GNN estações+radar nórdico](https://arxiv.org/abs/2606.18436) (estações melhoram previsão no ponto, não o mapa);
[Prithvi WxC](https://arxiv.org/abs/2409.13598) (foundation model 2,3 B parâmetros, base MERRA-2).

## O que a literatura converge / onde diverge
**Converge**
1. **Modelos treinados só com MSE borram e matam os extremos.** Estado da arte = modelos com
   componente físico de advecção (NowcastNet, TUPANN) e/ou generativo (GAN, difusão: CasCast, LDCast).
2. **Advecção aprendida + rede neural** é o padrão de maior sucesso em limiares altos:
   em Rio, os dois melhores (TUPANN, NowcastNet) usam o mesmo *warp* físico; Earthformer puro
   e pySTEPS ficam bem atrás em ≥64 mm/h.
3. **Limiares altos são muito difíceis:** CSI em chuva extrema fica tipicamente entre 0,01 e 0,3
   mesmo no estado da arte. Curcio et al. (Rio) acertaram 2% da classe extrema a 1 h.
4. **Métricas:** CSI/POD/FAR por limiar + versão com tolerância espacial (pooling/FSS) +
   métrica probabilística (CRPS/Brier) + diagnóstico de borramento (espectro de potência) +
   avaliação por evento/célula (Ritvanen).

**Diverge / lacunas**
- **Qual é a "verdade"?** Radar (NowcastNet, CasCast), produto de satélite (TUPANN) ou pluviômetro
  (Curcio, Cereto). Para Petrópolis isso é o ponto crítico: satélite subestima chuva orográfica e
  ninguém publicou nowcasting avaliado contra os pluviômetros CEMADEN da cidade.
- **Foundation models** (Prithvi WxC, Aurora) ainda não mostraram ganho claro em nowcasting
  convectivo de 0–3 h; são pesados demais para GPU gratuita.
- **Estações em modelos de grade:** ajudam no ponto, não necessariamente no mapa (GNN nórdico).

## Opções para o projeto

### Opção A — Nowcasting em grade com satélite (estilo TUPANN)
Entrada: sequência GOES-16/19 sobre a Serra Fluminense → saída: mapa de chuva futura.
Alvo: RRQPE ou Hydronn; avaliação final nos pluviômetros CEMADEN.
- **Prós:** caminho mais próximo do estado da arte na região; código TUPANN aberto e treinado no Rio
  (pode ser usado como baseline forte); GOES é gratuito, cobre tudo, a cada 10 min.
- **Contras:** a rede aprende a imitar uma *estimativa* de satélite que provavelmente subestima o
  extremo orográfico; o alvo limita o teto de acerto.
- **Custo:** médio–alto (TUPANN treinou em A100; modelos menores cabem em T4 com patches 128×128).

### Opção B — Previsão nos pontos dos pluviômetros, com satélite como contexto (híbrida)
Entrada: histórico dos ~33 pluviômetros CEMADEN + patch GOES (e, se existir, radar) →
saída: **probabilidade de exceder limiares** (ex.: ≥20, ≥50 mm/h ou acumulados de 1 h) em cada
estação de Petrópolis, para +30/+60/+120/+180 min.
- **Prós:** treina e avalia contra a medição mais confiável e exatamente o que dispara alertas;
  saída probabilística combina com tomada de decisão (Defesa Civil); evolução natural em degraus
  (baselines tabulares → GBM → rede com encoder de satélite); barato em GPU.
- **Contras:** poucos eventos extremos (forte desbalanceamento); generalização espacial é ruim
  (Cereto: F1 cai de 0,80 para 0,17 a 32 km) — exige modelo multiestação; sem mapa contínuo.
- **Custo:** baixo–médio.

### Opção C — Nowcasting de radar (estilo NowcastNet/CasCast)
- **Prós:** maior respaldo na literatura; radar vê a estrutura da célula em ~1 km/5 min.
- **Contras:** disponibilidade e cobertura de radar sobre Petrópolis ainda desconhecidas;
  modelos de difusão exigem várias A100.
- **Custo:** alto. Depende do inventário de dados da Fase 2.

## Recomendação (e por quê)
**Opção B como linha principal, usando a Opção A como fonte de features e baseline.**
- O objetivo é *alertar sobre chuva destrutiva em Petrópolis*; a métrica que importa é acertar
  nos pluviômetros da cidade, e só a Opção B otimiza diretamente isso.
- Permite começar simples e entender cada ganho (persistência → GBM → deep learning), o que
  combina com o objetivo de aprendizado do projeto.
- TUPANN (aberto, treinado no Rio) pode entrar como *feature* ou baseline: "o estado da arte em
  grade, avaliado nos pluviômetros de Petrópolis" já é um resultado novo — nenhum paper lido fez isso.
- A Opção C fica condicionada ao inventário de radar da Fase 2.

## Perguntas em aberto
1. Existe radar com cobertura útil da serra (INEA Guaratiba? CEMADEN? DECEA/Pico do Couto?) e com
   histórico acessível? → Fase 2.
2. Quantos eventos ≥50 mm/h os pluviômetros CEMADEN de Petrópolis registram por ano? Isso define
   se dá para treinar classificador de extremos ou se precisamos de limiares menores/regiões vizinhas.
3. Quão bem RRQPE/Hydronn/TUPANN reproduzem os pluviômetros de Petrópolis em eventos extremos?
4. Qual limiar/acumulado a Defesa Civil de Petrópolis usa para acionar sirenes? (alvo "operacional")
5. O Sistema Sirenes (83 pluviômetros usados por Curcio et al.) cobre Petrópolis e é acessível?

## Referências (links)
Ver tabela acima e [referencias.bib](referencias.bib).
