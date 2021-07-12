# Remoção de fundo em foto de pessoas

## Objetivo principal

O objetivo do projeto é trocar o fundo da imagem de uma pessoa, recortanto sua silhueta, removendo o restante da imagem e substituindo o fundo por alguma outra imagem.

## Imagens de entrada
Uma das imagens de entrada será a imagem de uma pessoa em fundo simples e relativamente uniforme (paredes, fundos coloridos, chroma key, etc.) Como é um tipo de imagem relativamente comum e fácil de ser encontrado, serão obtidas fotos da internet (Google Images) que se encaixam nesse padrão.

Além disso, será recebida também a imagem a ser colocada no fundo. Essa imagem, entretanto, não precisa ter nenhuma restrição. Serão obtidas, também no Google Images, imagens que fazem sentido para serem o fundo de imagens de pessoas.

### Exemplos de fotos de input (pessoas):

<img src="https://veja.abril.com.br/wp-content/uploads/2018/05/modelo-branca-oculos-20170723-001.jpg" width=300>
<img src="https://user-images.githubusercontent.com/26654694/123514987-127e2100-d66c-11eb-8a3c-f00d3e753292.jpg" width=300>
<img src="https://i.pinimg.com/736x/45/8d/d4/458dd4483c1a91323f1c226fd3031f08.jpg" width=300>

### Exemplos de fotos de input (fundos):

<img src="https://super.abril.com.br/wp-content/uploads/2018/04/bibliotecas.png" width=300>
<img src="https://cdn.awsli.com.br/600x450/1190/1190687/produto/44592263/595010e202.jpg" width=300>
<img src="https://diariodonordeste.verdesmares.com.br/image/contentid/policy:1.3071527:1618093643/Praia.jpg?f=16x9&h=720&q=0.8&w=1280&$p$f$h$q$w=aa52200" width=300>

## Passo a passo realizado

1- O primeiro passo, antes de detectar as bordas da pessoa, é transformar a imagem original RGB de 3 canais para apenas 1 canal (imagem em preto e branco) para isso ser feito, foi utilizada a técnica de luminance que faz uma média ponderada dos 3 canais de acordo com como são percebidos pela visão humana (0.299R + 0.587G + 0.114B).

Obs: Anteriormente, estava sendo feita uma conversão para HSV e selecionando apenas o canal de Value. Entretanto, esse método desconsidera a diferença de percepção de cores pela visão humana e poderia dificultar a detecção das bordas em alguns casos.

![image](https://user-images.githubusercontent.com/26654694/125357296-d3bfba80-e33d-11eb-862f-a0ac8626fdfa.png)

2- Filtrar a imagem da pessoa em preto em branco a fim de detectar as bordas (edges) correspondentes à silhueta da pessoa da foto. Foi utilizado o filtro LoG (Laplacian of Gaussian), que aplica um filtro laplaciano no resultado da aplicação de um filtro gaussiano. O filtro gaussiano é utilizado para suavizar a imagem (fazer um blur), reduzindo a sensitividade a ruídos, resultando em um melhor eficácia do filtro laplaciano, que é utilizado para detectar regiões de rápida mudança de intensidade (ou seja, bordas). 

![image](https://user-images.githubusercontent.com/26654694/125357549-24cfae80-e33e-11eb-8741-4d0e0b4f7d39.png)

3- É aplicado, então, na imagem filtrada, um algoritmo de segmentação. Por questões de facilidade, será detectada a região do background, e não a pessoa, pois a pessoa tem mais bordas além de sua silhueta, enquanto o background é uma região contínua limitada pela silhueta. 
As imagens de entrada desse algoritmo são imagens com preto e branco, onde o branco indica uma variação de intensidade na imagem original, ou seja, uma borda. Então, para definir o background, a imagem é varrida linha a linha da direita para a esquerda, e os pontos são adicionados a região do background até que se encontre um ponto com valor acima de um valor definido de tolerância, indicando que foi encontrada uma borda. O mesmo processo é feito para no sentido inverso, da esquerda para direita e, assim, as bordas da pessoa são definidas, criando uma máscara como a seguir.

![image](https://user-images.githubusercontent.com/26654694/125357830-87c14580-e33e-11eb-8c14-29056e7773ca.png)

4- A máscara resultante no passo anterior não consegue de forma totalmente satisfatória segmentar a pessoa do fundo e, portanto, foi feito alguns passos adicionais para incrementar essa segmentação. A máscara criada é utilizada na imagem original para delimitar a área correspondente ao fundo da imagem, e com, isso, é calculada a cor média do fundo. A partir dessa cor, é definido um valor de tolerância para gerar um intervalo de cores correspondentes ao fundo e é criada então uma máscara com os pixels que têm suas cores dentro desse intervalo.

![image](https://user-images.githubusercontent.com/26654694/125358206-fef6d980-e33e-11eb-9321-3d4be0cbd7b9.png)
![image](https://user-images.githubusercontent.com/26654694/125358220-05855100-e33f-11eb-915f-362cd3b29008.png)
![image](https://user-images.githubusercontent.com/26654694/125358261-0f0eb900-e33f-11eb-9c67-404f4c48706d.png)

5- As máscaras criadas nos passos 3 e 4 são unidas para gerar a máscara final. De forma geral, a máscara final é basicamente igual a última, mas elas podem diferir em caso de o fundo ter alguma região de cor muito distinta da média.

![image](https://user-images.githubusercontent.com/26654694/125358789-c3104400-e33f-11eb-8fbd-4a32bd90f1b4.png)

6- A máscara criada no passo anterior é utilizada para filtrar tanto a imagem da pessoa como do fundo desejado. Entretanto, como devem ser "recortadas" regiões distintas de cada uma delas, é utilizada uma lógica diferente para selecionar o que será mantido em cada uma (em uma, se mantem os zeros, em outra, os uns). É necessário também redimensionar a imagem de fundo para ter o mesmo tamanho da máscara (e da imagem da pessoa). 

![image](https://user-images.githubusercontent.com/26654694/125358825-cefc0600-e33f-11eb-9e50-7223f9f7c5a8.png)
![image](https://user-images.githubusercontent.com/26654694/125358912-ee932e80-e33f-11eb-9f43-f55f41548430.png)

7- Por fim, as imagens são unidas pare chegar ao resultado final, simplesmente fazendo uma soma (pixel a pixel).
![image](https://user-images.githubusercontent.com/26654694/125358843-d6231400-e33f-11eb-8278-df61cb5e5d70.png)


## Exemplos de resultados (inputs e outputs)

![image](https://user-images.githubusercontent.com/26654694/125359156-4467d680-e340-11eb-9c65-e472774a5696.png)
![image](https://user-images.githubusercontent.com/26654694/125359169-4a5db780-e340-11eb-94f8-5eb1bcc733fa.png)
![image](https://user-images.githubusercontent.com/26654694/125359193-521d5c00-e340-11eb-940f-a00ec1de6b98.png)
