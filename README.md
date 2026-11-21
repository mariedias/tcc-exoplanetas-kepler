# Detecção de exoplanetas com CNN 1D, BiLSTM e Transformer

Notebooks do Trabalho de Conclusão de Curso *Comparação entre Arquiteturas de Aprendizado Profundo na Detecção Automatizada de Exoplanetas em Curvas de Luz do Telescópio Kepler* (Maria Eduarda Silveira Dias, Bacharelado em Ciência da Computação, Unisinos, 2026).

O trabalho compara três arquiteturas na classificação de KOIs do catálogo Kepler DR25 em planetas confirmados e falsos positivos, sob um protocolo em que só a arquitetura varia. As curvas de luz são convertidas nas Visões Global e Local de Shallue e Vanderburg (2018), e as três redes, com dois ramos, um para cada visão, são treinadas com os mesmos dados, a mesma partição por estrela, a mesma perda e o mesmo otimizador.

## Notebooks

| Notebook | Conteúdo | Seção do TCC |
|---|---|---|
| `01_coleta_rotulos` | Tabela de KOIs do DR25 e recortes da amostra | 3.3 |
| `02_piloto_lightkurve` | Teste do pré-processamento em cinco alvos | 3.8 |
| `03_dataset_completo` | Visões dos 6.695 KOIs, partição por estrela e SMOTE | 3.3, 3.4 |
| `04_modelos` | Definição das três arquiteturas | 3.5 |
| `05_treino` | Treinamento inicial, sem *gradient clipping* | 4.1, B.1 |
| `05b_busca_hiperparametros` | Busca em grade com *gradient clipping* | 3.5, 4.1 |
| `05c_robustez_sementes` | Treinamento das configurações finais com três sementes | 4.1, B.2 |
| `06_avaliacao` | Métricas no teste e análise de erros | 4.2, 4.3 |
| `07_mapas_atencao` | Mapas de atenção do Transformer | C.1 |
| `08_interpretabilidade_comparativa` | *Saliency maps* e Grad-CAM | C.2 a C.4 |
| `09_inspecao_erros_profundos` | Hipótese do formato em V | B.3 |
| `10_diagnostico_outliers` | Corte de *outliers* e vazamento entre estrelas | 4.4, B.4, B.5 |

Os notebooks estão na ordem de execução. Cada um indica, no início, o que precisa dos anteriores e se usa GPU.

## Duas rodadas do experimento

O experimento foi executado duas vezes. Na primeira rodada, a análise de erros revelou dois problemas:

1. **Corte de *outliers* simétrico.** A configuração padrão do Lightkurve removia pontos acima e abaixo de 6 desvios-padrão da mediana. Em trânsitos profundos, isso apagava o fundo do trânsito, que a interpolação transformava em uma reta, em 10,4% das visões.
2. **Vazamento entre treino e teste.** A partição era feita por KOI, e KOIs da mesma estrela, que compartilham a mesma curva de luz, caíam em conjuntos diferentes.

Na rodada final, o corte passou a remover só os pontos acima da mediana e a partição passou a ser feita por estrela. Os resultados do TCC são os da rodada final.

O código da primeira rodada foi mantido como registro e como base do Apêndice B do TCC:
- `views/` guarda as visões da primeira rodada e `views_v2/`, as da rodada final;
- `v1_particao_por_koi/` guarda o dataset, os modelos e os resultados da primeira rodada;
- a célula de arquivamento do notebook 03 e a variável `VIEWS_DIR` do notebook 10 permitem alternar entre as duas versões.

Em uma execução do zero, basta seguir a ordem dos notebooks e pular a célula de arquivamento do notebook 03.

## Como executar

Os notebooks foram feitos para o Google Colab e gravam os arquivos no Google Drive, em `MyDrive/TCC-Exoplanetas/`. Os notebooks 05, 05b e 05c precisam de GPU (foi usada uma T4); os demais rodam em CPU.

O download das curvas de luz (notebook 03) leva várias horas. A Parte A desse notebook pode ser executada em várias sessões, porque sempre continua de onde parou.

Os dados pesados não estão no repositório, porque são públicos e gerados pelos próprios notebooks: o 01 baixa a tabela de KOIs do NASA Exoplanet Archive, e o 03 baixa as curvas do MAST.

A exceção é o `koi_labels.csv`, com a amostra exata usada no TCC (6.695 KOIs). O Exoplanet Archive atualiza as disposições à medida que novos planetas são confirmados, e uma execução posterior do notebook 01 já encontrou um confirmado a mais. Para reproduzir os resultados, use o arquivo do repositório.

## Ambiente

| Componente | Versão |
|---|---|
| Python | 3.13 (Google Colab) |
| PyTorch | 2.11.0 (CUDA 13.0) |
| Lightkurve | 2.6.0 |
| Astropy | 7.2.3 |
| NumPy | 2.4.6 |
| pandas | 2.2.3 |
| scikit-learn | 1.6.1 |
| imbalanced-learn | 0.14.2 |
| Matplotlib | 3.10.0 |

## Reprodutibilidade

As sementes são fixas: 42 na partição e no SMOTE; 42, 7 e 123 no treinamento das configurações finais. As configurações escolhidas e o protocolo ficam registrados em `config_final.json`, gerado pelo notebook 05b. Com três sementes, os resultados descrevem a variabilidade entre inicializações, mas não permitem testes de significância.

## Referências principais

- SALINAS, H. et al. Distinguishing a planetary transit from false positives: a transformer-based classification for planetary transit signals. **Monthly Notices of the Royal Astronomical Society**, v. 522, n. 3, p. 3201-3216, 2023. DOI: [10.1093/mnras/stad1173](https://doi.org/10.1093/mnras/stad1173).
- SHALLUE, C. J.; VANDERBURG, A. Identifying exoplanets with deep learning: a five-planet resonant chain around Kepler-80 and an eighth planet around Kepler-90. **The Astronomical Journal**, v. 155, n. 2, p. 94, 2018. DOI: [10.3847/1538-3881/aa9e09](https://doi.org/10.3847/1538-3881/aa9e09).
- THOMPSON, S. E. et al. Planetary candidates observed by Kepler. VIII. A fully automated catalog with measured completeness and reliability based on data release 25. **The Astrophysical Journal Supplement Series**, v. 235, n. 2, p. 38, 2018. DOI: [10.3847/1538-4365/aab4f9](https://doi.org/10.3847/1538-4365/aab4f9).
