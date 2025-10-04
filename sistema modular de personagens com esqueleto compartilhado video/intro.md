## importa o modelo base

- no lado direito no menu tem `data` onde mostra todos os ossos do modelo

- ### metodo 1

  - file `>` import `>` tipo
  - tanto o modelo base quando as partes tem que estar no ponto `0,0,0` nos eixos
  - se estiver fora do ponto `0,0,0` `alt + G` se estiver rotacionado `alt + R`
  - agora aperte `TAB` ou no  `edit mode`
  - no `edit mode` mova o objeto, posicione onde quer, a pos posicionar volte para o `object mode`, perceba que o objeto esta posicionado mas com o ponto pivô no centro (`o,o,o`)

  - agora seleciona o novo objeto e com `shift` pressionado selecione o osso, aperte `ctrl + p`, para anexar a malha ao esqueleto, selecione `with Empty Groups`: gropos vazios sem pesos

  - mas caso mova a cabeca  como nao a influencia dos ossos, ele permanece  imóvel, agora vamos ver a melhor parte: como esse objeto ex:`capacete, acessorio` nao deve se deformar como uma roupa etc.., vou anexar a influencia somente de uma parte do corpo, ja que a malha ja esta anexada, como saber: va em `modifier propets` vai ter o menu `armature` com objeto `root`

  - vamos para o `edit mode` procuramos em `data` o osso da cabeca `obs(`objeto selected`)`, seleciona o osso da cabeca e em `weight` seleciona o maior valor, finaliza com `assing`