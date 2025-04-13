# MVP_EngDados_PuC
MVP - Engenharia de Dados (40530010057_20250_01)

## Objetivo

Desafio: “Os Indicadores da agência reguladora de telecomunicações (Anatel), que compõe o IQS (Índice de Qualidade do Serviço), e o número de reclamações por problemas técnicos influenciam diretamente na variação da base de clientes das operadoras?”

25 anos após a privatização do setor de Telecomunicações, é sabido que os serviços de telefonia e banda larga fixos, assim como móveis, atingiram altos patamares de maturidade do ponto de vista de tecnologia e cobertura. As áreas comerciais das operadoras seguem constantemente se empenhando para que o cliente tenha uma boa experiência com atendimento e ofertas atrativas, e as áreas técnicas com qualidade de serviço e cobertura abrangente.

Neste MVP tentarei responder se os indicadores que compões o índice de qualidade do serviço (IQS) de Banda larga fixa (internet residencial) influenciam no número de reclamações técnicas na Anatel e se, consequentemente, impactam na manutenção de número de clientes na base.

Entende-se que o número de reclamações na agência reguladora é um termômetro de insatisfação com o serviço, e que o _churn_ (diminuição de base de clientes) é causado por muitos fatores. Mas será que os indicadores regulatórios do IQS são capazes de refletir o que acontece no mercado?

A partir de pandemia de covid em 2020, com a popularização do modelo de teletrabalho, aumento no uso de aplicativos de streaming, de rede social e dispositivos inteligentes (ie. Amazon Alexa), o serviço de banda larga fixa adquiriu um alto nível de exigência de seu público consumidor.

Neste MVP vou limitar o perímetro ao ano de 2024, aos três principais grupo de operadoras (Tim, Vivo e Claro), ao serviço Banda larga fixa (para a agência: SCM – serviço de comunicação multimídia) e à sete cidades capitais do Brasil.

As reclamações Anatel consideradas nesta avaliação são de motivo técnico (Qualidade, Funcionamento e Reparo, Instalação / Ativação ou Habilitação / Mudança de Endereço). Já os Indicadores SCM que compõe o Índice de Qualidade do Serviço (IQS) serão IND4, IND5, IND6, IND7, IND8 _de rede_, e o IN9 _de relacionamento_, conforme o resumo abaixo:

| Grupo          | Acrônimo | Indicador                                      | Descrição                                                                                                                                  |
|----------------|----------|------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| Redes          | IND4     | Cumprimento da velocidade de download e upload | Expressa a capacidade da rede em relação ao cumprimento das referências ou valores contratados de volume de dados transmitidos por segundo |
| Redes          | IND5     | Latência bidirecional da Conexão de Dados      | Expressa o desempenho da rede em relação ao tempo de transmissão de pacotes de dados                                                       |
| Redes          | IND6     | Variação de Latência da Conexão de Dados       | Expressa o desempenho da rede em relação à variação do tempo de transmissão de pacotes de dados                                            |
| Redes          | IND7     | Perda de Pacotes da Conexão de Dados           | Expressa a capacidade da rede de entregar os pacotes de dados ao destino sem ocorrência de perdas                                          |
| Redes          | IND8     | Disponibilidade                                | Expressa o tempo em que o serviço está em operação à disposição dos usuários sem interrupção                                              |
| Relacionamento | IND9     | Cumprimento de Prazo                           | Expressa o atendimento a solicitações de instalação, reparo e manutenção agendados e realizados dentro dos prazos acordados                |

Abaixo segue a lista dos valores de corte para indicadores de rede IND4 a IND7 para obtenção do resultado das medidas, onde o **resultado de medidas é o percentual de atingimento dos valores de corte**.

| Tecnologia | IND4 (DW) | IND4 (UP) | IND5  | IND6  | IND7 |
|------------|-----------|-----------|-------|-------|------|
| DSL        | 5 Mbit/s  | 1 Mbit/s  | 80 ms | 40 ms | 2%   |
| GPON       | 25 Mbit/s | 5 Mbit/s  | 80 ms | 40 ms | 2%   |

Valores de referência inferiores e superiores dos indicadores são informados pela Anatel, **sendo o intervalo aonde os resultados devem estar contidos**.

| Valor | IND4 | IND5  | IND6  | IND9  | IND8 |  IND9 |
|------------|-----------|-----------|-------|-------|------|------|
| VR sup        | 90%  | 95%  | 95% | 95% | 99,99% | 90% |
| VR inf       | 60% | 65%  | 65% | 65% | 98%  | 60% |


## Coleta, Modelagem e Carga

A coleta das informações foi feita do site da Anatel (anatel.gov.br) nas opções de Reclamações, Acessos banda larga fixa e Indicadores Rqual (Regulamento de Qualidade dos Serviços de Telecomunicações – RQUAL, aprovado pela Resolução nº 717/2019).

https://www.anatel.gov.br/dadosabertos/paineis_de_dados/consumidor/consumidor_reclamacoes.zip
https://www.anatel.gov.br/dadosabertos/paineis_de_dados/acessos/acessos_banda_larga_fixa.zip
https://www.anatel.gov.br/dadosabertos/paineis_de_dados/qualidade/indicadores_rqual.zip

Como os arquivos do site Anatel são grandes (~2G) e alguns possuem muitos arquivos históricos zipados, optei por pré-processá-los localmente, baixando os zips, dezipando, selecionando os arquivos do ano de 2024 e posteriormente fazendo upload para o DBFS (Databricks).

Usando como referência a arquitetura medalhão, os dados passaram pelas camadas bronze, prata e ouro até obter uma tabela final com os dados relevantes para a análise em Power BI da relação entre indicadores Anatel, número de clientes e reclamações técnicas.

Na camada bronze foram ajustados nomes de colunas com espaço e caracteres especiais, valores de data com padrões diferentes entre as 3 fontes e criação de nova coluna concatenando Ano+Mês+Operadora+CodIBGE para ser a chave entre as 3 tabelas.

Na camada prata foram criadas views selecionando tipo de produto internet na tabela de acessos, serviço banda larga fixa e indicadores IQS na tabela de indicadores, e reclamações de motivos técnicos (qualidade, funcionamento e reparação, instalação e mudança de endereço) para as reclamações Anatel. Limitando o escopo em 3 operadoras: Claro, Tim e Vivo, e em 7 capitais com número significativo de medidas para estas operadoras: Manaus, Brasília, São Paulo, Rio de Janeiro, Goiânia, Salvador e Belo Horizonte

![Modelo entidade-relacionamento](/images/modelo_entidade_relacionamento.png)

Finalmente na camada ouro foi feito um join entre as 3 views obtendo uma tabela final.

Na sequência foi extraído um arquivo da tabela **anatel_scm** a ser analisado em Power Bi, e elaborado seu respectivo **catálogo de dados “catalogo-de-dados_anatel_scm.xlsx”**.


## Análise

Com a tabela final foram criados em Power BI 7 dashboards, um para cada um dos 7 municípios selecionados expostos no documento **MVP_Anatel_2024_BI.pdf**.

Nos dashboards existem gráficos comparando as 3 operadoras para cada indicador proposto (IND4 ao IND9), contrastando também o número de clientes e número de reclamações técnicas na Anatel.

Para fim de comparações, no Power Bi foi criada a métrica “Reclamações a cada mil clientes” pois em alguns municípios o *marketshare* de uma operadora pode ser muito superior as outras, sendo possível com esta métrica avaliar as reclamações de forma proporcional ao número de clientes na base.

Desta forma, respondendo ao questionamento inicial proposto neste MVP, podemos avaliar para as 7 cidades se “Os Indicadores da agência reguladora de telecomunicações (Anatel), que compõe o IQS (Índice de Qualidade do Serviço), e o número de reclamações por problemas técnicos influenciam diretamente na variação da base de clientes das operadoras?”

 + Para **Belo Horizonte** (MG) do ponto de vista de indicadores de rede (IND4 ao IND8), a TIM e Claro lideram, porém para o indicador de atendimento (IND9) de solicitações de instalação, reparo e mudança de endereço realizados dentro dos prazos agendados, a TIM e a Vivo apresentam valores ruins impactando diretamente nas reclamações com a Anatel. Apesar disso, a base de clientes da Claro reduziu ao longo do ano, e TIM e Vivo aumentaram o número de acessos.

 + Para **Brasília** (DF) do ponto de vista de indicadores de rede (IND4 ao IND8), a TIM e Claro lideram, porém para o indicador de atendimento (IND9) de solicitações de instalação, reparo e mudança de endereço realizados dentro dos prazos agendados, a TIM e a Vivo apresentam valores ruins impactando diretamente nas reclamações com a Anatel. Apesar disto, a base de clientes de todas as operadoras aumentou.

 + Para **Goiânia** (GO) do ponto de vista de indicadores de rede (IND4 ao IND8), a TIM e Claro lideram, porém para o indicador de atendimento (IND9) de solicitações de instalação, reparo e mudança de endereço realizados dentro dos prazos agendados, a TIM e a Vivo apresentam valores ruins impactando diretamente nas reclamações com a Anatel. Apesar disto, a base de clientes da Claro reduziu ao longo do ano, e TIM e Vivo aumentaram o número de acessos.

 + Para **Manaus** (AM) do ponto de vista de indicadores de rede (IND4 ao IND8), todas as operadoras são excelentes, porém para o indicador de atendimento (IND9) de solicitações de instalação, reparo e mudança de endereço realizados dentro dos prazos agendados, a TIM apresenta valores ruins impactando diretamente nas reclamações com a Anatel. Apenas a base da Vivo cresceu em 2024. As reclamações da Claro são altas, porém a base de cliente é superior as outras.

 + Para o **Rio de Janeiro** (RJ) do ponto de vista de indicadores de rede (IND4 ao IND8), a TIM e Claro lideram, porém para o indicador de atendimento (IND9) de solicitações de instalação, reparo e mudança de endereço realizados dentro dos prazos agendados, para a TIM está crítico, impactando diretamente nas reclamações com a Anatel e também na diminuição da base de dados em 33% ao longo de 2024, enquanto Claro e Vivo aumentam o número de clientes.

 + Para o **Salvador** (BA) do ponto de vista de indicadores de rede (IND4 ao IND8), a TIM e Claro lideram, porém para o indicador de atendimento (IND9) de solicitações de instalação, reparo e mudança de endereço realizados dentro dos prazos agendados, para a TIM está bem abaixo das concorrentes, impactando diretamente nas reclamações com a Anatel e com discreta perda de clientes, enquanto Claro e Vivo aumentam o número de clientes.

 + Para o **São Paulo** (SP) do ponto de vista de indicadores de rede (IND4 ao IND8), todas as operadoras estão bem, porém para o indicador de atendimento (IND9) de solicitações de instalação, reparo e mudança de endereço realizados dentro dos prazos agendados, para a TIM está bem abaixo das concorrentes, impactando diretamente nas reclamações com a Anatel e com perda de 14% de clientes em 2024, enquanto Claro e Vivo aumentam o número de clientes.

Diante do exposto, ainda que o _churn_ (diminuição de base de clientes) seja composto por muitos fatores como qualidade de rede, relacionamento, fatura, ofertas locais, etc., a análise das 7 capitais propostas sugeriu que onde o atendimento (IND9 – cumprimento de prazo para reparos, instalações e mudanças de endereços) está em níveis críticos, há sim aumento significativo de reclamações na Anatel e queda na base de clientes, mesmo com valores de qualidade de rede bons (download, upload, jitter, latência e perda de pacotes). A operadora TIM para Rio de Janeiro e São Paulo ilustram esta conclusão.

O resultado é compreensível porque com o aumento de demandas de estudos e trabalhos remotos, uso de rede sociais, dispositivos inteligentes (i.e. Amazon Alexa), streaming, o usuário está cada vez mais exigente e não tem muito tolerância para aguardar por muito tempo a visita de um técnico para fazer reparos na internet residencial.

Já para as cidades de Belo Horizonte, Brasília, Goiânia, Manaus e Salvador as taxas de reclamação não afetaram nada ou muito pouco a base de clientes, o que me leva a sugerir que outros fatores como ofertas/promoções podem estar retendo clientes.

Outra conclusão a ser ressaltada, é que as redes de banda larga fixa já possuem uma maturidade do ponto de vista de tecnologia. Pois os indicadores de rede, salvo a Vivo em perda de pacotes, ao longo do ano de 2024 se mantiveram em patamares altos de qualidade.


## Autoavaliação

Após a análise das informações dentro do perímetro proposto (3 operadoras, 7 cidades) fiquei satisfeita com o resultado. O IND9 se mostrou como o KPI mais impactante para as reclamações técnicas na Anatel e para a manutenção da base de clientes, principalmente para as cidades de Rio de Janeiro e São Paulo.

Uma limitação que eu gostaria de ter procurado mais opções para ajustar a tempo do MVP seria uma carga das tabelas no portal Anatel sem precisar fazer upload manual dos arquivos no DBFS do Databricks Community Edition. O tamanho dos arquivos do site Anatel dificultaram o desempenho já que o tempo do cluster é limitado. Outro tema que entendi não ser possível de solucionar neste momento, por causa da minha licença limitada de Power BI, foi a integração do resultado da tabela final do Databricks ao Power BI (Conexão databrick ao Power Bi apenas com conta premium: https://docs.databricks.com/aws/en/partners/bi/power-bi).

Para uma avaliação mais completa futura, entendo que outros motivos de reclamação na Anatel além dos técnicos e valores de planos/ofertas deveriam ser analisados. Variações de temperatura e chuvas também afetam a qualidade do serviço, então informações meteorológicas agregariam positivamente no estudo.

No módulo do machine Learning entendo que será possível explorar uma quantidade maior de KPIs com a finalidade de entender qual teria mais peso na insatisfação do cliente.
