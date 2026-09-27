MVP — Engenharia de Dados: Análise do Desempenho Musical no Spotify
Contexto de Negócios e Perguntas
Este projeto desenvolve um pipeline de Engenharia de Dados em nuvem para organizar e analisar informações sobre o desempenho de músicas nas plataformas digitais.
O conjunto de dados principal utilizado é o Most Streamed Spotify Songs 2023, disponibilizado publicamente no Kaggle por Nidula Elgiriyewithana. A base reúne informações sobre músicas que tiveram destaque nas plataformas digitais, incluindo nome da faixa, artistas, data de lançamento, número de streams, presença em playlists e charts e características musicais como BPM, danceability, energy e valence.
No Kaggle, a licença do conjunto de dados está identificada como Other (specified in description). Como a documentação informa a utilização de múltiplas fontes sem identificá-las individualmente, o projeto preserva a proveniência declarada pelo responsável pelo dataset, sem assumir extração direta da API oficial do Spotify.
As perguntas definidas para orientar o projeto foram:
1. Quais músicas e artistas estão associados aos maiores volumes de streams?
2. Músicas presentes em mais playlists e charts apresentam também maior número de streams?
3. Como o desempenho das músicas varia de acordo com o período de lançamento?
4. Características musicais como danceability, energy e valence apresentam diferenças entre músicas de maior e menor alcance?
5. O consumo está concentrado em uma pequena parcela das músicas e artistas da base?
6. Existe associação entre o desempenho das músicas nas plataformas digitais e sua recepção pela crítica e pelos usuários?
Carga dos Dados
O arquivo spotify-2023.csv foi utilizado como fonte principal do pipeline.
A ingestão foi realizada no Databricks Free Edition, preservando inicialmente os dados recebidos na camada Bronze. Foram adicionados metadados técnicos para registrar e rastrear a ingestão.
A arquitetura adotada segue o padrão de camadas:
Fonte → Bronze → Silver → Gold → Análises
A camada Bronze preserva os dados recebidos. A Silver realiza os tratamentos de qualidade, padronização e conversão. A Gold disponibiliza os dados modelados para consumo analítico.
O código completo da ingestão e das demais etapas está disponível no notebook final deste repositório:
00_MVP_Engenharia_Dados_Spotify_27092026_vF.ipynb
Modelagem e Catálogo de Dados
A camada Gold foi estruturada utilizando modelagem dimensional.
Foram construídas as seguintes tabelas:
- gold_dim_musica
- gold_dim_artista
- gold_dim_tempo
- gold_bridge_musica_artista
- gold_fato_desempenho_musica
A tabela fato concentra as métricas de desempenho das músicas. As dimensões organizam informações de música, artista e tempo.
Como uma música pode estar associada a mais de um artista e um artista pode participar de várias músicas, foi criada a tabela gold_bridge_musica_artista para representar essa relação muitos-para-muitos.
O notebook contém o catálogo de dados com a identificação das camadas, tabelas, atributos, tipos e possibilidade de valores nulos.
Pipeline de Dados
O pipeline foi implementado em um único notebook no Databricks, permitindo acompanhar o fluxo completo dos dados e manter a rastreabilidade das transformações.
Na Bronze, os dados são ingeridos e preservados.
Na Silver, são executados os tratamentos necessários para disponibilizar uma estrutura consistente para modelagem.
Na Gold, são criadas e persistidas as dimensões, tabela fato e bridge utilizadas nas análises.
O notebook também contém verificações de integridade referencial entre as estruturas da camada Gold.
Qualidade de Dados
A qualidade foi analisada antes das transformações.
Foram verificadas, entre outras características:
- valores ausentes;
- registros duplicados;
- tipos de dados;
- valores percentuais fora dos intervalos esperados;
- valores quantitativos negativos;
- intervalos de mês e dia;
- campos numéricos recebidos como texto;
- registros incompatíveis com os tipos esperados.
Foram identificados valores ausentes nos campos key e in_shazam_charts. Esses valores foram preservados quando não havia evidência suficiente para justificar uma imputação.
Também foi identificado um registro com conteúdo incompatível no campo streams. O valor não foi artificialmente corrigido e foi mantido como NULL após o tratamento.
Análise de Dados
P1 — Músicas e artistas associados aos maiores volumes de streams
Blinding Lights, de The Weeknd, apresentou o maior volume de streams da base, com aproximadamente 3,70 bilhões de reproduções.
Na análise por artista, considerando os streams das músicas às quais cada artista está associado, The Weeknd aparece com aproximadamente 23,93 bilhões de streams associados a 37 músicas.
P2 — Relação entre presença em playlists e charts e volume de streams
Foi observada associação positiva entre presença em playlists e volume de streams.
A relação foi mais forte para playlists do Spotify, com correlação próxima de 0,79, seguida pelas playlists da Apple Music e Deezer. As associações encontradas para charts foram mais fracas.
Esses resultados representam associação entre as variáveis e não permitem concluir causalidade.
P3 — Desempenho por período de lançamento
A comparação entre períodos mostrou diferenças de desempenho, mas também evidenciou a necessidade de considerar a concentração de músicas recentes na base e o tempo disponível para acumulação de streams.
Os resultados devem, portanto, ser interpretados dentro da composição do conjunto de dados analisado.
P4 — Características musicais e nível de streams
Foram comparadas características como danceability, energy e valence entre grupos de músicas classificados pelo volume de streams.
As diferenças observadas foram relativamente pequenas e, isoladamente, não indicaram um perfil musical claramente associado aos maiores volumes de reprodução.
P5 — Concentração do consumo
A análise mostrou concentração relevante dos streams:
- Top 10% das músicas: 36,65% dos streams
- Top 20%: 57,16%
- Top 50%: 85,76%
Os resultados indicam que uma parcela relativamente pequena das músicas concentra uma proporção significativa das reproduções presentes na base.
P6 — Associação entre desempenho e recepção dos usuários
Foi testado o enriquecimento dos dados utilizando o MusicBrainz.
Três músicas foram utilizadas para validar a integração: Blinding Lights, Shape of You e As It Was. As três foram corretamente identificadas, com score_matching igual a 100.
Entretanto, a disponibilidade de avaliações foi insuficiente. Blinding Lights e Shape of You apresentaram 0 votos, enquanto As It Was apresentou rating 5 com apenas 1 voto.
Dessa forma, a P6 não foi respondida quantitativamente. A limitação foi documentada em vez de produzir uma conclusão não sustentada pelos dados.
Autoavaliação
O pipeline permitiu percorrer as principais etapas previstas para o MVP: ingestão, persistência da Bronze, análise de qualidade, preparação da Silver, modelagem dimensional da Gold e utilização das estruturas resultantes para responder às perguntas analíticas.
As cinco primeiras perguntas puderam ser respondidas com os dados disponíveis. A sexta exigia enriquecimento externo e foi mantida até o final para avaliação de sua viabilidade.
A integração com o MusicBrainz mostrou que a correspondência entre música e artista era tecnicamente possível, mas a baixa cobertura das avaliações impediu uma análise quantitativa consistente.
Entre as limitações do projeto também estão a composição do dataset, que reúne músicas de destaque e não representa o universo completo de lançamentos, e a documentação da fonte principal, que informa a utilização de múltiplas fontes sem detalhá-las individualmente.
Como evolução, o projeto poderá incorporar uma fonte externa com maior cobertura de avaliações, ampliar o universo e período das músicas analisadas e automatizar verificações adicionais de qualidade e atualização dos dados.
Arquivos do Repositório
O notebook final do MVP é:
00_MVP_Engenharia_Dados_Spotify_27092026_vF.ipynb
Ele contém o código executável, documentação do pipeline, evidências das tabelas persistidas, catálogo de dados e resultados das análises.
