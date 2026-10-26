# Segurança viária orientada a dados — Estado de São Paulo

Projeto Aplicado IV do curso de Ciência de Dados (EaD) da Universidade Presbiteriana Mackenzie, 2º semestre de 2026.

Autor: Felipe Carlos Campelo

O projeto usa os microdados do Infosiga SP para estudar os sinistros de trânsito com vítimas fatais no Estado de São Paulo. São duas análises principais:

- **Classificação:** quais características do sinistro, da via, dos veículos e das vítimas estão ligadas a um desfecho fatal.
- **Séries temporais:** previsão do número mensal de óbitos até 2030, comparando com a meta de reduzir as mortes pela metade.

## Estrutura do repositório

```text
.
├── cd_projeto_aplicado_IV_doc.ipynb   # notebook principal (documentação e código)
├── requirements.txt                   # bibliotecas Python usadas
├── .gitignore                         # deixa a base de dados fora do Git
├── figuras/                           # imagens usadas no notebook
│   ├── pipeline.png
│   └── cronograma.png
└── dados_infosiga/                    # base de dados (não vai para o GitHub, ver abaixo)
```

## Base de dados

Os dados vêm do conjunto **Sinistros (Infosiga)**, publicado pelo Detran-SP no Portal de Dados Abertos do Estado de São Paulo:

**<https://dadosabertos.sp.gov.br/dataset/sinistros-infosiga>**

A licença é Creative Commons Attribution 4.0, e os dados são atualizados todo mês, por volta do dia 15. Na mesma página está o dicionário de dados.

Os arquivos CSV somam cerca de 930 MB. Isso passa do limite do GitHub, então eles não estão no repositório e precisam ser baixados à parte.

### Como baixar

1. Acesse o link acima e baixe o arquivo ZIP com os dados.
2. Extraia o conteúdo na pasta `dados_infosiga/`, na raiz do projeto.
3. Confira se a pasta ficou assim:

```text
dados_infosiga/
├── sinistros_2015-2021.csv
├── sinistros_2022-2024.csv
├── sinistros_2025-2026.csv
├── pessoas_2015-2021.csv
├── pessoas_2022-2024.csv
├── pessoas_2025-2026.csv
├── veiculos_2015-2021.csv
├── veiculos_2022-2024.csv
└── veiculos_2025-2026.csv
```

Se os nomes dos arquivos no portal mudarem numa atualização futura, renomeie-os para ficarem iguais aos da lista acima.

### Sobre os arquivos

| Arquivo | Uma linha por | O que tem |
|---|---|---|
| `sinistros_*.csv` | sinistro | data, hora, município, tipo de via, coordenadas, tipo de sinistro |
| `pessoas_*.csv` | pessoa envolvida | sexo, idade, meio de transporte, gravidade da lesão, data do óbito |
| `veiculos_*.csv` | veículo envolvido | tipo, marca/modelo, ano de fabricação, cor |

As tabelas se ligam pela coluna `id_sinistro`. Pessoas e veículos também se ligam pela coluna `id_veiculo`.

Os CSVs usam codificação **latin-1** e separador **`;`**. Para ler com o pandas:

```python
import pandas as pd

sinistros = pd.read_csv("dados_infosiga/sinistros_2025-2026.csv", sep=";", encoding="latin-1")
```

Os registros de óbitos começam em 2015. Os sinistros sem mortes só aparecem a partir de 2019.

Os dados usados no projeto foram baixados em **16/09/2026**. Como o Detran-SP pode corrigir registros antigos, uma versão baixada depois pode dar números um pouco diferentes.

## Como rodar

### Opção 1: Google Colab

1. Abra o notebook pelo link:
   <https://colab.research.google.com/github/fccampelo/transito-saopaulo/blob/main/cd_projeto_aplicado_IV_doc.ipynb>
2. Envie a pasta `dados_infosiga/` para o seu Google Drive e monte o Drive no Colab:

   ```python
   from google.colab import drive
   drive.mount("/content/drive")
   ```

3. Ajuste o caminho da pasta de dados no notebook, se precisar, e execute as células em ordem.

### Opção 2: na sua máquina

Você precisa do Python 3.10 ou mais recente.

```bash
# 1. Clonar o repositório
git clone https://github.com/fccampelo/transito-saopaulo.git
cd transito-saopaulo

# 2. Criar e ativar um ambiente virtual
python -m venv .venv
source .venv/bin/activate        # no Windows: .venv\Scripts\activate

# 3. Instalar as bibliotecas
pip install -r requirements.txt

# 4. Baixar a base de dados para dados_infosiga/ (ver seção acima)

# 5. Abrir o notebook
jupyter notebook cd_projeto_aplicado_IV_doc.ipynb
```

Execute as células de cima para baixo. A primeira leitura dos CSVs demora um pouco por causa do tamanho dos arquivos. Por isso, o notebook salva uma cópia dos dados tratados em formato Parquet, e as execuções seguintes ficam mais rápidas.

## Status

O projeto está em andamento. Na implementação parcial (26/10/2026), foram concluídos uma análise exploratória inicial do recorte de sinistros de 2019–2024, um cache analítico em Parquet e um baseline de regressão logística com teste temporal em 2024. A célula do notebook apresenta as métricas, gráficos, tabelas de qualidade e perfil agregado de vítimas.

O modelo base obteve Average Precision de 0,2291 (prevalência no teste: 0,0294) e ROC-AUC de 0,8710. No limiar de 0,5, a revocação foi 0,8047 e a precisão, 0,0995; o limiar ainda precisa ser ajustado para o objetivo de uso. A análise não é causal e os volumes por região ou modal não são taxas de risco.

Continuam pendentes para a entrega final: série mensal de óbitos e modelos de previsão, integração agregada das tabelas de pessoas e veículos ao classificador, comparação com modelos baseados em árvores, calibração e limiar, interpretação avançada e análise espacial com denominadores adequados. Os arquivos Parquet são caches locais ignorados pelo Git e são recriados pelo notebook quando ausentes.
