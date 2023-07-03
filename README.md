# Projeto_de_Big_Data_Analytics
**Projeto de Big Data desenvolvido durante o curso de Banco de Dados da PUC Minas**

O objetivo deste trabalho é fazer uma análise histórica sobre a variação de preços dos combustíveis com o objetivo de aumentar a compreensão sobre como essa variação impacta nos custos da companhia. Além disso, pretende-se entregar uma análise sobre as tendências futuras para esses preços no próximo semestre, permitindo que o cliente faça um planejamento de custos mais adequado.  

Alguns dos objetivos chaves e análises que serão realizadas ao longo do projeto são: 

• Quantidade de postos pesquisados.  

• Evolução do preço médio, máximo e mínimo de revenda ao longo do tempo. 

• Preço de revenda por região e estados do Brasil. 

• Análise de qual produto teve maior variação (%) de preço.  

• Interessante explorar outras informações que possam complementar a análise e ter algum impacto / relação com o aumento dos preços, como: Histórico do PIB, variação do IPCA, ocorrência de eventos mundiais (ex.pandemia, guerras), valor do salário mínimo, cotação do dólar, mudança na política de preço da Petrobras entre outros. 

**ARQUITETURA**

Para definir a arquitetura do nosso projeto foi necessário fazer um planejamento sobre a estrutura necessária e definir qual seria a lógica seguida. Com base nisso, definimos a estrutura que está exposta abaixo em um esquema ilustrativo, representado pela figura 1. 

Na figura 1, a fonte de dados é composta pela base de dados disponibilizada no site da ANP (Agência Nacional do Petróleo) e que pode ser consultada através do site: https://www.gov.br/anp/pt-br/assuntos/precos-e-defesa-da-concorrencia/precos/precos-revenda-e-de-distribuicao-combustiveis/serie-historica-do-levantamento-de-precos . A base de dados encontra-se em formato XML no Excel e traz o levantamento semanal de preços dos combustíveis estratificada por munícipios.  

Ainda na figura 1, usamos a solução de armazenamento da Microsoft para cloud o Blob para armazenar as planilhas exportadas do site da ANP e para integração e transformação de dados usamos do serviço Data Factory. Após tratadas a tabela final foi carregada no Synapse Analytics. 

![image](https://github.com/RafaBBoaventura/Projeto_de_Big_Data_Analytics/assets/131798428/36d78165-4764-47c2-ba8a-98a6bf251add)

**METODOLOGIA**

Foi criado na plataforma Azure um grupo de recurso com as ferramentas e serviços que serão utilizados no projeto.

![image](https://github.com/RafaBBoaventura/Projeto_de_Big_Data_Analytics/assets/131798428/d57a6153-a415-4abe-b0f0-81a8120c5ba4)

A base de dados definida está no formato CSV, extraída do site da ANP, foi importada para Blob. Para importar o arquivo para o Blob foi necessário a criação de um recurso através das storage accounts (contas de armazenamento).  Após acessar a conta de armazenamento clique em +criar para fazer a criação. Em seguida acesse a conta de armazenamento criada e selecione a opção contêineres. Foi criado então um novo contêiner com o nome dadosbrutos e dentro dele realizado  o upload das tabelas que extraídas do site ANP. Em seguida no Data Factories foi criado uma nova pipeline com o nome ProjetoPUC. Em conjuntos de dados foi criado um fluxo de dados (dataflow) com o nome carregados. Dentro desse fluxo de dados foi importado as tabelas de dados que estavam armazenadas no blob. E foi realizado o tratamento das tabelas onde foi feito exclusão das colunas que não seriam necessarias para o projeto (5 ultimas colunas conforme mostra a figura abaixo) e também a união dos dados transformando-os em uma única tabela chamada tabela final.

![image](https://github.com/RafaBBoaventura/Projeto_de_Big_Data_Analytics/assets/131798428/3a2708d3-139f-4c5c-9266-1e27ae8794c9)

Essa tabela final foi alocada no Synapse Analytics com todos os dados já tratados para etapa final de visualização.

**ANÁLISE E VISUALIZAÇÃO DOS DADOS**

Para a etapa de análise e visualização de dados, ficou definido que seria utilizado o Power Bi como a ferramenta para fazer a transformação dos dados, construção do dashboard e posterior análise dos resultados. 

Como os dados fornecidos pela ANP trazem dados de diversos combustíveis diferentes, foi necessário fazer um recorte nesses dados para analisar apenas os combustíveis relevantes para o negócio do cliente. Dessa forma optou-se por considerar apenas os dados da gasolina (comum e aditivada), utilizadas para as entregas dentro dos centros urbanos e os dados do diesel (comum e S-10), utilizados para as entregas intermunicipais e interestaduais. Os dados dos demais tipos de combustíveis não foram utilizados. Essa etapa foi feita previamente dentro do Synapse Analytics do Azure e os dados já editados foram carregados no Power Bi.

No ambiente do Power Bi decidiu-se por construir dois dashboards diferentes, um para analisar os dados referentes a gasolina e outro para analisar os dados referentes ao diesel. Todas as demais transformações foram iguais para os dois combustíveis. Na figura abaixo temos o histórico de todas as transformações que foram feitas: 

![image](https://github.com/RafaBBoaventura/Projeto_de_Big_Data_Analytics/assets/131798428/e7e63828-aa7c-4fe3-8402-2e29f9c3254e)

A primeira transformação necessária foi unificar as tabelas de preços e transformá-las em uma única tabela para cada combustível. Essa tabela foi renomeada para BDGasolina e BDDiesel.

O próximo passo foi remover as colunas que não seriam utilizadas durante a análise. No caso das informações sobre a gasolina e o diesel foram removidas as colunas que traziam os preços dos distribuidores (esses dados só foram coletados até 2021) e a coluna referente a margem média de revenda que também não foram mais coletados após 2021. 

Após remover as colunas, foi criado um índice para registrar a id de cada dado coletado e esse índice foi deslocado para o início da planilha, para ser a primeira coluna do bando de dados (o mesmo passo foi feito no BDGasolina e BDDiesel).

Observou-se que existiam vários campos em branco nas colunas Preço Médio Revenda, Desvio Padrão Revenda, Preço Mínimo Revenda, Preço Máximo Revenda e Coeficiente de Variação Revenda. Como a coleta desses dados é feito via pesquisa nos postos de combustível, considerou-se que a melhor alternativa para tratar os dados faltantes seria remover as linhas que não possuíam valores. A justificativa para essa decisão deveu-se ao fato de que utilizar algum cálculo estatístico iria interferir mais nos valores da análise do que a remoção das linhas com dados faltantes. 

Após essas transformações adicionou-se a tabela com os dados sobre a variação da cotação do barril de petróleo de tipo Brent no mercado internacional. Esses dados são provenientes da bolsa de valores de Londres e sua periodicidade é diária, em todos os dias úteis. Essa tabela foi renomeada para dPrecosPetróleo. 

Para fazer a modelagem dos dados e construir os relacionamentos entre as tabelas, optou-se pela criação de uma tabela de calendário através do seguinte comando DAX:

![image](https://github.com/RafaBBoaventura/Projeto_de_Big_Data_Analytics/assets/131798428/0e7914a7-fdf4-49c1-89ef-7dff15f03a9d)

A modelagem do banco de dados criado no Power Bi foi a seguinte: 

![image](https://github.com/RafaBBoaventura/Projeto_de_Big_Data_Analytics/assets/131798428/56bb353a-4775-4525-b180-fbeec75bba60)

Para atender aos objetivos do trabalho, definiu-se as seguintes análises: 
    • contagem do número de postos pesquisados; 
    • variação média do preço da gasolina;
    • variação média do preço do diesel;
    • variação ano a ano do preço médio da gasolina;
    • variação ano a ano do preço médio do diesel;
    • média do preço do petróleo Brent;
    • variação ano a ano do preço médio do petróleo Brent;
    • uma análise comparativa dos preços médios da gasolina/diesel x preços médios do petróleo;
    • preço médio da gasolina/diesel por região;
    • preços máximos da gasolina/diesel por estado.

![image](https://github.com/RafaBBoaventura/Projeto_de_Big_Data_Analytics/assets/131798428/8e21eed7-bbc1-46c5-87b4-7426502daac2)
![image](https://github.com/RafaBBoaventura/Projeto_de_Big_Data_Analytics/assets/131798428/40d927f9-90a9-479a-a873-0a8cf0ba18d9)
![image](https://github.com/RafaBBoaventura/Projeto_de_Big_Data_Analytics/assets/131798428/c5bc1843-5a05-49c4-92d3-ae49f62d5807)

O dashboard foi publicado online e pode ser consultado no seguinte link: https://app.powerbi.com/view?r=eyJrIjoiZGZlYzc1MGQtNDYwNS00Y2MwLTg5NTQtYTJjNzZiZDRlNjY4IiwidCI6IjE0Y2JkNWE3LWVjOTQtNDZiYS1iMzE0LWNjMGZjOTcyYTE2MSIsImMiOjh9

**ANÁLISE DOS RESULTADOS**

No dashboard que analisa as informações da gasolina, foram pesquisados 136 mil postos de combustível em todos os 27 estados e o Distrito Federal. O preço médio da gasolina no país no período de 2018 a maio de 2023 foi de R$5,35 e a variação ano a ano desse preço médio foi de –0,15%. 

No dashboard que analisa as informações do diesel, foram pesquisados 152 mil postos de combustível em todos os 27 estados e o Distrito Federal. O preço médio da gasolina no país no período de 2018 a maio de 2023 foi de R$3,78 e a variação ano a ano desse preço médio foi de 0,0%.

No período pesquisado (2018 a maio de 2023) o preço médio do barril de petróleo no mercado internacional foi de $70,55 dólares e a variação ano a ano do preço médio foi de +1,09%. O advento da pandemia contribuiu fortemente para a grande redução do preço no ano de 2020 e a guerra entre Rússia e Ucrânia e a retomada da atividade econômica no mundo contribuiu para a alta nos anos de 2021 e 2022. Até o momento o preço do barril do petróleo está em queda e a mais aceita explicação está no fraco crescimento da economia mundial. 

Quando falamos sobre a definição dos preços dos combustíveis no Brasil é necessário observar que temos uma legislação que pode mudar a partir de decisões políticas. A partir de outubro de 2016, a Petrobras adota a política de Paridade de Importação (PPI), que vincula o preço dos derivados do petróleo nas refinarias ao comportamento do preço do produto em dólares no mercado internacional. Como nessa análise estamos utilizando o preço dos revendedores, é importante ainda analisar outros fatores como custo do frete para entrega desses combustíveis, oferta e demanda local, entre outros. 

Fazendo uma comparação entre os preços médios tanto da gasolina quanto do diesel em relação ao preço do barril do petróleo no mercado internacional, observa-se uma variação em linha com a mudança do preço internacional, entretanto os aumentos são claramente mais repassados do que as eventuais quedas no preço. Vivemos um período de redução no preço médio entre 2018 e 2020 e um forte aumento nos anos de 2021 e 2022. No caso dos preços da gasolina (temos dados até maio de 2023) observa-se uma queda nos preços. Vale ressaltar que a partir de maio desse ano a política de Paridade de Importação foi descontinuada e espera-se uma flutuação menor nos preços dos combustíveis e redução geral nos preços. 

Em relação ao preço médio nas regiões, no caso tanto da gasolina quanto do diesel as regiões com os maiores preços são a região Norte e a região Nordeste, seguidas por Centro-Oeste, Sudeste e Sul. A explicação disso deve-se ao fato de termos uma maior concentração de refinarias nas regiões Sudeste e Sul e as maiores dificuldades logísticas das demais regiões, o que acaba encarecendo o preço dos produtos ao consumidor final. 

Em relação aos preços máximos por estado, no caso da gasolina, os três estados com os maiores preços máximos são: Acre, Rio de Janeiro e Distrito Federal. No caso do diesel são: Acre, Amapá e Mato Grosso. A explicação para esses resultados não é tão óbvia para o caso da gasolina e os dados obtidos neste projeto não são suficientes para responder com assertividade a isso. Seria necessário pesquisar mais a fundo sobre o mercado consumidor de cada um desses estados. Para o caso do diesel, a hipótese mais provável diz respeito a distância dos centros consumidores do Acre e do Amapá das distribuidoras mais próximas e no caso do Mato Grosso possivelmente a grande demanda por esse tipo de combustível para as máquinas agrícolas e para o escoamento da produção de grãos desse estado. 

Com a mudança na política de preços da Petrobras e a intenção manifestada pelo governo eleito de aumentar os investimentos em refino e produção, além das notícias vindas do mercado internacional, acredita-se que teremos quedas consistentes dos preços da gasolina e do diesel no mercado, o que representa uma boa notícia para os transportadores e para a economia em geral, dado o peso dos custos do transporte rodoviário nos índices de inflação. 

**OBS: para construir os backgrounds dos dashboards foi utilizada a ferramenta Figma.**






