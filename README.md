# MVP_EngDados_PuC
MVP - Engenharia de Dados (40530010057_20250_01)


# Objetivo

Questionamento: “Os Indicadores da agência reguladora de telecomunicações - Anatel que compõe o IQS (Índice de Qualidade do Serviço) e o número de reclamações por problemas técnicos influenciam diretamente na variação da base de clientes das operadoras?”
25 anos após a privatização do setor de Telecomunicações, é sabido que os serviços de telefonia e banda larga fixos assim como móveis atingiram altos patamares de maturidade do ponto de vista de tecnologia e cobertura. As áreas comerciais das operadoras seguem constantemente se empenhando para que o cliente tenha uma boa experiência com atendimento e ofertas atrativas, e as áreas técnicas com qualidade de serviço e cobertura abrangente.
Neste MVP tentaremos responder se os indicadores que compões o índice de qualidade do serviço (IQS) de Banda larga influenciam no número de reclamações técnicas na Anatel e se consequentemente impactam na manutenção de número de clientes na base.
Entende-se que o número de reclamações na agência reguladora é um termômetro de insatisfação com o serviço, e que o Churn é causado por muitos fatores, mas será que os indicadores regulatórios do IQS são capazes de refletir                 o que acontece no mercado? 
A partir de pandemia de covid em 2020, com a popularização do modelo de teletrabalho, aumento de  aplicativos de streaming, rede social, dispositivos inteligentes (ex. alexa), o serviço de banda larga fixa adquiriu um alto nível de exigência de seu público consumidor.
Neste MVP vou limitar o perímetro aos três principais grupo de operadoras (Tim, Vivo e Claro), ao serviço Banda larga fixa (para a agência SCM – serviço de comunicação multimídia) e sete municípios capitais do Brasil. 
As reclamações Anatel consideradas nesta avaliação são de motivo técnico (Qualidade, Funcionamento e Reparo, Instalação | Ativação ou Habilitação | Mudança de Endereço) e os Indicadores SCM que compõe o Índice de Qualidade do Serviço (IQS) que são os de rede -IND4, IND5, IND6, IND7, IND8- e o de relacionamento- IND9, de acordo com o resumo abaixo:

Abaixo segue a lista dos valores de corte para indicadores de rede IND4  a IND7 para obtenção do resultado das medidas, onde o resultado de medidas é o percentual de atingimento dos valores de corte.
 


 


# Coleta, Modelagem e Carga

A coleta das informações foi feita diretamente do site da Anatel (anatel.gov.br) nas opções de Reclamações, Acessos banda larga fixa e indicadores Rqual (Regulamento de Qualidade dos Serviços de Telecomunicações – RQUAL, aprovado pela Resolução nº 717/2019).
https://www.anatel.gov.br/dadosabertos/paineis_de_dados/consumidor/consumidor_reclamacoes.zip
https://www.anatel.gov.br/dadosabertos/paineis_de_dados/acessos/acessos_banda_larga_fixa.zip
https://www.anatel.gov.br/dadosabertos/paineis_de_dados/qualidade/indicadores_rqual.zip
Os arquivos foram salvos em diretório local, e na sequência feito upload para o DBFS – Databricks.
Optei pela solução de pré-processar os arquivos de entrada, baixando na minha máquina local os zips, dezipando, e fazendo upload para o DBFS, pois recebi alguns erros no Community que não resolvi a tempo do MVP. 
Usando como referência a arquitetura medalhão, os dados passaram pelas camadas bronze, prata e ouro até obter uma tabela final com os dados relevantes para a análise em powerbi da relação entre indicadores Anatel, número de clientes e reclamações técnicas.
Na camada bronze foram ajustados nomes de colunas com espaço e caracteres especiais, valores de data com padrões diferentes entre as 3 fontes e criação de nova coluna concatenando Ano+Mês+Operadora+CodIBGE para ser a chave primária entre as 3 tabelas.
Na camada prata foram criadas views selecionando tipo de produto internet para a tabela de acessos, serviço banda larga fica e indicadores IQS para a tabela de indicadores, e reclamações de motivos técnicos (qualidade, funcionamento e reparação, instalação e mudança de endereço) para as reclamações Anatel. Limitando o escopo em 3 operadoras: Claro, Tim e Vivo, e em 7 capitais com número significativo de medidas para todas as operadoras: Manaus, Brasília, São Paulo, Rio de Janeiro, Goiânia, Salvador e Belo Horizonte 
 

Finalmente na camada ouro foi feito um Join entre as 3 views obtendo uma tabela final.
Na sequência foi extraído um arquivo “anatel_scm_2024.csv” a ser analisado em powerbi, e elaborado seu respectivo catálogo de dados “catalogo-de-dados_anatel_scm.xlsx”.


# Análise

Finalmente com a tabela final (anatel_scm_2024.csv) foram criados em powerbi 7 dashboards, um para cada município selecionado.
Nos dashboards existem gráficos comparando as 3 operadoras para cada indicador proposto (IND4 ao IND9), contrastando também o número de clientes e número de reclamações técnicas na Anatel (MVP_Anatel_2024_BI.pdf). 
Para fim de comparações, no Powerbi foi criada a métrica “Reclamações a cada mil clientes” pois em alguns municípios o marketshare de uma operadora pode ser muito superior as outras, sendo possível com esta métrica avaliar as reclamações de forma proporcional ao número de clientes na base.
Desta forma, respondendo ao questionamento inicial proposto neste MVP, podemos responder cidade a cidade: “Os Indicadores da agência reguladora de telecomunicações - Anatel que compõe o IQS (Índice de Qualidade do Serviço) e o número de reclamações por problemas técnicos influenciam diretamente na variação da base de clientes das operadoras?”

•	Para Belo Horizonte (MG) do ponto de vista de indicadores de rede (IND4 ao IND8), a TIM e Claro lideram, porém para o indicador de atendimento (IND9) de solicitações de instalação, reparo e mudança de endereço realizados dentro dos prazos agendados, a TIM e a Vivo apresentam valores ruins impactando diretamente nas reclamações com a Anatel. Apesar disso, a base de clientes da Claro reduziu ao longo do ano, e TIM e Vivo aumentaram o número de acessos.
•	Para Brasília (DF) do ponto de vista de indicadores de rede (IND4 ao IND8), a TIM e Claro lideram, porém para o indicador de atendimento (IND9) de solicitações de instalação, reparo e mudança de endereço realizados dentro dos prazos agendados, a TIM e a Vivo apresentam valores ruins impactando diretamente nas reclamações com a Anatel. Apesar disto, a base de clientes de todas as operadoras aumentou.
•	Para Goiânia (GO) do ponto de vista de indicadores de rede (IND4 ao IND8), a TIM e Claro lideram, porém para o indicador de atendimento (IND9) de solicitações de instalação, reparo e mudança de endereço realizados dentro dos prazos agendados, a TIM e a Vivo apresentam valores ruins impactando diretamente nas reclamações com a Anatel. Apesar disto, a base de clientes da Claro reduziu ao longo do ano, e TIM e Vivo aumentaram o número de acessos.
•	Para Manaus (AM) do ponto de vista de indicadores de rede (IND4 ao IND8), todas as operadoras são excelentes, porém para o indicador de atendimento (IND9) de solicitações de instalação, reparo e mudança de endereço realizados dentro dos prazos agendados, a TIM apresenta valores ruins impactando diretamente nas reclamações com a Anatel. Apenas a base da Vivo cresceu em 2024. As reclamações da Claro são altas, porém a base de cliente é superior as outras.
•	Para o Rio de Janeiro (RJ) do ponto de vista de indicadores de rede (IND4 ao IND8), a TIM e Claro lideram, porém para o indicador de atendimento (IND9) de solicitações de instalação, reparo e mudança de endereço realizados dentro dos prazos agendados, para a TIM está crítico, impactando diretamente nas reclamações com a Anatel e também na diminuição da base de dados em 33% ao longo de 2024, enquanto Claro e Vivo aumentam o número de clientes.
•	Para o Salvador (BA) do ponto de vista de indicadores de rede (IND4 ao IND8), a TIM e Claro lideram, porém para o indicador de atendimento (IND9) de solicitações de instalação, reparo e mudança de endereço realizados dentro dos prazos agendados, para a TIM está bem abaixo das concorrentes, impactando diretamente nas reclamações com a Anatel e com discreta perda de clientes, enquanto Claro e Vivo aumentam o número de clientes.
•	Para o São Paulo (SP) do ponto de vista de indicadores de rede (IND4 ao IND8), todas as operadoras estão bem, porém para o indicador de atendimento (IND9) de solicitações de instalação, reparo e mudança de endereço realizados dentro dos prazos agendados, para a TIM está bem abaixo das concorrentes, impactando diretamente nas reclamações com a Anatel e com perda de 14% de clientes em 2024, enquanto Claro e Vivo aumentam o número de clientes.

Diante do exposto, ainda que o churn (diminuição de base de clientes) seja composto por muitos fatores como qualidade de rede, relacionamento, fatura, ofertas locais, etc., a análise das 7 capitais propostas sugeriu que onde o atendimento (IND9 – cumprimento de prazo para reparos, instalações e mudanças de endereços) está em níveis críticos, há sim aumento significativo de reclamações na Anatel e queda na base de clientes (operadora), mesmo tendo valores de qualidade de rede estando bons (download, upload, jitter, latência e perda de pacotes). A operadora TIM para Rio de Janeiro e São Paulo ilustram esta conclusão.
É um resultado compreensível porque com o aumento de demandas de estudos e trabalhos remotos, uso de rede sociais, dispositivos inteligentes (i.e. Alexa), streaming, o usuário está cada vez mais exigente e não tem mais disponibilidade de aguardar tanto tempo a visita de um técnico para fazer reparos na “internet de casa”.
Já para as cidades de Belo horizonte, Brasília, Goiânia, Manaus e Salvador as taxas de reclamação não afetaram nada ou muito pouco a base de clientes, o que me leva a sugerir que outros fatores como ofertas/promoções podem estar retendo clientes.
Outra conclusão a ser ressaltada, é que as redes de banda larga fixa já possuem uma maturidade do ponto de vista de tecnologia. Pois os indicadores de rede, salvo a Vivo em perda de pacotes, ao longo do ano de 2024 se mantiveram em patamares altos de qualidade.


# Autoavaliação

Após a análise das informações dentro do perímetro proposto (3 operadoras, 7 cidades) fiquei satisfeita com o resultado. O IND9 se mostrou como o KPI mais impactante para as reclamações técnicas na Anatel e para a manutenção da base de clientes, principalmente para as cidades de Rio de Janeiro e São Paulo.

Duas limitações que eu gostaria de ter ajustado a tempo do MVP seriam  a carga do portal Anatel sem precisar salvar em diretório local, dezipar e fazer upload dos arquivos no DBFS, e integração do resultado do Databricks ao Power BI (Conexão databrick ao powerbi apenas com conta premium:                  https://docs.databricks.com/aws/en/partners/bi/power-bi).

Para uma avaliação mais completa futura, entendo que outros motivos de reclamação na Anatel além dos técnicos e valores de Planos/ofertas deveriam ser analisados. Variações de temperatura e chuvas também afetam a qualidade do serviço, então informações meteorológicas agregariam positivamente no estudo.
No módulo do machine Learning entendo que será possível explorar melhor uma quantidade maior de KPIs com a finalidade de entender qual teria mais peso na insatisfação do cliente.




