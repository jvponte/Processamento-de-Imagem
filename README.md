# Remoção de fundo em foto de pessoas

## Objetivo principal

O objetivo do projeto é trocar o fundo da imagem de uma pessoa, recortanto sua silhueta, removendo o restante da imagem e substituindo o fundo por alguma outra imagem.

## Imagens de entrada
Uma das imagens de entrada será a imagem de uma pessoa em fundo relativamente simples (paredes, fundos coloridos, sem texturas complexas), posicionada no centro da imagem. Como é um tipo de imagem relativamente comum e fácil de ser encontrado, serão obtidas fotos da internet (Google Images) que se encaixam nesse padrão e, se necessário, também do dataset [CelebA](https://mmlab.ie.cuhk.edu.hk/projects/CelebA.html).

Além disso, será recebida também a imagem a ser colocada no fundo. Essa imagem, entretanto, não precisa ter nenhuma restrição. Serão obtidas, também no Google Images, imagens que fazem sentido para serem o fundo de imagens de pessoas.

### Exemplos de fotos de input (pessoas):

<img src="https://user-images.githubusercontent.com/26654694/123514987-127e2100-d66c-11eb-8a3c-f00d3e753292.jpg" width=300>
<img src="https://i.pinimg.com/736x/45/8d/d4/458dd4483c1a91323f1c226fd3031f08.jpg" width=300>
<img src="https://veja.abril.com.br/wp-content/uploads/2018/05/modelo-branca-oculos-20170723-001.jpg" width=300>

### Exemplos de fotos de input (fundos):

<img src="https://cdn.awsli.com.br/600x450/1190/1190687/produto/44592263/595010e202.jpg" width=300>
<img src="https://diariodonordeste.verdesmares.com.br/image/contentid/policy:1.3071527:1618093643/Praia.jpg?f=16x9&h=720&q=0.8&w=1280&$p$f$h$q$w=aa52200" width=300>
<img src="https://super.abril.com.br/wp-content/uploads/2018/04/bibliotecas.png" width=300>

## Passos a serem realizados 

1- Transformar a imagem da pessoa de RGB (Red-Green-Blue) para HSV (Hue-Saturation-Value) e, a partir disso, utilizar apenas a dimensão Value (V) do HSV, que não leva em conta a matiz ou saturação das cores da imagem, correspondendo simplesmente a imagem em preto em branco.

![image](https://user-images.githubusercontent.com/26654694/123513753-3427da00-d665-11eb-89a4-504e36839a54.png)

2- Filtrar a imagem da pessoa em preto em branco a fim de detectar as bordas (edges) correspondentes à silhueta da pessoa da foto. Será utilizado o filtro LoG (Laplacian of Gaussian), que aplica um filtro laplaciano no resultado da aplicação de um filtro gaussiano. O filtro gaussiano é utilizado para suavizar a imagem (fazer uma espécie de Blur), reduzindo a sensitividade a ruídos, resultando em um melhor eficácia do filtro laplaciano, que é utilizado para detectar regiões de rápida mudança de intensidade (ou seja, bordas). É esperado, como resultado, uma imagem como abaixo:

![image](https://user-images.githubusercontent.com/26654694/123515750-2414f800-d66f-11eb-8c20-8e763c9fa649.png)

3- É aplicado, então, na imagem filtrada, um algoritmo de segmentação. Por questões de facilidade, será detectada a região do background, e não a pessoa, pois a pessoa tem mais bordas além de sua silhueta, enquanto o background é uma região contínua limitada pela silhueta. 
As imagens de entrada desse algoritmo serão imagens com preto e branco, onde o branco indica uma variação de intensidade na imagem original, ou seja, uma borda. Então, para definir o background, a imagem será varrida linha a linha da direita para a esquerda, e os pontos serão adicionados a região do background até que se encontre um ponto com valor acima de um valor definido de tolerancia, indicando que foi encontrada uma borda. O mesmo processo é feito para no sentido inverso, da esquerda para direita e, assim, as bordas da pessoa são definidas, criando uma máscara como a seguir. Em alguns casos com silhuetas que tenham regiões côncavas, pode ser necessário um aprimoramento. Esse aprimoramento pode ser feito repetindo o processo de verredura na direção vertical, ou utilizando uma variação do algoritmo growing region para expandir a região do background para os demais pontos.

![image](https://user-images.githubusercontent.com/26654694/123515765-3000ba00-d66f-11eb-8d93-27d263202b16.png)

4- O resultado do passo anterior é utilizado para filtrar tanto a imagem da pessoa como do fundo desejado. Entretanto, como devem ser "recortadas" regiões distintas de cada uma delas, é necessário inverter a imagem antes de ser utilizada como máscara ou utilizar uma lógica diferente para selecionar o que será mantido em cada uma. Será necessário, em alguns casos, recortar a imagem de fundo para ter o mesmo tamanho da imagem da pessoa. Espera-se obter resultados como:

![image](https://user-images.githubusercontent.com/26654694/123515470-e368af00-d66d-11eb-990e-3b18334c97da.png)
![image](https://user-images.githubusercontent.com/26654694/123515475-e8c5f980-d66d-11eb-96ef-3ef096820d65.png)

5- Por fim, as imagens são unidas pare chegar ao resultado final.

![image](https://user-images.githubusercontent.com/26654694/123515488-f8ddd900-d66d-11eb-99da-817e89d9637d.png)

Em resumo, esse será o pipeline de processamento a ser seguido pelo projeto:

![image](https://user-images.githubusercontent.com/26654694/123515502-085d2200-d66e-11eb-880f-64cb1afa14d3.png)

Obs: as imagens mostradas nesse passo a passo foram feitas com ajuda de um programa de edição de imagens, e não do código. Os resutados obtidos até o momento estão no notebook do repositório.

## Códigos iniciais

Já foram feitos testes que contemplam todas as etapas descritas acima, com as imagens de entrada mostradas como exemplo. O notebok com os código e resultados, bem como as imagens, foram adicionadas ao repositório. 

Entretanto, houve uma maior dificuldade no passo 2, na aplicação do filtro LoG para detecção de bordas e os testes com diferentes tamanhos de filtros e valores de desvio padrão não tiveram resultados satisfatórios até então. Para continuar o desenvolvimento e testes dos passos seguintes, esse passo foi simulado em um programa externo de edição de imagens, gerando as imagens com as bordas já demarcadas. No repositório, essas imagens estão identificadas pelo sufixo '_filt'.

O restante dos passos foram realizados e foram obtidos resultados satisfatórios, mostrados no próprio notebook. Entretanto, ainda há espaço para melhorias, como a citada na descrição do passo 3 para solucionar o problema de regiões côncavas. Esse problema pode ser observado nos casos de teste 2 e 3, no fim do notebook. Também podem ser estudadas formas de reduzir as bordas deixadas pela máscara na silhueta da pessoa.
