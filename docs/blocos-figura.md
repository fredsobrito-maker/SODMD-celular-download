# Blocos de figura do SODMD

O SODMD (celular, computador e extensão, versão 1.5 em diante) desenha figuras a partir de blocos de código
com a linguagem `figura`. O bloco guarda só os dados; o desenho é feito na hora de ler
o plano, sem internet. Se o bloco tiver erro, aparece uma mensagem no lugar da figura
e o resto do plano continua normal.

Formato: um bloco de código cercado por três crases, com a palavra `figura` na primeira
linha, e um objeto JSON dentro. Aspas retas (`"`), sem comentários.

## 1. Plano cartesiano (`plano_cartesiano`)

| Campo | Obrigatório | Valores |
|---|---|---|
| `tipo` | sim | `"plano_cartesiano"` |
| `titulo` | não | texto acima da figura |
| `eixo_x`, `eixo_y` | não | objeto com `nome`, `min`, `max`, `passo` (marcas) e `passo_grade` (grade). Padrão: −10 a 10 |
| `curvas` | não | lista de curvas (abaixo) |
| `pontos` | não | lista de `{"x": 4, "y": 16, "rotulo": "vértice"}` (o rótulo é opcional) |
| `grade` | não | `false` para esconder a grade |
| `legenda` | não | `true` ou `false`. Sem o campo, a legenda aparece quando há 2 ou mais curvas com `nome` |

Cada curva usa **uma** destas três formas:

- `"formula"`: `"8*x - x^2"`. Aceita `+ - * / ^`, parênteses, `x` (ou `t`), `pi`, `e`,
  multiplicação implícita (`2x+1`), prefixo `y =`, e as funções `sen`, `cos`, `tan`, `asin`,
  `acos`, `atan`, `sqrt`, `abs`, `ln`, `log`, `exp`.
- `"reta"`: `[[0, 9], [8, 1]]`, segmento entre dois pontos. Com `"prolongar": true` a reta
  atravessa o gráfico inteiro.
- `"pontos"`: `[[0,0],[1,2.1],[2,3.9]]`. Liga os pontos; `"ligar": false` mostra só os pontos
  e `"marcadores": false` esconde as bolinhas.

Campos opcionais de qualquer curva: `nome` (legenda), `cor` (`azul`, `vermelho`, `verde`,
`dourado`, `ciano`, `preto`), `estilo` (`"tracejado"` ou `"pontilhado"`), `de` e `ate`
(limites em x).

Exemplo:

```figura
{
  "tipo": "plano_cartesiano",
  "titulo": "Movimento uniformemente variado",
  "eixo_x": {"nome": "t (s)", "min": 0, "max": 8, "passo": 1},
  "eixo_y": {"nome": "x (m)", "min": 0, "max": 18, "passo": 2},
  "curvas": [{"formula": "8*x - x^2", "nome": "posição"}],
  "pontos": [{"x": 4, "y": 16, "rotulo": "vértice (4; 16)"}]
}
```

Receitas para temas da Física (testadas):

- **Fator de Lorentz** (relatividade): `{"formula": "1/sqrt(1-x^2)", "de": 0, "ate": 0.99}`, com
  eixo x de 0 a 1 (v/c) e eixo y de 0 a 8. Para marcar a assíntota v = c, acrescente uma
  segunda curva `{"reta": [[1, 0], [1, 8]], "estilo": "tracejado", "nome": "v = c"}` e `"legenda": true`.
- **Decaimento radioativo**: `{"formula": "exp(-0.693*x)"}` com x em meias-vidas (N/N0 cai à metade
  a cada unidade).
- **Intensidade, ondas e MHS**: `sen(2*pi*x)`, `cos(x)`, e assim por diante.

## 2. Diagrama de corpo livre (`diagrama_corpo_livre`)

Diagrama de corpo livre **puntiforme**: o corpo é um ponto e todas as forças saem dele.
O tamanho de cada seta é proporcional ao módulo. O aplicativo soma as forças, projeta as
componentes e marca os ângulos, mas só quando o plano pede (campos opcionais abaixo).

| Campo | Obrigatório | Valores |
|---|---|---|
| `tipo` | sim | `"diagrama_corpo_livre"` |
| `forcas` | sim | lista de 1 a 8 forças (abaixo) |
| `titulo` | não | texto acima da figura |
| `unidade` | não | unidade do módulo, padrão `"N"` |
| `valores` | não | `false` mostra só os nomes das forças, sem os módulos |
| `resultante` | não | `true` desenha a força resultante (seta tracejada grossa) e escreve módulo e ângulo na legenda. Se a soma for nula, a legenda diz que há equilíbrio |
| `componentes` | não | `true` (todas as forças) ou lista de nomes, como `["P"]`: desenha as componentes x e y tracejadas |
| `eixos` | não | `true` desenha os eixos x e y pelo ponto |
| `inclinacao` | não | ângulo em graus que gira os eixos (plano inclinado). Já desenha os eixos. As componentes passam a ser projetadas nos eixos girados |
| `angulos` | não | `true` marca com um arco o ângulo de cada força com o semieixo mais próximo |

Cada força usa **uma** destas duas formas:

- `{"nome": "P", "modulo": 50, "angulo": 270}`: módulo positivo e ângulo em graus, medido a
  partir da horizontal para a direita (→ = 0°), no sentido anti-horário (↑ = 90°, ← = 180°, ↓ = 270°).
- `{"nome": "Fat", "fx": -10, "fy": 0}`: componentes horizontal e vertical.

Opcionais em qualquer força: `cor` (`azul`, `vermelho`, `verde`, `dourado`, `ciano`, `preto`).
Sem `nome`, a força se chama F1, F2…

Os ângulos das forças são sempre medidos a partir da **horizontal**, mesmo com `inclinacao`.
Num plano inclinado de 30°, a normal fica em 90 + 30 = 120° e o peso em 270°.

Exemplo (bloco em um plano inclinado de 30°, com a componente do peso):

```figura
{
  "tipo": "diagrama_corpo_livre",
  "titulo": "Bloco no plano inclinado",
  "inclinacao": 30,
  "forcas": [
    {"nome": "P", "modulo": 50, "angulo": 270},
    {"nome": "N", "modulo": 43.3, "angulo": 120},
    {"nome": "Fat", "modulo": 25, "angulo": 30}
  ],
  "componentes": ["P"],
  "angulos": true
}
```

Exemplo (corpo pendurado por uma corda inclinada, com a resultante):

```figura
{"tipo": "diagrama_corpo_livre", "forcas": [{"nome": "T", "modulo": 30, "angulo": 53}, {"nome": "P", "modulo": 20, "angulo": 270}, {"nome": "F", "fx": 10, "fy": 0}], "resultante": true}
```

## Instruções para quem escreve o plano (NotebookLM)

1. Para inserir uma figura, escreva o bloco `figura` no ponto do texto onde ela deve aparecer.
   Não descreva a imagem em palavras nem invente links de imagem.
2. Use só os campos listados acima, com aspas retas e números com ponto decimal.
3. Em fórmulas, use `*` para multiplicar e `^` para potência (`8*x - x^2`).
4. No diagrama de corpo livre, informe só as forças do enunciado. Não escreva a resultante nem as
   componentes: peça-as com `"resultante": true` ou `"componentes"` quando a aula precisar.
5. Depois do bloco, comente a figura em texto normal.
