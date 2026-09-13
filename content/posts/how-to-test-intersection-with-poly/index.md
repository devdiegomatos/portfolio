---
date: '2026-07-18T11:30:00-03:00'
draft: false
title: 'Teste de ponto em polígono usando o algoritmo do Número de Voltas'
math: true
cover:
  image: "cover.webp"
  alt: "Vector illustration for cover"
---

Recentemente comecei a trabalhar na Tecgraf PUC-Rio e precisei resolver um problema interessante: dado um polígono desenhado pelo usuário, o sistema deve selecionar todos os objetos contidos, total ou parcialmente, em seu interior.

<!-- more -->

## Introdução
À primeira vista esse problema parece apenas mais um teste de interseção entre polígonos. Como o sistema trabalha em 2 dimensões a minha primeira ideia foi utilizar o Teorema do Eixo Separador (SAT).

Entretanto, o polígono de seleção pode possuir qualquer formato, sendo convexo ou côncavo. O SAT, por outro lado, pressupõe que ambos os polígonos sejam convexos. Para utilizá-lo seria necessário decompor previamente o polígono de seleção em um conjunto de polígonos convexos menores, utilizando algoritmos como *Ear Clipping*, e somente então executar os testes de interseção.

Embora essa abordagem seja perfeitamente válida, ela me pareceu desnecessariamente complexa para o problema que eu precisava resolver.

No meu caso, seria suficiente determinar se um dos vértices de cada objeto estava contido no polígono de seleção. Dessa forma, o problema pode ser reduzido a um problema clássico da Geometria Computacional: determinar se um ponto pertence ao interior de um polígono.

Sorte a minha que esse problema já estava resolvido dentro da aplicação pois, é isso que se faz para determinar se o clique do mouse havia selecionado o polígono. Ainda assim, como essa função funciona?

## O problema Ponto-no-Polígono
O problema de determinar se um ponto está contido em um polígono, aparece em diversas áreas da computação gráfica, motores de física, sistemas de informação geográfica (GIS), CAD e visão computacional.

Seja um polígono fechado P e um ponto \\(q \in \mathbb{R}^2\\). Nosso objetivo é classificar *q* em uma das seguintes regiões:

* interior do polígono;
* fronteira do polígono;
* exterior do polígono.

Existem diversos algoritmos para resolvê-lo. Os mais conhecidos são:

* Número de Cruzamentos: conta quantas vezes uma semirreta lançada a partir do ponto cruza o contorno do polígono;
* Número de Voltas: contabiliza quantas vezes o contorno do polígono envolve o ponto.

Walaber usa um algoritmo de traçado de raios (conta o número de cruzamentos) em seu sistema de detecção de colisão em seu jogo JellyCar. Neste texto apresentarei um algoritmo baseado no Número de Voltas, proposto por Dan Sunday, por ser simples de implementar, eficiente e adequado ao meu problema.

## Algoritmo do Número de Voltas
Considere um ponto *q* qualquer, um polígono P representado por seus vértices ordenados \\(P={v_0,v_1,\ldots,v_{n-1}}\\), onde \\(v_n=v_0\\), de forma que cada par consecutivo de vértices define uma aresta do polígono e o número de voltas \\(wn\\).

A ideia do algoritmo é bastante simples. Traça-se uma semirreta horizontal partindo do ponto *q* para a direita. Em seguida percorremos todas as arestas do polígono, observando como elas cruzam essa semirreta.

Sempre que uma aresta cruza a semirreta de baixo para cima, contabilizamos uma contribuição positiva. Quando a aresta cruza de cima para baixo, contabilizamos uma contribuição negativa. A Figura 1 ilustra esse processo.

{{<rawhtml>}}
<figure>
    <img src="./Winding_number_algorithm_example.svg" alt="Exemplo do algoritmo Winding Number" style="background-color: white; display: block; margin: auto">
    <figcaption style="text-align:center">
        Figura 1 – Funcionamento do algoritmo do Número de Voltas. Fonte: Adaptado de Avelludo (Wikimedia Commons).
    </figcaption>
</figure>
{{</rawhtml>}}

A intuição por trás do algoritmo é que, se o ponto estiver fora do polígono, toda vez que uma aresta entrar na região à direita da semirreta haverá outra aresta saindo dessa região. Assim, as contribuições positivas e negativas se cancelam, produzindo \\(wn = 0\\). Por outro lado, quando o ponto está no interior do polígono, esse cancelamento não ocorre completamente e o número de voltas permanece diferente de zero. Portanto,

* (wn = 0): o ponto está no exterior do polígono;
* (wn = 1): o polígono envolve o ponto uma vez no sentido anti-horário;
* (wn = -1): o polígono envolve o ponto uma vez no sentido horário;
* (|wn| > 1): o contorno envolve o ponto múltiplas vezes (caso possível em polígonos auto-intersectantes).

O algoritmo pode ser implementado da seguinte forma:

```pseudocode
function windingNumber(Q, P)
    wn = 0
    para cada aresta E de P
        se E cruza a semirreta de baixo para cima
            se Q está à esquerda de E
                wn++
        senão se E cruza a semirreta de cima para baixo
            se Q está à esquerda de E
                wn--
    return wn
```

Repare que a semireta horizontal partindo do ponto *q* para direita é virtual, testar se o ponto *q* está à esquerda é o suficiente.

Como cada aresta é analisada exatamente uma vez, a complexidade do algoritmo é O(n), onde n é o número de vértices do polígono. No meu caso, o número de vértices do polígono de seleção costuma ser pequeno, tornando essa solução extremamente simples e suficientemente eficiente.

## Teste de orientação
Até agora omitimos um detalhe importante: como determinar se um ponto está à esquerda de uma aresta? Dan Sunday resolve esse problema utilizando um teste de orientação baseado no sinal do produto vetorial entre dois vetores. Dessa forma evitando um teste explícito de interseção entre a semirreta e a aresta. 

Considere uma aresta representado pelo vetor \\( \vec s = \vec{v_{i+1}} - \vec{v_i} \\) e o vetor \\( \vec r = \vec q - \vec{v_i} \\).

{{<rawhtml>}}
<figure>
    <img src="./teste-orientacao.svg" alt="Visualização do teste de orientação" style="background-color: white; display: block; margin: auto; padding: 1rem;">
    <figcaption style="text-align:center">
        Figura 2 – Visualização do teste de orientação.
    </figcaption>
</figure>
{{</rawhtml>}}

O sinal do determinante

$$ \vec r \times \vec s = \begin{vmatrix} r_x & s_x \\\\ r_y & s_y \end{vmatrix} = r_xs_y-s_xr_y $$

permite determinar a orientação relativa entre os vetores.

* resultado positivo: o ponto está à esquerda da aresta;
* resultado negativo: o ponto está à direita da aresta;
* resultado igual a zero: o ponto é colinear com a aresta.

{{<rawhtml>}}

<iframe src="https://devdiegomatos.github.io/point-in-poly-winding-number-algorithm/" width="100%" height="400px"></iframe>

{{</rawhtml>}}

## Tratando casos de borda
Uma dificuldade desse algoritmo ocorre quando a semirreta horizontal passa exatamente por um vértice do polígono.

Nessa situação, duas arestas compartilham o mesmo ponto de interseção e uma implementação ingênua pode contabilizar esse cruzamento duas vezes. Alciatore e Miranda propõem tratar esses casos atribuindo meia contribuição \\(\pm\frac{1}{2}\\) aos vértices, evitando contagens duplicadas.
Quando a semirreta passa exatamente por um vértice horizontal, nenhuma contribuição adicional deve ser contabilizada. Veja a Figura 3.

{{<rawhtml>}}
<figure>
    <img src="./casos-borda.svg" alt="Casos de borda" style="background-color: white; display: block; margin: auto; padding: 1rem;">
    <figcaption style="text-align:center">
        Figura 3 – Casos de borda
    </figcaption>
</figure>
{{</rawhtml>}}

## Limitações
Embora o teste de ponto em polígono seja suficiente para o problema encontrado na aplicação, ele não resolve o problema geral de interseção entre polígonos. Considere, por exemplo, a situação abaixo.

{{<rawhtml>}}
<figure>
    <img src="./limitacao.svg" alt="Limitação do algoritmo" style="background-color: white; display: block; margin: auto; padding: 1rem;">
    <figcaption style="text-align:center">
        Figura 4 – Limitação do algoritmo
    </figcaption>
</figure>
{{</rawhtml>}}

Os dois polígonos se intersectam. Entretanto, nenhum vértice de um polígono pertence ao interior do outro. Assim, um algoritmo baseado exclusivamente em testes de ponto-no-polígono concluirá incorretamente que não existe interseção entre eles, produzindo um falso negativo. 

No contexto deste trabalho essa limitação não representa um problema, pois as características dos objetos manipulados permitem reduzir o teste de interseção a testes de inclusão de pontos.

## Referências

1. Sunday, D. *Inclusion of a Point in a Polygon*. SoftSurfer Geometry Algorithms.
1. ALCIATORE, D.; MIRANDA, R. A winding number and point-in-polygon algorithm. Fort Collins: Department of Mechanical Engineering, Colorado State University, 1995.
1. Avelludo. *Winding Number Algorithm Example*. Wikimedia Commons. https://commons.wikimedia.org/wiki/File:Winding_number_algorithm_example.svg
1. *Physics of Jellycar*, por Walaber: https://youtu.be/3OmkehAJoyo?si=0Y1m1r2XhRRhg3Zs
