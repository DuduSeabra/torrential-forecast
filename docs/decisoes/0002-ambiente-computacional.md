# 0002 — Ambiente computacional

Data: 2026-11-22 · Status: aceita

## Contexto
A GPU local é uma AMD Radeon RX 470/480/570/580 (arquitetura Polaris, gfx803). Essa geração não é
suportada pelas versões atuais do ROCm, então o PyTorch não a utiliza para aceleração.
Recursos disponíveis: Google Colab e Kaggle (camadas gratuitas, GPUs T4/P100 de ~16 GB).

## Decisão
- **Local (CPU):** download, processamento de dados, EDA, baselines leves, testes de código.
- **Colab/Kaggle:** treino e avaliação de redes neurais.

## Consequências
- Modelos e lotes precisam caber em ~16 GB de VRAM e em sessões de poucas horas
  (checkpoints frequentes, dados pré-processados e compactos).
- Notebooks de treino devem rodar em ambos os serviços sem alterações grandes.
