# Eleições Brasil

Esse repositório de código contém programas que baixam dados do Tribunal
Superior Eleitoral, normalizam, agrupam e limpam. Atualmente as bases de dados
disponíveis são:

- Candidaturas (1996 a 2020)
- Bens declarados (2006 a 2020)
- Votação por zona eleitoral (1996 a 2020)
- Prestação de contas (2002 a 2020)

## Metodologia

A fonte primária é o [Repositório de Dados Eleitorais do
TSE](http://www.tse.jus.br/eleicoes/estatisticas/repositorio-de-dados-eleitorais-1).
O processo é o seguinte:

- Baixar arquivos ZIP
- Para cada ZIP, extrair (em memória) os arquivos internos
- Para cada arquivo interno, processá-lo (identificar cabeçalho, limpar)
- Juntar todos os resultados em um arquivo


## Normalização

Algumas etapas de normalização são necessárias para facilitar análises e
conversões dos dados, como:

- Retirar todos os acentos: alguns nomes aparecem com acentos em um ano e sem
  em outros, dificultando muito os agrupamentos;
- Retirar strings inúteis: valores como `#NULO#`, `#NULO`, `#NE` e `#NE#` são
  retirados, deixando as células em branco;
- Normalização dos códigos de cargo: os códigos de cargo variam para alguns
  anos, tornando difícil o agrupamento entre anos e, para facilitar as
  análises, normalizamos todos os anos;
- Renomear colunas: nem todas as colunas possuem nomes intuitivos e foram
  nomeadas (exemplo: `COD_SIT_TOT_TURNO` foi renomeado para
  `codigo_totalizacao_turno`). Para saber mais detalhes sobre as colunas que
  foram renomeadas, olhe os arquivos no diretório `headers/` (caso você altere
  algum desses arquivos, gere novamente os cabeçalhos finais com
  `python tse.py headers`).

> Nota: nem todos os códigos/descrição foram normalizados (alguns apresentam
> inconsistências ainda não resolvidas e serão feitos em breve).


### Privacidade

Para garantir a privacidade e evitar SPAM, o script limpa algumas colunas com
informações sensíveis. Essa é a forma padrão de funcionamento para não
facilitar a exposição desses dados. Os dados censurados são:

- Na tabela `candidatura`:
  - A coluna `email` terá seu conteúdo totalmente limpo
  - A coluna `cpf` terá os 3 primerios dígitos trocados por `*`

Caso queira rodar o script sem o modo censura, altere o script `run.sh` e
adicione a opção `--no-censorship` para o script `tse.py`.


## Instalando

Os programas requerem Python 3.7+. Instale as dependências executando:

```bash
pip install -r requirements.txt
```


## Rodando

O script `tse.py` baixa, trata e extrai os dados. Basta rodá-lo, passando que
tipo de dado quer baixar/tratar/extrair:

```bash
python tse.py candidatura
python tse.py bem-declarado
python tse.py votacao-zona
python tse.py receita
python tse.py despesa
```

Os dados ficarão disponíveis em:

- `data/download/`: arquivos originais baixados, por ano
- `data/output/`: arquivos extraídos (agrupados por tipo)

Caso queira converter os arquivos `.csv.gz` gerados em um banco de dados
SQLite (facilita as análises), execute:

```bash
./csv2sqlite.sh
```

Esse script irá rodar o comando `rows csv-to-sqlite` em todos os arquivos
gerados em `data/output` e criará o arquivo `data/eleicoes-brasil.sqlite`.

### Opções

As opções listadas abaixo podem ser utilizadas em conjunto.

#### Anos

Você pode especificar para quais anos deseja a extração (separados por
vírgulas), como em:

```bash
python tse.py candidatura --years=2014,2018
```

#### Alterar fonte dos dados

Como os servidores do TSE podem estar lentos em épocas de eleição, você pode
baixar os dados de algum servidor mais rápido que hospede uma cópia deles
(chamamos de *mirror*). No Brasil.IO mantemos um *mirror* de alguns arquivos do
Repositório de Dados Eleitorais do TSE, **porém** note que ainda não temos uma
política de atualização frequente desses dados e, por isso, eles podem estar
desatualizados (estamos trabalhando para que sejam atualizados com mais
frequência e que a data de cópia dos dados fique explícita em uma página).

Para que o script colete os daos usando nosso *mirror*, execute-o com a opção
`--use-mirror`:

```bash
python tse.py candidatura --use-mirror
```

Você pode especificar também a URL base do *mirror*, que por padrão é do
[Brasil.IO](https://brasil.io/):

```bash
python tse.py candidatura --use-mirror --mirror-url=https://data.brasil.io/mirror/eleicoes-brasil/
```


#### Apenas baixar

Caso queira apenas baixar os arquivos, utilize a opção `--download-only`.

#### Forçar download

Por padrão, caso os arquivos necessários para uma extração já existam em
`data/download/`, eles não serão baixados novamente. Você pode utilizar a opção
`--force-redownload` para que eles sejam deletados e baixados novamente.

#### Arquivo de saída

Você pode especificar o arquivo de saída (que será sempre um CSV, mas pode
estar compactado):

```bash
python tse.py candidatura --output=candidatura.csv.gz
```

#### Observações

Em alguns casos o TSE libera arquivos compactados no formato RAR (mesmo com a
extensão ".zip"). Para extrair os dados desses arquivos você precisa instalar
em seu sistema o bsdtar ou unrar (em sistemas Debian e derivados):
`apt install libarchive-tools` ou `apt install unrar` - o último não é software
livre).


## Desenvolvendo/contribuindo

Instale as dependências de desenvolvimento:

```bash
pip install -r dev-requirements.txt
```

Rode os testes:

```bash
pytest tests.py
```

Ao alterar os arquivos, rode o comando `black .` para normalizá-los com relação
à [PEP-0008](https://www.python.org/dev/peps/pep-0008/).
