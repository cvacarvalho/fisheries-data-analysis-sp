# Análise do Desembarque Pesqueiro no Estado de São Paulo (2023 - 2026)

Este projeto apresenta uma análise exploratória de dados sobre o desembarque da pesca comercial no estado de São Paulo, utilizando dados reais do IP/APTA/SAA/SP.  

***Fonte: Estatística Pesqueira Marinha e Estuarina do Estado de São Paulo. Consulta On-line. Programa de Monitoramento da Atividade Pesqueira Marinha e Estuarina do Estado de São Paulo. Instituto de Pesca (IP), Agência Paulista de Tecnologia dos Agronegócios (APTA), Secretaria de Agricultura e Abastecimento do Estado de São Paulo (SAA/SP). Disponível em: http://www.propesq.pesca.sp.gov.br/. Acesso em:28-06-2026. 

O objetivo é mapear as principais espécies capturadas, compreender as tendências volumétricas e financeiras ano a ano, e identificar os principais polos geográficos de desembarque no estado.

Esse projeto faz parte do meu portfólio de Análise de Dados, combinando Oceanologia e Aquicultura com técnicas de modelagem e consulta em SQL.

## Tecnologias e Ferramentas Utilizadas
* **Tratamento de Dados:** Excel / Power Query (limpeza, padronização de cabeçalhos e tratamento de tipos de dados).
* **Banco de Dados:** PostgreSQL (instanciado localmente).
* **SGBD / Interface:** DBeaver (criação de tabelas, importação de arquivos CSV e execução de queries).

## Estrutura do Banco de Dados
A tabela `desembarque_pesca` foi modelada com a seguinte estrutura no PostgreSQL:

```sql
CREATE TABLE desembarque_pesca (
    ano INT,
    mes VARCHAR(20),
    municipio VARCHAR(150),
    aparelho_pesca VARCHAR(150), 
    nivel_taxonomico VARCHAR(100),
    pescado VARCHAR(150),
    tipo_pesca VARCHAR(100), 
    kg_periodo NUMERIC,
    valor_estimado_periodo NUMERIC
);
```
<img width="453" height="747" alt="fig2" src="https://github.com/user-attachments/assets/f330e15d-f2de-4d50-9cbe-3f2b1b23ef07" />

### 1. As 5 Espécies Mais Desembarcadas (Volume Acumulado)
**Principais pescados no estado:**
```
```sql
SELECT
    pescado, 
    SUM(kg_periodo) AS total_kg
 FROM desembarque_pesca dp 
 GROUP BY dp.pescado 
 ORDER BY total_kg DESC
LIMIT 5;

```
A corvina lidera o ranking 4,84 M kg seguida de perto pelo camarão_sete_barbas 4,47 M kg. Isso demonstra forte dependência do estado em recursos demersais e de arrasto de fundo. Em seguida vêm a pescada-foguete (2,08 M kg), manjuba-de-iguape (1,61 M kg) e sardinha-verdadeira (1,41 M kg). 


### 2. Análise de Tendência Temporal
**a) Comportamento do volume e faturamento do estado entre 2023 e 2025**
````
```sql
SELECT
      ano,
      SUM(kg_periodo) AS volume_total_kg,
      SUM (valor_estimado_periodo) AS faturamento_total_reais
  FROM desembarque_pesca dp 
  GROUP BY ano 
  ORDER BY ano ASC 
```
````

Analisando 2023, 2024, 2025 e o que foi coletado no ano corrente (2026), observa-se um pico em 2024, o volume saltou de 5,3 milhões de kg em 2023 para quase 5,9 milhões de kg em 2024, acompanhado de um faturamento de 45 milhões de reais. Em 2025 houve um recuo, voltando para a casa dos 5,5 milhões de kg, com o faturamento mais baixo dos três anos completos (43,7 milhoes de reais). Os dados de 2026, até março, registram 706 mil kg de pescado. Os números demonstram que a atividade não é linear. 

**b) Evolução dos 5 principais pescados ano a ano**
```sql
SELECT
      ano,
      pescado,
      SUM(kg_periodo) AS total_kg_ano
FROM desembarque_pesca dp
WHERE pescado IN ('Corvina' , 'Camarão-sete-barbas', 'Pescada-foguete', 'Manjuba-de-iguape', 'Sardinha-verdadeira')
GROUP BY ano, pescado
ORDER BY pescado, ano ASC;
```
Dados organizados por peixe,  mostrando a evolução ao longo dos anos (2023, 2024, 2025 e 2026). 
*Sardinha-verdadeira: caiu entre 2023 e 2024, mas mais que dobrou em 2025 - comportamento típico de pequenos pelágicos,  que pode indicar sucesso de recrutamento ou reflexo de uma safra espetacular pós defeso. 
*Camarão-sete-barbas, pescada-foguete e manjuba-iguape: tendência de queda constante de 2023 a 2025. Como o camarão e a pescada costumam compartilhar o mesmo ambiente (fundo arrastável), essa queda conjunta pode indicar pressão da pesca ou algum fator ambiental. 
*Corvina: captura em alta constante de  2023 a 2025, confirmando-se como o principal recurso desembarcado do estado. 

### 3. Geografia do Desembarque (Polos Regionais)
**a) Concentração geográfica da produção**
```sql
SELECT
     municipio,
     SUM (kg_periodo) AS total_kg
FROM desembarque_pesca dp
GROUP BY municipio
ORDER BY total_kg DESC
LIMIT 5;
```

   <img width="388" height="461" alt="fig3" src="https://github.com/user-attachments/assets/9b7e16d0-45ae-43d7-903d-063722fc205b" />


Santos/Guarujá 9,01 M kg; Cananéia 3,11 M kg; Iguape 1,85 M kg; Ubatuba 1,25 M kg; São Sebastião 0,83 M kg.


**b) Qual espécie domina qual região?**
```sql
SELECT
     municipio,
     pescado,
     SUM(kg_periodo) AS total_kg
FROM desembarque_pesca
WHERE municipio IN ('Santos/Guarujá', 'Cananéia')
GROUP BY municipio, pescado
ORDER BY municipio, total_kg DESC;
```
*Santos/Guarujá se consolida com corvina no topo,  camarão-sete-barbas e a sardinha-verdadeira. 
*Cananéia mostra a força do litoral sul com a corvina e a pescada-foguete dominando os maiores volumes, seguida pela tainha. 

### 4. Principais Insights

* **Hegemonia Regional Diferenciada:** O eixo Santos/Guarujá funciona como o hub Industrial do estado, isolado no topo do ranking com 9,0M kg, ancorado na pesca de escala de corvina, camarão-sete-barbas e sardinha-verdadeira. Já o Litoral Sul (Cananéia + Iguape) se consolida como a potência da pesca artesanal/estuarina, somando quase 5M kg com forte expressão da tainha no inverno.
* **A Rainha do Desembarque:** a corvina provou ser o recurso mais estratégico e resiliente de São Paulo. Além de liderar o volume acumulado total, apresentou crescimento estável de 2023 a 2025 e ocupa o topo do ranking nas duas regiões.
* **Anomalia de Comportamento (Efeito Sanfona):** a sardinha-verdadeira registou quedas consecutivas em 2023 e 2024, mas mais do que dobrou o seu volume em 2025. Este comportamento sugere forte dependência de fatores sazonais, recrutamento biológico ou safras excecionais pós-defeso.
* **Alerta de Tendência:** as espécies *camarão-sete-barbas, pescada-foguete e manjuba-de-iguape enfrentam uma tendência de queda constante na produção de 2023 a 2025, sinalizando uma necessidade de atenção para à gestão dos estoques demersais.

### 5. Conclusão
Este projeto demonstra:
* Domínio de ETL e tratamento de tipos de dados na origem (Excel).
* Criação, modelagem e estruturação de tabelas relacionais no PostgreSQL.
* Importação de dados brutos e mapeamento de colunas via DBeaver.
* Execução de consultas SQL exploratórias e cruzamentos  (`SUM`, `GROUP BY`, `WHERE IN`, `ORDER BY`).
* Interpretação analítica cruzando a dinâmica biológica da pesca com a realidade socioeconómica do setor em São Paulo.
