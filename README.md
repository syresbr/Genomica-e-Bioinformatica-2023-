# Genomica e Bioinformatica-2023
  Projeto feito para a cadeira de Genômica e Bioinformática (2023) do programa de pós-graduação do CENA.

bla bla bla

  Esse pipeline serve para a descoberta e analise dos motifs presentes dentro dos promotores dos genes de _Chlamydomonas_, em estado de escassez de carbono, que possuem diferenciação na expressão.

  O arquivo Tabela_de_genes.xls possuiu os genes de interesse


## Preparação dos arquivos

### MEME

  O meme é uma ferramenta que encontra motifs dentro compartilhados por sequencias presentes dentro de um arquivo. Pra isso ele utiliza um modelo probabilistico chamado markov model (https://en.wikipedia.org/wiki/Markov_model) como background para conseguir encontrar esses motifs. É possivel utilizar o modelo de markov integrado no meme, mas para se ter uma construção mais precisa é melhor criarmos o nosso proposito. O MEME suite possuiu uma ferramenta para isso, "fasta-get-markov" que produz esse arquivo a partir de um arquivo com sequencias. Nós utilizaremos o arquivo "CRE_v5.6_promoters_1500bp.fasta" para produzir o background pois esse arquivo representa todos os promotores existentes dentro do genoma do _Chlamydomonas_.

```
fasta-get-markov -m 0 CRE_v5.6_promoters_1500bp.fasta background_0
```

  Podemos fazer com outros niveis de ordem para poder observar a diferença que isso faz nos dados.

```
fasta-get-markov -m 3 CRE_v5.6_promoters_1500bp.fasta background_1
fasta-get-markov -m 2 CRE_v5.6_promoters_1500bp.fasta background_2
fasta-get-markov -m 1 CRE_v5.6_promoters_1500bp.fasta background_3
```

  Após conseguirmos os backgrounds podemos utilizar o meme para encontrar os motifs. Importante ter em mente o que cada argumento serve dentro da linha de comando. 
`-dna` Fala qual tipo de sequencia está presente no arquivo;
`-oc` Especifica o nome da pasta que os arquivos serão colocados e irá sobreescrever as coisas dessa pasta caso ja exista
`-mod` Permite a escolha de qual dos modos de encontrar motifs.
`-nmotifs` Numero de motifs que deve ser encontrado
`-minw` Permite escolher o tamanho minimo dos motifs
`-maxw` Permite escolher o tamanho maximo dos motifs
`-revcomp` Permite procurar na parte + e - da fita
`-bfile` Diz a ferramenta qual é o background

```
meme interseccion_genespromotores.fasta -dna -oc interseccion_0 -mod anr -nmotifs 5 -minw 4 -maxw 20 -revcomp -bfile background_0
meme interseccion_genespromotores.fasta -dna -oc interseccion_1 -mod anr -nmotifs 5 -minw 4 -maxw 20 -revcomp -bfile background_1
meme interseccion_genespromotores.fasta -dna -oc interseccion_2 -mod anr -nmotifs 5 -minw 4 -maxw 20 -revcomp -bfile background_2
meme interseccion_genespromotores.fasta -dna -oc interseccion_3 -mod anr -nmotifs 5 -minw 4 -maxw 20 -revcomp -bfile background_3
meme Intersecção_faire.fasta -dna -oc Faire_1 -mod anr -nmotifs 5 -minw 4 -maxw 20 -objfun classic -revcomp -bfile background_1
meme Intersecção_faire.fasta -dna -oc Faire_2 -mod anr -nmotifs 5 -minw 4 -maxw 20 -objfun classic -revcomp -bfile background_2
meme Intersecção_faire.fasta -dna -oc Faire_3 -mod anr -nmotifs 5 -minw 4 -maxw 20 -objfun classic -revcomp -bfile background_3
```
  Cada um irá produzir uma pasta nova que dentro dela possuiu um arquivo HTML que abre a pagina do meme com os resultados dentro dela, dentro dela é possivel visulizar os motifs encontrados dentro das sequencias. Nesse trabalho, a ordem deixa mais confiavel a analise, mas é interessante ver os resultados das outras analises e ver a diferença entre os resultados. 
Os motifs que foram encontradors no arquivo interseccion_genespromotores.fasta com ordem 3 foram esses.

![image](https://github.com/syresbr/Genomica-e-Bioinformatica-2023-/assets/46944679/ba75035f-f65d-4ce2-afd8-54ad931a16ea)

  E no arquivo Intersecção_faire.fasta:

![image](https://github.com/syresbr/Genomica-e-Bioinformatica-2023-/assets/46944679/c939fa71-a181-4a67-a1e9-02232b0993cf)


  Depois disso nos baixamos os motifs em formato `minimal meme` para poder ser usado no mast


### MAST

  MAST procura por motifs gerados pelos MEME em um arquivo fasta para encontrar outras sequencias que compartilham esses motifs atravez do alinhamento desse motifs contra a sequencia. Aqui nós iremos utilizar ele para descobrir quais sequencias possuem esses motifs para tentar descobrir quais outros promotores podem ser ativados pelas mesmas condições que os genes que nos temos dados. Para isso usaremos o genoma inteiro como background.
```
mast -oc mast_1_intersecção Resultadosmeme/intersecção/RCGKCAKAACGTGSACATRT.meme CRE_v5.6_promoters_1500bp.fasta
mast -oc mast_2_intersecção Resultadosmeme/intersecção/YCCCGAAATCCSCCGGRMCC.meme CRE_v5.6_promoters_1500bp.fasta
mast -oc mast_3_intersecção Resultadosmeme/intersecção/YTYCTMCCCCCCCCC.meme CRE_v5.6_promoters_1500bp.fasta
mast -oc mast_1_faire Resultadosmeme/FAIRE/AYRMGCACWRWCAMWSCKKG.meme CRE_v5.6_promoters_1500bp.fasta 
mast -oc mast_2_faire Resultadosmeme/FAIRE/GKGTSCGKSGGGDGGBGGGG.meme CRE_v5.6_promoters_1500bp.fasta
```

Ele vai gerar 3 arquivos, igual ao MEME. Ao abrir o HTML vai abrir essa pagina:
![image](https://github.com/syresbr/Genomica-e-Bioinformatica-2023-/assets/46944679/1e33b71b-132d-4679-9d9a-bef90b475518)

Utilizando a ferramenta cut do linux nos arquivos .txt é possivel extrair quais os genes cada motifs é encontrado e gerar um novo arquivo com os genes de interesse. Esses arquivos vão ser os utilizados no TopGO.

### Tomtom

  Tomtom serve para avaliar um motif contra uma base de dados que possuem varios motifs conhecidos. A ideia aqui é ver se os motifs encontrados no MEME já são conhecidos ou se dentro desses motifs encontrados existem motifs menores, pois normalmente motifs são pequenos, de 4 a 10 bp, mas os que nos encontramos como o MEME são maiores, de 15 a 20 bp, e dentro deles é possivel que existam outros motifs. 

### TopGO

  TopGO é uma ferramenta do R que permite utilizar de gene ontology para entender o provavel funcionamento dos genes de interesse. Utilizamos ele para enriquecer os genes de cada resultado do MAST e tentar descobrir se eles possuem uma ligação entre si. Para isso utilizamos um script de R que permite a criação de uma tabela com as ontologias dos genes e disso criar uma plot mostrando a significancia de cada gene encontrado.
  Como por exemplo, a tabela e o plot do ![image](https://github.com/syresbr/Genomica-e-Bioinformatica-2023-/assets/46944679/44560b32-8b3c-4ff1-a8a4-027d9c2cf189)

  ![image](https://github.com/syresbr/Genomica-e-Bioinformatica-2023-/assets/46944679/95a29f70-04a8-487b-be97-f53eb995f8b1)
![mi1plot](https://github.com/syresbr/Genomica-e-Bioinformatica-2023-/assets/46944679/7260a61f-5d97-4649-80c2-f0041610772a)
