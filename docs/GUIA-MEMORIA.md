# Guia: memória, limites e como medir

Regras de consumo de memória que valem para qualquer pacote da plataforma, e as
armadilhas de medição que fazem uma campanha de redução chegar à conclusão errada.

## O que medir

**Não compare fases de uma campanha pelo total do runtime de container.** Esse
número inclui *page cache*, que é recuperável e não é custo do serviço — ele
oscila em mais de um gigabyte conforme o container acabou ou não de compilar algo,
sem que nada tenha mudado nos processos.

Para comparar duas medições, use a memória **anônima** de cada processo. O total
do runtime serve para olhar um container agora, não para comparar.

**Mas medir só o heap também engana — e engana para menos.** Ele torna invisível
a memória de arquivo mapeado que não é compartilhada com ninguém. Meça
`Private_Clean` junto: um valor alto e **igual** em vários containers é a
assinatura de conteúdo duplicado — o mesmo binário presente N vezes, em N inodes,
porque as imagens deixaram de compartilhar a camada que o continha. O mesmo
número aparecendo em `Shared_Clean` é o oposto: conteúdo compartilhado, que custa
uma vez só.

Mais duas regras de método:

- **Memória residente estável esconde pico.** Um trabalho que aloca muito e
  devolve tudo termina com residência baixa e pico alto. Dentro de um limite de
  cgroup, **é o pico que mata** — meça-o explicitamente.
- **Some a subárvore de processos, não o processo inicial.** Quem cria processos
  filhos só é medido corretamente somando todos. Medir apenas o processo que o
  runtime reporta pode dar dois megabytes para um serviço que consome cento e
  trinta.

Meça sempre com o ambiente **em repouso**: um serviço recém-provisionado ainda
está compilando, e medi-lo nesse instante mede a compilação, não o serviço.

## O teto do V8

**Dentro de um container, o Node respeita o cgroup.** Ele dimensiona o heap a
partir da cota, sem nenhuma flag. Para apertar um serviço, **baixe a cota** — o
V8 se reajusta sozinho.

Três consequências que custam tempo:

1. **Baixar a cota a quente não reduz o consumo.** O dimensionamento acontece no
   arranque; mudar a cota de um processo vivo muda o teto, não o uso.
2. **Variável de ambiente não resolve.** Um executável empacotado como binário
   **ignora `NODE_OPTIONS`**. Um teto explícito só entra na construção do binário.
3. **Fora do container o descasamento é real.** Um processo no hospedeiro enxerga
   a RAM da máquina inteira, não a fatia que se pretendia dar a ele.

## O V8 não devolve o pico ao sistema

Um pico de memória — tipicamente uma compilação — **fica retido** no processo
depois de terminar. Soltar a referência interrompe o crescimento, mas só o
processo terminando devolve a memória ao sistema operacional.

A consequência de projeto: **trabalho pesado e de vida curta não deve rodar no
mesmo processo que serve por horas**. Um processo que compila e depois atende
carrega o pico da compilação pelo resto da vida. Mover a compilação para um
processo filho, que morre ao terminar, elimina o resíduo — e é o que separa um
serviço com interface de um sem interface no consumo final.

Isso também impõe uma ordem: enquanto a compilação acontece **dentro** do
processo, a cota não pode descer, porque o pico precisa caber. A cota só cai de
verdade depois que o trabalho pesado sai.

## Carregar o que não se usa

Um registro que carrega **todos** os componentes disponíveis no arranque faz cada
processo pagar por tudo que existe no ecossistema, inclusive o que ele nunca vai
instanciar. Carregamento sob demanda é o padrão: um serviço sem interface não deve
pagar o custo de quem monta interface.

## Crescimento contínuo é outro problema

Um processo que cresce de forma constante — e não em degraus — não se resolve
apertando o teto: o teto troca crescimento lento por queda abrupta. Antes de
limitar, capture um retrato do heap e descubra o que está sendo retido.

Ver também: [Guia: Como o Código Chega ao Ar](./GUIA-PROVISIONAMENTO.md).
