# Sistema A.D.P. | Colônia de Marte

**FIAP · Ciência da Computação · Etapa 3 · Missão Aurora · 2026**

O **Sistema A.D.P. (Análise, Decisão e Previsão)** é um projeto acadêmico desenvolvido em Python para simular o acompanhamento energético de uma colônia em Marte. O programa reúne informações sobre energia disponível, geração solar e eólica, consumo e velocidade do vento. Com esses dados, identifica situações que exigem atenção, aplica regras de economia e estima a geração eólica.

O objetivo da equipe foi juntar, em um único programa, os conteúdos estudados nesta etapa da FIAP. O resultado é uma aplicação de terminal, com menu interativo, funções separadas e uma previsão feita por regressão linear simples. O projeto também inclui um **notebook Jupyter** com o mesmo código e explicações passo a passo.

## Integrantes

| Integrante | Curso |
| --- | --- |
| Henrique Floriano Alcantara | Ciência da Computação |
| Nicolas França | Ciência da Computação |
| Joao Pedro Candido Souza | Ciência da Computação |
| Katia Regina Bispo | Ciência da Computação |
| Yago Souza Araujo | Ciência da Computação |

## O que o programa faz

Ao iniciar o A.D.P., o usuário encontra um menu com cinco opções e o comando para sair. Primeiro, cadastra cinco informações: energia disponível, geração solar, geração eólica, consumo de energia e velocidade do vento. Depois, pode consultar cada análise separadamente. A geração total da colônia é a soma da geração solar com a eólica.

Abaixo, o fluxograma ilustra como o sistema processa essas informações e toma decisões:

### Fluxo de Funcionamento
![Fluxograma do Sistema A.D.P.](assets/flowchart/fluxograma_adp.png)

A opção **Analisar energia** compara geração total e consumo. Se o consumo for maior, aparece um alerta; se a geração superar o consumo, o programa sugere armazenar o excedente; se os valores forem iguais, informa equilíbrio. Os valores são exibidos em barras de texto para facilitar a comparação.

A opção **Decidir** aplica regras em ordem de prioridade. Quando a energia está abaixo de 50 e o consumo passa de 70, entra em modo de economia e desliga os sistemas não essenciais (primeiro o lazer, depois a iluminação), mantendo sempre o suporte à vida e o sistema energético. Se apenas a energia estiver abaixo de 50, recomenda reduzir o consumo. Se a energia estiver normal, mas o consumo superar a geração, avisa que a reserva está sendo gasta. Quando tudo se normaliza, os sistemas desligados pela economia são religados.

A opção **Prever geração eólica** pede uma velocidade de vento, mostra o histórico, a equação da reta, a estimativa de energia e abre o gráfico da regressão com o matplotlib. Já **Mostrar sistemas** apresenta a hierarquia dos subsistemas em forma de árvore, com o estado de cada um.

**Importante:** as ações de economia e religação alteram o estado dos sistemas no dicionário do programa (simulação em memória). Esta versão não controla equipamentos reais.

## Organização dos dados

Os históricos de vento e geração eólica foram registrados em duas listas. Cada posição corresponde à mesma observação nas duas variáveis:

```python
vento_historico = [8.0, 10.0, 12.0]
energia_historico = [20.0, 25.0, 30.0]
```

Os sistemas da colônia são representados por um dicionário, com uma divisão interna para as fontes energéticas:

```python
sistemas = {
    "energetico": {"solar": True, "eolico": True},
    "suporte_vida": True,
    "iluminacao": True,
    "lazer": True,
}
```

Os valores informados durante a execução ficam em um segundo dicionário (chave-valor), e a prioridade entre os sistemas é definida por duas tuplas:

```python
dados = {"energia": 0.0, "solar": 0.0, "eolica": 0.0,
         "geracao": 0.0, "consumo": 0.0, "vento": 0.0}

ESSENCIAIS = ("energetico", "suporte_vida")   # nunca são desligados
ORDEM_CORTE = ("lazer", "iluminacao")         # o primeiro a ser desligado vem antes
```

Essa organização permite consultar os subsistemas, seus estados e a ordem de prioridade. Os dados não são gravados em arquivo: ao encerrar o programa, tudo volta ao estado inicial.

## Regras de análise e decisão

O código separa a comparação entre geração e consumo da decisão baseada na energia disponível. As regras de decisão são avaliadas **na ordem da tabela**, e a primeira que for verdadeira é aplicada. Os limites ficam em constantes no topo do código (`LIMITE_ENERGIA = 50` e `LIMITE_CONSUMO_ALTO = 70`).

| Condição | Resposta |
| --- | --- |
| Consumo maior que geração | Alerta de consumo acima da geração |
| Geração maior que consumo | Sugestão de armazenar o excedente |
| Geração igual ao consumo | Equilíbrio |
| Energia menor que 50 **e** consumo maior que 70 | Modo de economia: desliga lazer e iluminação, mantém suporte à vida |
| Energia menor que 50, sem consumo acima de 70 | Recomendação de reduzir o consumo |
| Energia igual ou maior que 50 e consumo maior que a geração | Atenção: a reserva está sendo gasta, reduzir o consumo |
| Energia igual ou maior que 50 e consumo até o valor da geração | Operação normal; religa os sistemas desligados pela economia |

As três primeiras linhas pertencem à análise (opção 2). As quatro últimas pertencem à decisão (opção 3), que agora também considera a geração.

## Previsão com regressão linear

Para estimar a geração eólica, a função `calcular_previsao(vento_alvo)` utiliza `numpy.polyfit()` com grau 1. Ela ajusta uma reta aos pares de dados do histórico:

```python
a, b = np.polyfit(vento_historico, energia_historico, 1)
previsao = max(0.0, a * vento_alvo + b)
```

Com os dados iniciais do projeto, a relação obtida é **energia prevista = 2,5 × velocidade do vento**. Assim, para um vento de **11 km/h**, o programa apresenta **27,50 unidades de energia**. A previsão nunca fica abaixo de zero, e a função avisa quando não há dados suficientes (menos de dois pontos ou ventos todos iguais).

A cada cadastro, o vento e a geração eólica informados entram no histórico, e a reta é recalculada. Antes de incluir o novo ponto, o programa compara o que o modelo previa com o valor observado (verificação do modelo), para que a previsão nunca use o próprio dado que está sendo testado. Por isso, depois de novos cadastros, a previsão para 11 km/h pode deixar de ser exatamente 27,50.

Trata-se de uma demonstração didática de regressão linear. Os três registros iniciais são exemplos definidos para o exercício, e o programa não especifica uma unidade física de energia nem representa um modelo validado de geração eólica em Marte.

## Como executar

É necessário ter **Python 3** e as bibliotecas **NumPy** e **matplotlib** instaladas. No terminal, dentro da pasta do projeto, execute:

```bash
python -m pip install numpy matplotlib
```

Em seguida, rode o arquivo principal. Por exemplo, caso ele esteja salvo como `main.py`:

```bash
python main.py
```

Se o seu ambiente usa `python3`, substitua `python` por `python3` nos comandos. O nome `main.py` é apenas um exemplo; utilize o nome real do arquivo enviado ao repositório.

Para acompanhar o código com explicações, abra o notebook `Sistema_ADP_Notebook.ipynb` no Jupyter, VS Code ou Google Colab e execute as células em ordem. O notebook traz demonstrações prontas e, na última seção, o mesmo menu interativo do terminal.

## Exemplo de entrada e saída

O exemplo abaixo mostra um cenário de energia crítica. Após escolher **1 — Cadastrar dados**, informe:

```text
Energia disponível: 40
Geração solar: 10
Geração eólica: 20
Consumo atual: 80
Velocidade do vento (km/h): 11
```

O programa confirma o cadastro e compara o que o modelo previa com o valor observado:

```text
[OK] Dados cadastrados e histórico atualizado com sucesso!

Verificação do modelo (antes deste registro):
Eólica prevista                   27.5
Eólica observada                  20.0
Diferença                          7.5
```

Ao selecionar **2 — Analisar energia** (geração total = 30), o programa responde:

```text
[ALERTA] Consumo maior que geração!
```

Na opção **3 — Decidir**, a saída é:

```text
[ALERTA] Modo de economia ativado!
         Sistemas desligados: lazer, iluminação
         Suporte à vida mantido ligado.
```

Na opção **4 — Prever geração eólica**, ao informar um vento de 11 km/h com o histórico inicial, a previsão é de **27,50 unidades** e o gráfico é aberto. A opção **5 — Mostrar sistemas** exibe a árvore de subsistemas (com iluminação e lazer agora desligados), e a opção **0** encerra o programa.

Para o excedente de energia, com solar = 50, eólica = 30 e consumo = 30, a análise responde `[SUGESTÃO] Armazenar energia excedente!`.

Esses cenários foram executados no próprio programa, e as saídas completas estão na seção seguinte. O programa também trata entradas não numéricas com `try/except`, bloqueia valores negativos e exige o cadastro antes de permitir a análise e a decisão.

## Prints de execução

As imagens abaixo mostram o programa em funcionamento, na ordem de uso. Todas ficam na pasta `prints/` do repositório.

### 1. Menu inicial e estado dos sistemas
![Menu inicial e estado dos sistemas](assets/screenshots/01_menu_sistemas_inicial.png)

### 2. Cadastro de dados
![Cadastro de dados](assets/screenshots/02_cadastro_dados.png)

### 3. Análise de energia: consumo maior que geração
![Análise com alerta](assets/screenshots/03_analise_alerta.png)

### 4. Decisão: modo de economia
![Decisão com modo de economia](assets/screenshots/04_decisao_economia.png)

### 5. Sistemas após o modo de economia
![Sistemas após a economia](assets/screenshots/05_sistemas_apos_economia.png)

### 6. Novo cadastro e verificação do modelo
![Cadastro com verificação do modelo](assets/screenshots/06_cadastro_verificacao_modelo.png)

### 7. Decisão: operação normal e sistemas religados
![Decisão com sistemas religados](assets/screenshots/07_decisao_religa_sistemas.png)

### 8. Análise de energia: excedente
![Análise com excedente](assets/screenshots/08_analise_excedente.png)

### 9. Decisão: consumo maior que a geração
![Decisão com déficit](assets/screenshots/09_decisao_deficit.png)

### 10. Previsão com regressão linear
![Previsão com regressão linear](assets/screenshots/10_previsao_regressao.png)

### 11. Gráfico da regressão (matplotlib)
![Gráfico da regressão linear](assets/screenshots/11_grafico_regressao.png)

### 12. Validações de entrada
![Validações de entrada](assets/screenshots/12_validacoes_entrada.png)

Nesta última imagem, o menu foi omitido para caber todas as validações: opções bloqueadas antes do cadastro, opção inválida, texto no lugar de número, valor negativo no cadastro e vento negativo na previsão.

## Estrutura do programa

O código é organizado em blocos, e cada função tem uma única responsabilidade:

- **Lógica** (calcula e retorna, sem imprimir): `analisar_energia()` compara geração e consumo; `decidir()` aplica as regras por prioridade e altera o estado dos sistemas; `calcular_previsao()` ajusta a reta e estima a energia eólica.
- **Exibição** (só mostra informações): `mostrar_menu()`, `mostrar_dados()`, `mostrar_analise()`, `mostrar_decisao()`, `mostrar_previsao()`, `abrir_grafico()`, `mostrar_sistemas()` e `imprimir_arvore()`, além de funções de formatação (`titulo()`, `secao()`, `linha()`, `barra()`).
- **Entrada de dados**: `ler_float()` e `cadastrar_dados()` leem e validam os valores.
- **Controle**: `main()` organiza o menu e controla o fluxo de execução.

A implementação usa listas, dicionários (inclusive aninhados), tuplas, funções, recursão, `if/elif/else`, o operador lógico `and`, repetição com `while` e tratamento de erros de conversão. As bibliotecas externas são o **NumPy** (regressão) e o **matplotlib** (gráfico).

## Resultados e limitações

O A.D.P. reúne três tarefas que antes poderiam ser tratadas separadamente: acompanhar dados, aplicar regras e fazer previsões. Com isso, a equipe conseguiu demonstrar, em um mesmo programa, conceitos de programação estruturada, organização de informações, lógica com prioridades e modelagem matemática.

O sistema ainda é um protótipo acadêmico. Não recebe leituras de sensores, não mantém histórico entre execuções, não calcula reservas acumuladas e não aciona sistemas físicos: o desligamento e a religação acontecem apenas no estado simulado em memória. Os limites de 50 e 70 são fixos, e o modelo de previsão usa poucos pontos. Como evolução, seria possível salvar as medições em arquivo, permitir a configuração dos limites, ampliar o conjunto de dados da regressão e calcular a reserva de energia ao longo do tempo.

## Repositório oficial

O código e a documentação deste projeto estão disponíveis no GitHub: [Sistema A.D.P. — Colônia de Marte](https://github.com/YagoSDev/Sistema_ADP_Colonia_Marte).

## Relatório técnico

A explicação acadêmica detalhada do projeto está no documento **Relatório Técnico — Etapa 3 — Sistema A.D.P.**. O PDF apresenta a organização dos dados, as regras, a previsão, exemplos e as limitações da implementação.


[Consultar relatório técnico (PDF)](./Relatorio_FIAP_Etapa3_Sistema_ADP.pdf)

## Materiais de referência

O trabalho foi desenvolvido com base nos oito capítulos da Etapa 3 da missão Aurora, disponibilizados pela FIAP em 2026. Entre os conteúdos aplicados diretamente estão lógica booleana, programação estruturada, estruturas de dados, vetores e NumPy, regressão linear simples e fundamentos de energia solar e eólica. Os demais capítulos ajudam a contextualizar a organização computacional da colônia.

O relatório técnico traz a relação dos oito capítulos e os títulos completos dos materiais utilizados.

---

**Projeto acadêmico em equipe · FIAP · Ciência da Computação · Etapa 3 · 2026**
