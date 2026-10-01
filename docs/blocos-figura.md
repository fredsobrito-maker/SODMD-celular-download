# Blocos de figura do SODMD

O SODMD (celular, versão 1.3 em diante) desenha figuras a partir de blocos de código
com a linguagem `figura`. O bloco guarda só os dados; o desenho é feito na hora de ler
o plano, sem internet. Se o bloco tiver erro, aparece uma mensagem no lugar da figura
e o resto do plano continua normal.

Formato: um bloco de código cercado por três crases, com a palavra `figura` na primeira
linha, e um objeto JSON dentro. Aspas retas (`"`), sem comentários.

## 1. Espelho esférico (`espelho_esferico`)

Calcula a imagem por 1/f = 1/p + 1/p′ e A = −p′/p e desenha os três raios principais.

| Campo | Obrigatório | Valores |
|---|---|---|
| `tipo` | sim | `"espelho_esferico"` |
| `espelho` | sim | `"concavo"`, `"convexo"` ou `"plano"` |
| `f` | sim, exceto no plano | distância focal em cm, número positivo |
| `p` | sim | distância do objeto ao espelho em cm, positivo |
| `h` | sim | altura do objeto em cm, positivo |
| `titulo` | não | texto acima da figura |

Exemplo:

```figura
{"tipo": "espelho_esferico", "espelho": "concavo", "f": 10, "p": 25, "h": 4}
```

Casos clássicos do espelho côncavo com f = 10: `p` = 25 (além de C), 20 (em C),
15 (entre C e F), 10 (no foco: imagem no infinito), 5 (entre F e V: imagem virtual).

## 2. Lente esférica (`lente_esferica`)

Lente delgada. Calcula a imagem por 1/f = 1/p + 1/p′ e A = −p′/p e desenha os três raios
principais (paralelo ao eixo, pelo centro óptico O e dirigido ao foco). Objeto à esquerda,
luz da esquerda para a direita.

| Campo | Obrigatório | Valores |
|---|---|---|
| `tipo` | sim | `"lente_esferica"` |
| `lente` | sim | `"convergente"` ou `"divergente"` |
| `f` | sim | distância focal em cm, número positivo (o app aplica o sinal) |
| `p` | sim | distância do objeto à lente em cm, positivo |
| `h` | sim | altura do objeto em cm, positivo |
| `titulo` | não | texto acima da figura |

Exemplo:

```figura
{"tipo": "lente_esferica", "lente": "convergente", "f": 10, "p": 25, "h": 4}
```

Casos clássicos da lente convergente com f = 10: `p` = 25 (além de 2f), 20 (em 2f),
15 (entre f e 2f), 10 (no foco: raios paralelos), 5 (entre a lente e o foco: imagem virtual).
A lente divergente sempre forma imagem virtual, direita e menor.

## 3. Hidrostática dos fluidos (`hidrostatica_fluidos`)

Um só tipo, com o campo `situacao` escolhendo o desenho. O aplicativo calcula pressões,
forças, empuxo e o valor que faltar; o plano nunca traz o resultado. Unidades no SI
(kg/m³, m, m², m³, N, Pa) e g = 10 m/s², salvo se o bloco disser outra coisa (`g`, `unidade`,
`unidade_area`, `unidade_volume`). Resultados saem com vírgula decimal e ponto nos milhares.

**`"situacao": "coluna"`** (Stevin): líquido num recipiente, com setas de pressão e pontos marcados.

| Campo | Obrigatório | Valores |
|---|---|---|
| `densidade` | sim | kg/m³ |
| `h` | sim | profundidade do líquido |
| `pontos` | não | lista de 1 a 5 profundidades (A, B, C…) em que mostrar a pressão; padrão: só o fundo |
| `p0` | não | pressão sobre a superfície, em Pa. Se faltar, mostra a pressão efetiva d·g·h |
| `unidade` | não | `"m"` (padrão) ou `"cm"`, vale para `h` e `pontos` |

```figura
{"tipo": "hidrostatica_fluidos", "situacao": "coluna", "densidade": 1000, "h": 2, "pontos": [0.5, 1, 2], "p0": 100000}
```

**`"situacao": "tubo_u"`** (vasos comunicantes): dois líquidos que não se misturam, com a regra
d·h do lado leve = d·h do lado denso. Informe três dos quatro valores e o aplicativo calcula o
quarto; se informar os quatro e eles não baterem, aparece um aviso.

| Campo | Valores |
|---|---|
| `d_leve`, `h_leve` | densidade e altura da coluna do líquido menos denso, acima do nível de referência |
| `d_denso`, `h_denso` | densidade e altura da coluna do líquido mais denso, acima do mesmo nível |
| `unidade` | `"m"` (padrão) ou `"cm"` para as alturas |

```figura
{"tipo": "hidrostatica_fluidos", "situacao": "tubo_u", "d_leve": 800, "h_leve": 10, "d_denso": 1000, "unidade": "cm"}
```

**`"situacao": "prensa"`** (Pascal): F₁/A₁ = F₂/A₂. Informe uma só das forças.

| Campo | Valores |
|---|---|
| `A1`, `A2` | áreas dos êmbolos (obrigatórias) |
| `F1` ou `F2` | a força conhecida; a outra é calculada |
| `d1` | deslocamento do êmbolo 1 (opcional); o app calcula o do êmbolo 2 |
| `unidade_area` | `"m2"` (padrão) ou `"cm2"` |
| `unidade` | `"m"` (padrão) ou `"cm"`, para `d1` |

```figura
{"tipo": "hidrostatica_fluidos", "situacao": "prensa", "A1": 10, "A2": 200, "unidade_area": "cm2", "F1": 50, "d1": 0.4}
```

**`"situacao": "empuxo"`** (Arquimedes): corpo num líquido; o app decide se flutua, fica em
equilíbrio ou afunda, e mostra P, E e (se afunda) a normal N.

| Campo | Valores |
|---|---|
| `d_liquido`, `d_corpo` | densidades em kg/m³ |
| `V` | volume do corpo |
| `unidade_volume` | `"m3"` (padrão), `"cm3"` ou `"L"` |
| `g` | opcional, padrão 10 |

```figura
{"tipo": "hidrostatica_fluidos", "situacao": "empuxo", "d_liquido": 1000, "d_corpo": 600, "V": 0.002}
```

## 4. Eletromagnetismo (`eletromagnetismo`)

Um só tipo, com o campo `situacao` escolhendo o desenho. Corrente convencional (do positivo para o
negativo). Sentidos em relação à página: `"saindo"` (⊙) ou `"entrando"` (⊗). O aplicativo aplica a regra da
mão direita, calcula os valores e desenha os sentidos; o plano nunca traz esses resultados. SI por padrão
(A, T, m, N, C, kg, V, Ω); `unidade` aceita `"m"` ou `"cm"` onde houver comprimento. Constante μ₀ = 4π·10⁻⁷ T·m/A.

| `situacao` | Campos obrigatórios | Opcionais | O aplicativo calcula e mostra |
|---|---|---|---|
| `"fio"` (Oersted) | `i`, `sentido` (`"saindo"` ou `"entrando"`) | `pontos` (até 3 distâncias), `unidade` | sentido de B (circunferências) e B = μ₀·i/(2π·r) em cada ponto |
| `"espira"` | `i`, `R`, `sentido` (`"horario"` ou `"antihorario"`) | `N` (padrão 1), `unidade` | B no centro = μ₀·N·i/(2R) e seu sentido |
| `"solenoide"` | `i`, `campo` (`"direita"` ou `"esquerda"`: sentido de B no interior) e `n` (espiras por metro) **ou** `N` e `L` | `unidade` | B = μ₀·n·i, polos N e S, sentido da corrente |
| `"carga"` | `q` (em C, com sinal), `v`, `B`, `campo` (`"saindo"`/`"entrando"`) | `m` (kg), `angulo` (`90` padrão ou `0`) | F = \|q\|·v·B, sentido da força, R = m·v/(\|q\|·B) e T se houver `m` |
| `"fio_em_campo"` | `i`, `L`, `B`, `campo` (`"saindo"`/`"entrando"`), `corrente` (`"direita"`/`"esquerda"`) | `unidade` | F = B·i·L e seu sentido |
| `"inducao"` (Faraday e Lenz) | `ima` (`"aproximando"` ou `"afastando"`), `polo` (`"norte"` ou `"sul"`, o voltado para a espira) | `fluxo_inicial` e `fluxo_final` (Wb, juntos), `tempo` (s), `R` (Ω) | sentido da corrente induzida (Lenz), ε = \|ΔΦ\|/Δt e i = ε/R |
| `"ima"` com `"ima": "barra"` | `ima` | `polo_direito` (`"norte"` padrão ou `"sul"`) | linhas de campo fora e dentro do ímã |
| `"ima"` com `"ima": "autofalante"` | `ima`, `forca` (`"para_fora"` ou `"para_dentro"`: movimento do cone) | `i`, `B`, `L` (juntos) | campo radial no entreferro, sentido da corrente na bobina e F = B·i·L |

Exemplos (um de cada situação):

```figura
{"tipo": "eletromagnetismo", "situacao": "fio", "i": 5, "sentido": "saindo", "pontos": [0.1, 0.2, 0.4]}
```

```figura
{"tipo": "eletromagnetismo", "situacao": "espira", "i": 2, "R": 0.1, "N": 10, "sentido": "antihorario"}
```

```figura
{"tipo": "eletromagnetismo", "situacao": "solenoide", "i": 2, "N": 500, "L": 0.25, "campo": "direita"}
```

```figura
{"tipo": "eletromagnetismo", "situacao": "carga", "q": 1.6e-19, "m": 1.67e-27, "v": 1000000, "B": 0.5, "campo": "saindo"}
```

```figura
{"tipo": "eletromagnetismo", "situacao": "fio_em_campo", "i": 4, "L": 0.5, "B": 0.2, "campo": "saindo", "corrente": "direita"}
```

```figura
{"tipo": "eletromagnetismo", "situacao": "inducao", "ima": "aproximando", "polo": "norte", "fluxo_inicial": 0, "fluxo_final": 0.02, "tempo": 0.5, "R": 4}
```

```figura
{"tipo": "eletromagnetismo", "situacao": "ima", "ima": "barra"}
```

```figura
{"tipo": "eletromagnetismo", "situacao": "ima", "ima": "autofalante", "forca": "para_fora", "i": 1, "B": 0.8, "L": 5}
```

## 5. Circuito elétrico (`circuito_eletrico`)

Resistores ligados a uma fonte. O campo `circuito` descreve a ligação com `+` (em série), `//` (em paralelo)
e parênteses; `//` vale antes de `+`, mas use parênteses sempre que houver dúvida. Cada resistor de `resistores`
aparece **uma vez** (e só uma) em `circuito`. O aplicativo desenha o esquema, resolve o circuito e mostra
resistência equivalente, correntes, tensões e potências.

| Campo | Obrigatório | Valores |
|---|---|---|
| `tipo` | sim | `"circuito_eletrico"` |
| `fem` | sim | força eletromotriz da fonte, em V |
| `resistores` | sim | objeto com 1 a 8 resistores em Ω, como `{"R1": 10, "R2": 20}` (nomes com letras e números) |
| `circuito` | sim | texto, como `"R1 + (R2 // R3)"` |
| `r` | não | resistência interna do gerador, em Ω (padrão 0). Com `r`, o app mostra a tensão nos terminais e o rendimento |
| `medidores` | não | lista de até 4 itens `{"tipo": "amperimetro" ou "voltimetro", "em": "R2" ou "fonte"}`. Amperímetro em série com o resistor (ou na linha principal, com `"fonte"`); voltímetro em paralelo com o resistor (ou nos terminais da fonte) |
| `mostrar` | não | lista do que escrever em cada resistor: `"i"`, `"V"`, `"P"` e `"leituras"` (dos medidores). Padrão: `["i", "V", "leituras"]`. Use `[]` para uma figura de exercício, só com os dados |
| `titulo` | não | texto acima da figura |

A corrente convencional sai do polo positivo da fonte (traço longo). Os medidores são ideais.

Exemplos (série e paralelo, gerador real com medidores, associação mista com medidores e figura de exercício):

```figura
{"tipo": "circuito_eletrico", "fem": 12, "resistores": {"R1": 10, "R2": 20, "R3": 30}, "circuito": "R1 + (R2 // R3)"}
```

```figura
{"tipo": "circuito_eletrico", "fem": 12, "r": 1, "resistores": {"R1": 5, "R2": 6}, "circuito": "R1 // R2", "medidores": [{"tipo": "amperimetro", "em": "fonte"}, {"tipo": "voltimetro", "em": "fonte"}]}
```

```figura
{"tipo": "circuito_eletrico", "fem": 9, "resistores": {"R1": 6, "R2": 3, "R3": 2, "R4": 4}, "circuito": "(R1 // R2) + (R3 // R4)", "medidores": [{"tipo": "amperimetro", "em": "R1"}, {"tipo": "voltimetro", "em": "R3"}], "mostrar": ["i", "V", "P", "leituras"]}
```

```figura
{"tipo": "circuito_eletrico", "fem": 12, "resistores": {"R1": 10, "R2": 20, "R3": 30}, "circuito": "R1 + R2 + R3", "mostrar": []}
```

## 6. Ondulatória e espectro (`espectro_ondulatoria`)

Um só tipo, com o campo `situacao`. O aplicativo calcula o que faltar (`v = λ·f`, `T = 1/f`, harmônicos, frequência
percebida). Com `mostrar`, você escolhe o que escrever na figura; `"mostrar": []` esconde os valores
(figura de exercício). Gráficos de MHS (x(t), v(t), a(t)) e de intensidade usam o `plano_cartesiano`.

| `situacao` | Campos | O aplicativo calcula e mostra |
|---|---|---|
| `"onda"` | informe **dois** entre `lambda` (m), `f` (Hz) ou `T` (s), e `v` (m/s); opcional `A` (amplitude, m) e `mostrar` (`"lambda"`, `"f"`, `"T"`, `"v"`, `"A"`) | onda transversal com λ e A marcados; os valores que faltarem. Se os informados forem incompatíveis, aparece um aviso |
| `"harmonicos"` | `meio` (`"corda"`, `"tubo_aberto"` ou `"tubo_fechado"`), `L` (m), `n` (inteiro de 1 a 9 ou lista de até 4; no tubo fechado só ímpares); opcionais `v` (m/s) e `mostrar` (`"lambda"`, `"f"`) | modos de vibração com nós, λ = 2L/n (corda e tubo aberto) ou 4L/n (tubo fechado) e, com `v`, f = n·v/(2L) ou n·v/(4L) |
| `"doppler"` | `f` (Hz), `v_fonte` (m/s, menor que o som), `fonte` (`"aproximando"` ou `"afastando"` do observador); opcionais `v_som` (padrão 340), `v_obs` com `observador` (`"aproximando"` ou `"afastando"`) e `mostrar` (`["f_percebida"]` ou `[]`) | frentes de onda, observador e f′ = f·(v ± v_obs)/(v ∓ v_fonte) |
| `"espectro"` | opcional `destaque` (`"radio"`, `"microondas"`, `"infravermelho"`, `"visivel"`, `"ultravioleta"`, `"raios_x"` ou `"gama"`) | espectro eletromagnético em ordem crescente de frequência, com λ e f de cada faixa e uma faixa em destaque |

Exemplos (onda, harmônicos em corda e tubo fechado, Doppler com fonte e com observador em movimento, espectro):

```figura
{"tipo": "espectro_ondulatoria", "situacao": "onda", "A": 0.5, "lambda": 2, "f": 170}
```

```figura
{"tipo": "espectro_ondulatoria", "situacao": "harmonicos", "meio": "corda", "L": 1, "n": [1, 2, 3], "v": 100}
```

```figura
{"tipo": "espectro_ondulatoria", "situacao": "harmonicos", "meio": "tubo_fechado", "L": 0.5, "n": [1, 3], "v": 340}
```

```figura
{"tipo": "espectro_ondulatoria", "situacao": "doppler", "f": 400, "v_fonte": 30, "fonte": "aproximando"}
```

```figura
{"tipo": "espectro_ondulatoria", "situacao": "doppler", "f": 400, "v_fonte": 30, "fonte": "afastando", "v_obs": 10, "observador": "aproximando"}
```

```figura
{"tipo": "espectro_ondulatoria", "situacao": "espectro", "destaque": "visivel"}
```

Receitas de MHS com o `plano_cartesiano` (massa-mola com A = 0,1 m e f = 2 Hz):

- posição: `{"formula": "0.1*cos(2*pi*2*x)", "nome": "x(t)"}`
- velocidade: `{"formula": "-0.1*2*pi*2*sen(2*pi*2*x)", "nome": "v(t)"}`
- aceleração: `{"formula": "-0.1*(2*pi*2)^2*cos(2*pi*2*x)", "nome": "a(t)"}`

Use eixo x de 0 a 1 (tempo em s), `"legenda": true` e eixos y que comportem as amplitudes.

## 7. Plano cartesiano (`plano_cartesiano`)

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

## Instruções para quem escreve o plano (NotebookLM)

1. Para inserir uma figura, escreva o bloco `figura` no ponto do texto onde ela deve aparecer.
   Não descreva a imagem em palavras nem invente links de imagem.
2. Use só os campos listados acima, com aspas retas e números com ponto decimal.
3. Em espelhos, escolha `f`, `p` e `h` coerentes com a situação da aula. Não escreva a posição
   da imagem: ela é calculada pelo aplicativo.
4. Em fórmulas, use `*` para multiplicar e `^` para potência (`8*x - x^2`).
5. Depois do bloco, comente a figura em texto normal.
