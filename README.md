# Explorando Relações entre Dados Demográficos e Financeiros
A análise abaixo foi realizada utilizando um conjunto de dados de credito de uma instituição financeira público utilizado nos cursos Carreira de Analista de Dados, Python e SQL da EBAC

O objetivo da análise é compreender os padrões de clientes de uma instituição financeira, utilizando dados demográficos, comportamentais e financeiros. Essa compreensão pode ser crucial para diversas áreas, como marketing, atendimento ao cliente e gestão de riscos.

Perguntas que queremos responder:
- Clientes com faixas salariais maiores tem proprensão ao Defauft menor?
- Qual é a distribuição da escolaridade entre os clientes?
- Como é a distribuição do salário dos clientes?
- Quantos dos nossos clientes estão em situação de inadimplência?
- Como é a distribuição dos limites de créditos dos clientes? E qual a relação desses limites com o salário anual deles?
- O salário dos clientes tem impacto com a inatividade dos mesmos?
- Clientes com salários maiores, tem uma quantidade maior de transações? E possuem maiores limites?
- Clientes com salários menores, tem uma probabilidade de ficar mais meses inativos?
- Clientes mais velhos possuem quantidades maiores de transações no último ano?
# 2. Carregando Bibliotecas
```import pandas as pd 
import matplotlib.pyplot as plt
import seaborn as sns 
import numpy as np
 ```
# 3. Carregando Dataset
```caminho = r"C:\Users\gabriel.goldman\Área de Trabalho\Modelo Crédito\credito.csv"
credito = pd.read_csv(caminho)
```
# 4. Limpeza e Processamento dos Dados
Nessa etapa primeiro identifiquei quais são os tipos dos dados, as variáveis 'limite_credito' e 'valor_transacoes_12m' estavam em um formato de texto, então converti para numérico e como o python entende os valores decimais apenas com ponto e não por vírgula, substitui os mesmos. Depois verifiquei se havia valores nulos (não havia).
```credito.dtypes
credito['limite_credito'] = pd.to_numeric(credito['limite_credito'].str.replace('.', '').str.replace(',', '.'))
credito['valor_transacoes_12m'] = pd.to_numeric(credito['valor_transacoes_12m'].str.replace('.', '').str.replace(',', '.'))
print(credito.isnull().sum())
```
### Entendendo os dados
Além de identificar os dados, preciso entender com quais dados eu irei trabalhar, então separei as variáveis categóricas e printei os valores únicos, percebi que existiam respostas 'na' que provavelmente foram de clientes que não preencheram todos os dados, e por ser uma análise é demográfica, decidi então criar um novo DataFrame sem esses valores (que poderiam distorcer a análise)
```categoricas = ['sexo','escolaridade','estado_civil','salario_anual', 'tipo_cartao']
for coluna in categoricas:
    valores_unicos = credito[coluna].unique()
    print(f'Valores únicos em {coluna}: {valores_unicos}')
    
credito1 = credito.loc[(credito['escolaridade'] != 'na') & (credito['salario_anual'] != 'na') & (credito['estado_civil'] != 'na')]
```
# 5. Análises
## 5.1 Análise de cada variável
### Idade
Analisando as idades dos nossos clientes foi possível concluir que a grande parte dos clientes possuem de 41 à 52 anos de idade, com uma média e mediana de 46 anos.
#### Aqui é importante analisarmos tanto a média como a mediana, pois no gráfico vemos a presença de outliers (dados que são excessões/extremos) em idades mais velhas, oque poderia "puxar" a nossa média para cima.
![Figure_1](https://github.com/GabrielTavaresGoldman/Analise-Dados-Credito/assets/149710830/2f9a1612-62fa-4ff9-b20c-093c6b00c6ce)
### Sexo
Identifiquei que o sexo dos nossos clientes é bem semelhante sendo 52,3% são mulheres e 47,7% são homens.
``` sexo = credito1['sexo'].value_counts()
sexo
```
### Escolaridade 
Nessa análise foi possível observar que a grande parte dos clientes (37%) possuem mestrado, seguido de 23% que fizeram até o ensino médio, 17% não tem uma educação formal, 12% possuem graduação e 11% possuem doutorado.
### Renda
Na renda percebi que as respostas dos clientes foram baseadas em faixas de valores, então decidi realizar uma segmentação dos dados para dividir os clientes em 3 categorias ( Baixa Renda - menos de $40K anual, Média Renda - Entre $40K - $120 anual e Alta Renda - mais de $120K anual)
```credito1['salario_anual'].value_counts()
credito1['categoria_cliente'] = np.where(credito1['salario_anual']=='menos que $40K', 'baixa renda',
                                         np.where(credito1['salario_anual']=='$120K +','alta renda',
                                                  np.where(credito1['salario_anual'].isin(['$40K - $60K','$80K - $120K','$60K - $80K']),'media renda',None)))                                       
credito1['categoria_cliente'].value_counts(1)
plt.hist(credito1['categoria_cliente'])
plt.show()
```
![Figure_2](https://github.com/GabrielTavaresGoldman/Analise-Dados-Credito/assets/149710830/90a469bc-6ce1-4325-afeb-636998c3d3e2)

A distribuição de renda anual dos clientes ficou: 52,4% Média Renda, 39,5% Baixa Renda e 8,1% Alta Renda.
### Default
Após a análise das rendas, decidi investigar a proporção de clientes que se encontravam em situação de inadimplência e o resultado foi de 15,7%, mesmo que essa proporção possa parecer relativamente pequena em relação ao total, o impacto financeiro da inadimplência ainda pode ser substancial, especialmente se os clientes em situação de inadimplência representarem uma parcela significativa da receita da empresa.
```credito1['default'].value_counts(1)
``` 
### Limite de Crédito
Por fim, conduzi uma análise dos limites de crédito com o objetivo de compreender sua distribuição e comportamento.
```sns.boxplot(credito1['limite_credito'])
plt.show()
credito1['limite_credito'].describe()
```
![Figure_3](https://github.com/GabrielTavaresGoldman/Analise-Dados-Credito/assets/149710830/435bfb0f-e789-430a-85d6-4dab0f4dbdce)
A média do limite de crédito é de aproximadamente $8500, o que representa um valor considerável. No entanto, devido à presença de outliers (valores extremos), essa medida pode ser distorcida. Por esse motivo, uma métrica mais apropriada para análise é a mediana, que representa o valor do limite de crédito que 50% dos clientes possuem. A mediana é de $4287, o que sugere um valor mais comum entre os clientes. A presença de outliers, como observado nos limites de crédito de até $34516, contribui para elevar a média, o desvio padrão, é de $9126, um valor que é maior até mesmo que a média, indicando uma grande disparidade nos dados.
## 5.2 Relação entre variáveis
Primeiramente eu criei uma nova coluna e atribuí valores numéricos às faixas de salários anuais, começando de '1' para a faixa 'menos que $40K', até '5' para a faixa '$120K +'. Essa transformação foi feita com o intuito de poder analisar e comparar relações numéricas.
```credito1['salario_anual_num'] =  np.where(credito1['salario_anual']=='menos que $40K', '1', np.where(credito1['salario_anual']=='$40K - $60K','2',
    np.where(credito1['salario_anual']=='$60K - $80K','3',np.where(credito1['salario_anual']=='$80K - $120K','4',np.where(credito1['salario_anual']=='$120K +','5',None)))))
credito1['salario_anual_num'] = pd.to_numeric(credito1['salario_anual_num'])                            
credito1['salario_anual_num'].value_counts()
```
### Relação entre salario anual e limite de crédito
Análisei a correlação entre as duas variáveis para verificar a existência de alguma tendência e o resultado foi de 0,60, indicando uma correlação positiva moderada entre o salário anual e o limite de crédito, hrevelando uma certa tenêndia de que clientes com salários anuais mais altos também tenham limites de crédito mais altos, no gráfico também fica visível essa tendência. No entanto correlação não implica em causalidade, ou seja, não podemos afirmar com certeza que um salário anual mais alto causa um limite de crédito mais alto, pode haver outros fatores que influenciam essa relação, como a idade do cliente, seu histórico de crédito, interações no último ano e o tipo de cartão de crédito que ele possui.
```correl_salar_limit = credito1['salario_anual_num'].corr(credito1['limite_credito'])
print('A correlação entre Salario Anual e Limite de Credito', correl_salar_limit)
sns.boxplot(x='salario_anual_num', y='limite_credito', data=credito1)
plt.show()
```
![Figure_4](https://github.com/GabrielTavaresGoldman/Analise-Dados-Credito/assets/149710830/7f961146-9c6f-436f-8625-be54da6b5cd6)
### Relação entre idade e limite de crédito
Após a análise entre salario anual e limite de crédito decidi realizar uma outra análise para entender qual ou quais são as variáveis que possuem relação com o limite de crédito, nesse caso fiz a correlação entre idade e limite de crédito e o resultado foi de 0.02 indicando uma correlação praticamente inexistente, a idade de um cliente não parece estar relacionado ao limite de crédito.
```correl_idade_limit = credito1['idade'].corr(credito1['limite_credito'])
print('A correlação entre Idade e Limite de Credito', correl_idade_limit)
```
### Relação entre interações e limite de crédito
Analisei então a relação entre as interações no último ano com o limite de crédito e o resultado foi de 0.01 indicando uma correlação praticamente inexistente, ou seja, a quantidade de interações de um cliente não está relacionado com o seu limite de crédito.
```correl_inter_limit = credito1['iteracoes_12m'].corr(credito1['limite_credito'])
print('A correlação entre Interações no último ano e Limite de Credito', correl_inter_limit) 
```
### Relação entre tipo de cartão e limite de credito
Por fim, analisei a relação entre o tipo de cartão e o limite de crédito, e para isso, foi necessário transformar essa variável categórica em numérica, começando de '1' para o cartão 'Blue', até '5' para o cartão 'Platinum', que é uma relação moderada positiva, e pode indicar uma tendência de aumento no limite de crédito à medida que o tipo do cartão aumenta, no gráfico não fica tão evidente, porém vemos que a distribuição do limite de crédito do cartão Platinum é maior.
```credito1['tipo_cartao_num'] =  np.where(credito1['tipo_cartao']=='blue', '1', np.where(credito1['tipo_cartao']=='silver', '2', np.where(credito1['tipo_cartao']=='gold', '3',
    np.where(credito1['tipo_cartao']=='platinum', '4',None))))
credito1['tipo_cartao_num'] = pd.to_numeric(credito1['tipo_cartao_num'])
correl_cartao_limit = credito1['tipo_cartao_num'].corr(credito1['limite_credito'])
print('A correlação entre o Tipo de Cartão no último ano e Limite de Credito', correl_cartao_limit) 
sns.boxplot(x='tipo_cartao_num', y='limite_credito', data=credito1)
plt.show()
```
![Figure_5](https://github.com/GabrielTavaresGoldman/Analise-Dados-Credito/assets/149710830/4bf2edf0-16aa-4019-84ef-ea509b42dbc8)
