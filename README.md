![_6d364e2b-af16-4955-8b33-f2130cef11c4](https://github.com/GabrielTavaresGoldman/Analise-Dados-Credito/assets/149710830/51c3fb98-ae41-49ba-af1b-480ba9c0fc41)
# Explorando Relações entre Dados Demográficos e Financeiros
 A análise abaixo foi realizada utilizando um conjunto de dados de crédito de uma instituição financeira público utilizado nos cursos Carreira de Analista de Dados, Python e SQL da EBAC.
O objetivo da análise é compreender os padrões de clientes de uma instituição financeira, utilizando dados demográficos, comportamentais e financeiros. Essa compreensão pode ser crucial para diversas áreas, como marketing, atendimento ao cliente e gestão de riscos.
Perguntas que queremos responder: 
- Qual é a distribuição da escolaridade entre os clientes? 
- Como é a distribuição do salário dos clientes? 
- Clientes que realizam um maior número de transações ao longo do ano tendem a efetuar transações de valores mais elevados? 
- Quanto dos nossos clientes estão em situação de inadimplência? 
- Como é a distribuição dos limites de créditos dos clientes? E qual a relação desses limites com o salário anual deles? 
- O salário dos clientes tem impacto com a inatividade deles? 
- Clientes com salários maiores, tem quantidades maiores de transações? E possuem maiores limites? 
- Clientes com salários menores, tem uma probabilidade de ficarem mais meses inativos? 
- Clientes mais velhos possuem maiores limites de crédito? 
- Clientes com faixas salariais maiores tem propensão ao Default menor? 
# 2. Carregando Bibliotecas 
```
import pandas as pd 
import matplotlib.pyplot as plt 
import seaborn as sns 
import numpy as np
 ``` 
# 3. Carregando Dataset 
```
caminho = r"C:\Users\gabriel.goldman\Área de Trabalho\Modelo Crédito\credito.csv"
credito = pd.read_csv(caminho) 
``` 
# 4. Limpeza e Processamento dos Dados 
Nessa etapa primeiro identifiquei quais são os tipos dos dados, as variáveis 'limite_credito' e 'valor_transacoes_12m' estavam em um formato de texto, então converti para numérico e como o python entende os valores decimais apenas com ponto e não por vírgula, substitui os mesmos. Depois verifiquei se havia valores nulos (não havia). 
```
credito.dtypes 
credito['limite_credito'] = pd.to_numeric(credito['limite_credito'].str.replace('.', '').str.replace(',', '.')) 
credito['valor_transacoes_12m'] = pd.to_numeric(credito['valor_transacoes_12m'].str.replace('.', '').str.replace(',', '.'))
print(credito.isnull().sum())
```
 ### Entendendo os dados 
Além de identificar os dados, preciso entender com quais dados eu irei trabalhar, então separei as variáveis categóricas e analisei os valores únicos, percebi que existiam respostas 'na' que provavelmente foram de clientes que não preencheram todos os dados, e por ser uma análise demográfica, decidi então criar um Data Frame sem esses valores (que poderiam distorcer a análise). 
``` 
categoricas = ['sexo','escolaridade','estado_civil','salario_anual', 'tipo_cartao'] 
for coluna in categoricas:
 valores_unicos = credito[coluna].unique() 
print(f'Valores únicos em {coluna}: {valores_unicos}') 
credito1 = credito.loc[(credito['escolaridade'] != 'na') & (credito['salario_anual'] != 'na') & (credito['estado_civil'] != 'na')] 
```
# 5. Análises 
## 5.1 Análise Univariada
### Idade 
Analisando as idades dos nossos clientes foi possível concluir que a grande parte dos clientes possuem de 41 a 52 anos de idade, com uma média e mediana de 46 anos. 
```
print('media:',credito1['idade'].mean()) # 46 anos 
print('mediana:', credito1['idade'].median()) # 46 anos 
sns.boxplot(credito1['idade']) 
plt.show() 
``` 
![Figure_1](https://github.com/GabrielTavaresGoldman/Analise-Dados-Credito/assets/149710830/2f9a1612-62fa-4ff9-b20c-093c6b00c6ce)
#### Aqui é importante analisarmos tanto a média como a mediana, pois no gráfico vemos a presença de outliers (dados que são exceções/extremos) em idades mais velhas, o que poderia "puxar" a nossa média para cima. 
### Sexo 
Identifiquei que o sexo dos nossos clientes é bem semelhante sendo 52,3% são mulheres e 47,7% são homens. 
``` 
sexo = credito1['sexo'].value_counts()
sexo 
``` 
### Escolaridade 
Nessa análise foi possível observar que a grande parte dos clientes (37%) possuem mestrado, seguido de 23% que fizeram até o ensino médio, 17% não tem uma educação formal, 12% possuem graduação e 11% possuem doutorado. 
```
credito1['escolaridade'].value_counts(1) 
``` 
### Renda 
Na renda percebi que as respostas dos clientes foram baseadas em faixas de valores, então decidi realizar uma segmentação dos dados para dividir os clientes em 3 categorias (Baixa Renda - menos de $40K anual, Média Renda - Entre $40K - $120 anual e Alta Renda - mais de $120K anual). 
``` 
credito1['salario_anual'].value_counts() 
credito1['categoria_cliente'] = np.where(credito1['salario_anual']=='menos que $40K', 'baixa renda', 
np.where(credito1['salario_anual']=='$120K +','alta renda',
np.where(credito1['salario_anual'].isin(['$40K - $60K','$80K - $120K','$60K - $80K']),'media renda',None))) 
credito1['categoria_cliente'].value_counts(1) 
plt.hist(credito1['categoria_cliente']) 
plt.ylabel('número clientes') plt.show() 
``` 
![Figure_2](https://github.com/GabrielTavaresGoldman/Analise-Dados-Credito/assets/149710830/86a70b9c-8fe6-4368-8a23-e89034e4d0c6) 
A distribuição de renda anual dos clientes ficou: 52,4% Média Renda, 39,5% Baixa Renda e 8,1% Alta Renda. 
### Default 
Após a análise das rendas, decidi investigar a proporção de clientes que se encontravam em situação de inadimplência e o resultado foi de 15,7%, mesmo que essa proporção possa parecer relativamente pequena em relação ao total, o impacto financeiro da inadimplência ainda pode ser substancial, especialmente se os clientes em situação de inadimplência representarem uma parcela significativa da receita da empresa. 
```
credito1['default'].value_counts(1) 
``` 
### Limite de Crédito 
Por fim, conduzi uma análise dos limites de crédito com o objetivo de compreender sua distribuição e comportamento. 
```
sns.boxplot(credito1['limite_credito'])
plt.show() credito1['limite_credito'].describe()
```
 ![Figure_3](https://github.com/GabrielTavaresGoldman/Analise-Dados-Credito/assets/149710830/435bfb0f-e789-430a-85d6-4dab0f4dbdce)
A média do limite de crédito é de aproximadamente $8500, o que representa um valor considerável. No entanto, devido à presença de outliers (valores extremos), essa medida pode ser distorcida. Por esse motivo, uma métrica mais apropriada para análise é a mediana, que representa o valor do limite de crédito que 50% dos clientes possuem. A mediana é de $4287, o que sugere um valor mais comum entre os clientes. A presença de outliers, como observado nos limites de crédito de até $34516, contribui para elevar a média, o desvio padrão, é de $9126, um valor que é maior até mesmo que a média, indicando uma grande disparidade nos dados. 
## 5.2 Relação entre Variáveis
Primeiramente, criei uma coluna e atribuí valores numéricos às faixas de salários anuais, começando de '1' para a faixa 'menos que $40K', até '5' para a faixa '$120K +'. Essa transformação foi feita com o intuito de poder analisar e comparar relações numéricas.
``` credito1['salario_anual_num'] = np.where(credito1['salario_anual']=='menos que $40K', '1', np.where(credito1['salario_anual']=='$40K - $60K','2',
np.where(credito1['salario_anual']=='$60K - $80K','3',np.where(credito1['salario_anual']=='$80K - $120K','4',np.where(credito1['salario_anual']=='$120K +','5',None)))))
credito1['salario_anual_num'] = pd.to_numeric(credito1['salario_anual_num'])
credito1['salario_anual_num'].value_counts()
``` 
### Relação entre Salário Anual e Limite de Crédito 
Analisei a correlação entre as duas variáveis para verificar a existência de alguma tendência e o resultado foi de 0,60, indicando uma correlação positiva moderada entre o salário anual e o limite de crédito, revelando uma certa tendência de que, clientes com salários anuais mais altos também tenham limites de crédito mais altos.
```
correl_salar_limit = credito1['salario_anual_num'].corr(credito1['limite_credito'])
print('A correlação entre Salário Anual e Limite de Credito', correl_salar_limit)
ordem_salario = ['menos que $40K','$40K - $60K','$60K - $80K','$80K - $120K','$120K +']
credito1['salario_anual'] = pd.Categorical(credito1['salario_anual'], categories=ordem_salario, ordered=True) 
sns.boxplot(x='salario_anual', y='limite_credito', data=credito1) 
plt.show() 
```
![Figure_4](https://github.com/GabrielTavaresGoldman/Analise-Dados-Credito/assets/149710830/ce5cfb48-5c69-43cc-8b7f-22f305953862) 
Quando observamos o gráfico, vemos essa tendência, onde: Faixa Salarial "Menos de $40K" (1): A maior parte dos dados está concentrada em um limite de crédito entre $2000 e $4300. Isso significa que a maioria dos clientes com salários nessa faixa tem limites de crédito nessa faixa de valores. Faixa Salarial "$40K - $60K" (2): A maior parte dos dados está concentrada em um limite de crédito entre $2400 e $6500. Faixa Salarial "$60K - $80K" (3): A maior parte dos dados está concentrada em um limite de crédito entre $3600 e $15000. Faixa Salarial "$80K - $120K" (4): A maior parte dos dados está concentrada em um limite de crédito entre $5700 e $25300. Faixa Salarial "Mais de $120K" (5): A maior parte dos dados está concentrada em um limite de crédito entre $7900 e $34500. No entanto correlação não implica em causalidade, ou seja, não podemos afirmar com certeza que um salário anual mais alto causa um limite de crédito mais alto, pode haver outros fatores que influenciam essa relação, como a idade do cliente, seu histórico de crédito, interações no último ano e o tipo de cartão de crédito que ele possui. 
### Relação entre Idade e Limite de Crédito 
Após a análise entre salário anual e limite de crédito decidi realizar uma outra análise para entender qual ou quais são as variáveis que possuem relação com o limite de crédito, nesse caso fiz a correlação entre idade e limite de crédito e o resultado foi de 0.02 indicando uma correlação praticamente inexistente, a idade de um cliente não parece estar relacionada ao limite de crédito. 
``` 
correl_idade_limit = credito1['idade'].corr(credito1['limite_credito']) 
print('A correlação entre Idade e Limite de Credito', correl_idade_limit) 
``` 
### Relação entre Interações e Limite de Crédito 
Analisei então a relação entre as interações no último ano com o limite de crédito e o resultado foi de 0,01 indicando uma correlação praticamente inexistente, ou seja, a quantidade de interações de um cliente não está relacionada com o seu limite de crédito. 
```
correl_inter_limit = credito1['iteracoes_12m'].corr(credito1['limite_credito']) 
print('A correlação entre Interações no último ano e Limite de Credito', correl_inter_limit) 
``` 
### Relação entre Tipo de Cartão e Limite de Crédito 
Por fim, analisei a relação entre o tipo de cartão e o limite de crédito, e para isso, foi necessário transformar essa variável categórica em numérica, começando de '1' para o cartão 'Blue', até '5' para o cartão 'Platinum', que é uma relação moderada positiva, e pode indicar uma tendência de aumento no limite de crédito à medida que o tipo do cartão aumenta, no gráfico não fica tão evidente, porém, vemos que a distribuição do limite de crédito do cartão Platinum é maior quando comparada as outras.
```
credito1['tipo_cartao_num'] = np.where(credito1['tipo_cartao']=='blue', '1', np.where(credito1['tipo_cartao']=='silver', '2', np.where(credito1['tipo_cartao']=='gold', '3', np.where(credito1['tipo_cartao']=='platinum', '4',None)))) 
credito1['tipo_cartao_num'] = pd.to_numeric(credito1['tipo_cartao_num']) 
correl_cartao_limit = credito1['tipo_cartao_num'].corr(credito1['limite_credito']) 
print('A correlação entre o Tipo de Cartão no último ano e Limite de Credito', correl_cartao_limit) 
ordem_cartao = ['blue','silver','gold','platinum'] 
credito1['tipo_cartao'] = pd.Categorical(credito1['tipo_cartao'], categories=ordem_cartao, ordered=True) 
sns.boxplot(x='tipo_cartao', y='limite_credito', data=credito1) 
plt.show() 
``` 
![Figure_5](https://github.com/GabrielTavaresGoldman/Analise-Dados-Credito/assets/149710830/ac02d242-b9f3-40d5-990b-f46bb943b11a) 
### Agora, optei por explorar a relação entre a variável salário anual e outras duas variáveis, meses inativos e quantidade de transações, a fim de identificar possíveis tendências e relações entre elas. 
### Relação entre valor das transações e quantidade de transações 
Uma das questões que eu levantei inicialmente foi: "Clientes que realizam um maior número de transações ao longo do ano tendem a efetuar transações de valores mais elevados?" Isso foi feito para identificar padrões entre os clientes que movimentam quantias financeiras mais expressivas. Pude encontrar uma correlação bem forte, de 0,81 indicando que, clientes que realizam um maior número de transações ao longo do ano têm, em média, transações de valores mais elevados, no gráfico essa relação se mostra evidente. 
``` 
correl_valor_qtdde = credito1['valor_transacoes_12m'].corr(credito1['qtd_transacoes_12m']) 
print('A correlação entre o Valor de Transições Anual e a Quantidade de Transações no último ano', correl_valor_qtdde) 
intervalos = range(0, credito1['qtd_transacoes_12m'].max() + 10, 10) 
rotulos = [f'{inicio} - {inicio+9}' for inicio in intervalos[:-1]] 
credito1['intervalo_transacoes'] = pd.cut(credito1['qtd_transacoes_12m'], bins=intervalos, labels=rotulos) 
interv_valor = credito1.groupby('intervalo_transacoes')['valor_transacoes_12m'].mean() 
interv_valor.plot(kind='bar') 
plt.xticks(rotation=90) 
plt.xlabel('Quantidade de Transações no Último Ano') 
plt.ylabel('Valor Médio de Transações no Último Ano') 
plt.show() 
``` 
![Figure_6](https://github.com/GabrielTavaresGoldman/Analise-Dados-Credito/assets/149710830/6db16dc8-97d7-4dfc-89ee-a8e0d074ccb5) 
### Relação entre Salário Anual e Meses Inativos 
Outra questão que eu havia questionado era se o salário anual de um cliente tinha alguma relação direta com a quantidade de meses inativos, pude analisar que a correlação era praticamente inexistente (-0,01) indicando que o salário anual não está relacionado diretamente com a quantidade de meses inativos dos clientes. Podemos ver no gráfico que as distribuições são extremamente semelhantes. 
```correl_salar_inat = credito1['salario_anual_num'].corr(credito1['meses_inativo_12m']) 
print('A correlação entre Salário Anual e Meses inativos no último ano', correl_salar_inat) 
sns.boxplot(x='salario_anual', y='meses_inativo_12m', data=credito1) 
plt.show() 
``` 
![Figure_7](https://github.com/GabrielTavaresGoldman/Analise-Dados-Credito/assets/149710830/8f231cd8-d1db-4895-ade5-24485c650342) 
### Relação entre Salário Anual e Quantidade de Transações 
Sabendo da relação forte entre valor de transações e quantidade de transações, decidi analisar a relação entre o salário anual dos clientes e a quantidade de transações que eles realizam no ano, a fim de entender, se, clientes que recebem maiores salários realizam mais transações, porém, ao analisar descobri que não havia relação entre essas duas variáveis com um valor de correlação igual a -0,05, podemos observar no gráfico que não existe nenhuma tendência. 
```correl_salar_trans = credito1['salario_anual_num'].corr(credito1['qtd_transacoes_12m']) 
print('A correlação entre Salário Anual e Quantidade de transações no último ano', correl_salar_trans) 
sns.boxplot(x='salario_anual', y='qtd_transacoes_12m', data=credito1) 
plt.show() 
```
![Figure_8](https://github.com/GabrielTavaresGoldman/Analise-Dados-Credito/assets/149710830/07445338-09ae-4714-b17b-757968faf1fc)
### Enfim, decidi realizar a variável "default" para entender as padrões e tendências de inadimplência dos clientes. 
### Relação entre Inadimplência e Salários 
A porcentagem da relação de inadimplência por salário anual revela uma tendência negativa, ou seja, clientes com salários mais baixos tendem a ter uma maior proporção de inadimplência, enquanto clientes com salários mais altos tendem a ter uma menor proporção de inadimplência. 
```
 total_default = credito1['default'].sum() default_por_categoria = credito1.groupby('salario_anual')['default'].sum() 
porcent_default_categ = (default_por_categoria/total_default)*100 
porcent_default_categ 
sns.set_palette('deep') 
plt.pie(porcent_default_categ, labels=porcent_default_categ.index,autopct='%1.1f%%', startangle=90) 
plt.title('Porcentagem de Default por Salário Anual') 
plt.subplots_adjust(top=0.9) 
plt.legend(title='Salário Anual', loc='lower left') 
plt.axis('equal') 
plt.show() 
``` 
![Figure_9](https://github.com/GabrielTavaresGoldman/Analise-Dados-Credito/assets/149710830/5e99f073-1220-463f-a900-9c15a27307ba)Para verificar se essa relação entre salário anual e inadimplência é estatisticamente significativa, realizei um teste qui-quadrado de independência. O p-valor obtido foi aproximadamente 0.0152. Como o valor-p é menor que 0.05, rejeitamos a hipótese nula de independência entre salário anual e inadimplência, isso confirma que há uma associação estatisticamente significativa entre essas duas variáveis. Esses resultados sugerem que o salário anual pode ser um indicador importante na determinação da probabilidade de inadimplência dos clientes podendo ser considerado um dos fatores para avaliar o risco de inadimplência de um cliente. 
``` 
contingencia = pd.crosstab(credito1['salario_anual_num'], credito1['default']) 
contingencia 
from scipy.stats import chi2_contingency 
chi2, p_value, dof, expected = chi2_contingency(contingencia) 
print(p_value) 
```
### Após uma análise abrangente dos dados demográficos e financeiros dos clientes da instituição financeira, podemos tirar várias conclusões significativas que podem orientar as estratégias de negócios e a tomada de decisões. Aqui estão algumas das conclusões principais:

#### Perfil dos Clientes:
A maioria dos clientes tem entre 41 e 52 anos de idade, com uma distribuição equilibrada entre os sexos.
A maioria dos clientes possui algum nível de educação avançado, com destaque para o mestrado como a escolaridade mais comum.
#### Renda e Inadimplência:
Existe uma forte associação entre renda anual e inadimplência, com clientes de salários mais baixos tendendo a apresentar uma proporção maior de inadimplência. Isso sugere que o salário anual pode ser um indicador importante para avaliar o risco de inadimplência.
#### Limite de Crédito:
Observamos uma correlação positiva moderada entre o salário anual dos clientes e seus limites de crédito, indicando que clientes com salários mais altos tendem a ter limites de crédito mais altos.
#### Comportamento Financeiro:
Clientes que realizam um maior número de transações ao longo do ano tendem a efetuar transações de valores mais elevados, indicando uma forte relação entre a quantidade e o valor das transações.
#### Outros Fatores:
Não foi identificada uma relação significativa entre a idade dos clientes e seus limites de crédito ou meses inativos.
O tipo de cartão de crédito também parece influenciar os limites de crédito, com clientes com cartões de categoria mais alta tendo limites mais altos.

#### Com base nessas conclusões, a instituição financeira pode considerar ajustes em suas políticas de concessão de crédito, estratégias de marketing direcionadas a diferentes segmentos de clientes e medidas para mitigar o risco de inadimplência. Além disso, a análise destaca a importância contínua da coleta e análise de dados para entender melhor o comportamento dos clientes e informar decisões comerciais mais sólidas.
