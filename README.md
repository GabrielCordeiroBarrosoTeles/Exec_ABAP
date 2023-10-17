# Execicio ABAP

REPORT zprog_007_05.  " Alteração do nome do relatório com título e parâmetros.
LINE-SIZE 70 LINE-COUNT 5.

TABLES: zprodutos_05,  " Referência para a tabela z de produtos
        zestoque_05,   " Referência para a tabela z de estoques
        zvendas_05.    " Referência para a tabela z de vendas

" Tipo para estoques
TYPES: BEGIN OF ty_estoque,
         produto    TYPE zproduto_05,
         quantidade TYPE zquantidade_05,
         unidade    TYPE zunidade_05,
       END OF ty_estoque.

"Estruturas e tabelas internas
DATA: lt_estoque TYPE TABLE OF ty_estoque,    " Tabela interna para estoque
      ls_estoque LIKE LINE OF lt_estoque,     " Linha de tabela interna
      ls_produto TYPE zprodutos_e_05,         " Estrutura para produtos
      lt_produto TYPE TABLE OF zprodutos_e_05, " Tabela interna para produtos
      lt_vendas  TYPE TABLE OF zvendas_05,     " Tabela interna para vendas
      ls_venda   LIKE LINE OF lt_vendas.       " Linha de tabela interna

DATA: lv_maior  TYPE zquantidade_05,  " Variável para armazenar a maior quantidade
      lv_vlrtot TYPE zpreco_05,      " Variável para armazenar o valor total
      lv_maxidx TYPE sy-tabix.      " Variável para armazenar o índice máximo

SELECTION-SCREEN BEGIN OF BLOCK b1 WITH FRAME TITLE TEXT-001.

SELECT-OPTIONS: s_prod FOR zprodutos_05-produto OBLIGATORY, " Opção de seleção para produtos
                s_prec FOR zprodutos_05-preco.             " Opção de seleção para preços

SELECTION-SCREEN END OF BLOCK b1.

START-OF-SELECTION.
  " Seleciona dados da tabela de produtos para a tabela interna lt_produto
  SELECT produto
         desc_produto
         preco
    FROM zprodutos_05
    INTO CORRESPONDING FIELDS OF TABLE lt_produto
    WHERE produto IN s_prod
      AND preco IN s_prec.

  " Seleciona dados da tabela de estoques para a tabela interna lt_estoque
  SELECT produto
         quantidade
         unidade
    FROM zestoque_05
    INTO TABLE lt_estoque
    FOR ALL ENTRIES IN lt_produto
    WHERE produto = lt_produto-produto.

  " Seleciona dados da tabela de vendas para a tabela interna lt_vendas
  SELECT venda
         item
         produto
         quantidade
         preco
    FROM zvendas_05
    INTO CORRESPONDING FIELDS OF TABLE lt_vendas
     FOR ALL ENTRIES IN lt_produto
    WHERE produto = lt_produto-produto.

END-OF-SELECTION.

  LOOP AT lt_produto INTO ls_produto.

    " Busca o produto na tabela de estoque
    READ TABLE lt_estoque INTO ls_estoque WITH KEY produto = ls_produto-produto.
    IF sy-subrc <> 0.
      ls_estoque-quantidade = 0.  " Define quantidade como zero se não encontrado no estoque
    ENDIF.

    " Calcula o valor total do estoque
    lv_vlrtot = ls_produto-preco * ls_estoque-quantidade.

    " Busca a maior venda do produto
    CLEAR: ls_venda, lv_maior, lv_maxidx.
    LOOP AT lt_vendas INTO ls_venda WHERE produto = ls_produto-produto.
      IF ls_venda-quantidade > lv_maior.
        lv_maior  = ls_venda-quantidade.
        lv_maxidx = sy-tabix.
      ENDIF.
    ENDLOOP.

    " Exibe informações na tela
    WRITE: / ls_produto-produto, ls_produto-desc_produto, ls_produto-preco, ls_estoque-quantidade, lv_vlrtot.
    IF ls_venda-produto = ''.
      WRITE: '-- Nenhuma venda registrada --'.
    ELSE.
      READ TABLE lt_vendas INTO ls_venda INDEX lv_maxidx.
      WRITE: ls_venda-quantidade, ls_venda-preco, ls_venda-data, ls_venda-hora.
    ENDIF.

  ENDLOOP.
