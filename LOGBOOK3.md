# Trabalho realizado na Semana #3

## Identificação

* Heap Buffer Overflow encontrado no Blink engine 
Um heap overflow é um buffer overflow(quando um programa "acede fora" da memória alocada), ocorrida na heap.
* Sistema operativo: Linux
* As aplicações afetadas por esta CVE são browsers que utlizam o Blink Engine que se encontram nos que são baseados em Chromium Open Source Software, Chrome Version: 83.0.4103.97 stable.


## Catalogação

* Quando: Reportada no dia 15 de Outubro de 2020
* Quem: Reportada por Liang Dong
* Nível de gravidade: Tem um CVSS score de 6.8, significando que
	apresenta uma vulnerabilidade de gravidade média


## Exploit

* Não é necessario autenticação para a utilização deste exploit
* Permitia um atacante remoto explorar "heap corruption" via página HTML
* Causava um heap buffer overflow in Blink no Google Chrome

## Ataques

* Não foram reportados ataques bem-sucedidos utilizando esta vulnerabilidade.
