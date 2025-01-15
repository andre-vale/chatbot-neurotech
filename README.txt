----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Chatbot para Análise de Dados
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

O arquivo detalha a implementação um chatbot capaz de interpretar perguntas em linguagem natural, traduzi-las para consultas SQL e fornecer insights significativos a partir de um banco de dados (PostgreSQL) hospedado na nuvem. 
A solução é desenvolvida com Flask, SQLAlchemy e Hugging Face Transformers, utilizando o modelo BERTimbau.

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Instruções para Execução
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Execução Local

1. Baixar o Dataset disponivel em:

https://github.com/Neurolake/challenge-data-scientist/blob/main/datasets/credit_01/train.gz.


2. Configure um Ambiente Virtual (AWS):

- No RDS, Crie um Banco de Dados 'chatbotbd'
- Certificar que ele tenha acesso público.
- Fazer conexao via algum client (Ex.: DBeaver)

3. Executar o arquivo "Desafio_Chatbot_1_Pre_Process.ipynb":

- Carregue o arquivo .csv
- Selecione as colunas importantes: 'REF_DATE', 'TARGET', 'VAR2', 'IDADE', 'VAR4', 'VAR5', 'VAR8'
- Fazer tratamento dos valores da variável VAR4(Flag de óbito): Substituir os valores 'S' por 1 e os valores NaN por 0
- Fazer tratamento dos valores da variável VAR8(Classe social): Substituir os valores 'A' por 1, 'B' por 2, 'C' por 3, 'D' por 4 e 'E' por 5.
- Tratando valores ausentes:
    - Uso de Gaussiana para a Coluna IDADE. As colunas "REF_DATE" e "TARGET" não possuem valores ausentes. 
    - A Coluna IDADE possui uma distruibuição parcialmente normal, por isso o uso da Gaussiana para imputar valores.
    - As colunas "Sexo" "VAR5: Unidade Federativa (UF)" apesar de possuirem valores ausentes, possuem dados categóricos, logo, a Gaussiana n é aplicável. 
    - As colunas "VAR4(Flag de óbito)" e "VAR8 (Classe social)" possuem uma alta proporção de valores ausentes. Gaussiana não é aplicável. Para esses casos, foi escolhido o modelo de Regressão Linear para imputar valores.
- Salvar o dataset processado como "dataset_imputado.csv"

4. Executar o arquivo "Desafio_Chatbot_2_Conexao_BD.ipynb":

- Carregue o arquivo dataset_imputado.csv
- instale o boto3: pip install boto3 psycopg2 pandas
- Conexão com o Banco de Dados
    - "host": "chatbotdb.cz0yqgoeikkw.us-east-2.rds.amazonaws.com",
    - "database": "postgres",
    - "user": "postgres",
    - "password": "sua-senha",
    - "port": 5432
- Criar a tabela "chatbot_data2"
- Carregar os Dados para o Banco
- Testando a Conexão e Consultas    

5. Executar o arquivo que gera o chatbot "app.py":

- Instalando o flask: pip install flask transformers psycopg2 pandas
- Instalando o SQLAlchemy: pip install sqlalchemy
- Importa as bibliotecas:
    - from flask import Flask, request, jsonify, render_template_string
    - from transformers import AutoTokenizer, AutoModelForMaskedLM
    - from sqlalchemy import create_engine
    - import psycopg2
    - import pandas as pd
- Fazer conexão com o Banco de Dados
- Carregar o modelo BERTimbau
- Processar e Tokenizar a consulta do usuário (escalável para mais perguntas, se necessário)
- Configurando a interface com Flask
    - Página inicial para o chatbot (interface amigável para realizar as perguntas)
    - Endpoint para executar as consultas
    - Endpoint para retornar resultados processados
    
6. Execute o Servidor:

- python app.py
- O servidor estará disponível em http://127.0.0.1:5000.    
    
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------    
Escolhas de Design
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Arquitetura

1. Flask como Framework:
- Escolhido pela simplicidade na criação de APIs e sua compatibilidade com bibliotecas Python.

2. SQLAlchemy:
- Utilizado para interagir com o banco de dados PostgreSQL de forma segura e eficiente.

3. BERTimbau:
- Selecionado por ser um modelo robusto e treinado para o português, garantindo melhor compreensão das consultas.


----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Desafios Enfrentados
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

1. Base de dados com algumas colunas com alta proporção de valores ausentes fazendo com que não fosse possível verificar a distribuição normal destes, impedindo o uso de Gaussiana para tratamento. 
2. Base de dados com algumas colunas com dados categórigos possuindo alta proporção de valores ausentes. Gaussiana também não pode ser usada. Para esses casos, o preenchimento com a Moda foi utilizado.
3. Tratamento com Regressão Logística exigiu transformação de dados CHAR, como a Classe Social, para Inteiros para que o modelo pudesse ser trabalhado. 
4. Formato das datas precisou ser convertendo para o formato YYYY-MM-DD.
5. A primeira tentativa foi usando pandas para manipulação dos dados com o Banco de Dados. Não deu certo. o método read_sql deu muito erro. tentativa 2 usando SQLAlchemy foi mais exitosa.
6. Alguns problemas de execução também foram observados, o Prompt reconhecia as consultas, porém, o objeto enviado ao navegador, não. Foi preciso uma lógica para identificar se o objeto tratado seria um json ou un form.