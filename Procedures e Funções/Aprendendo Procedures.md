#

-  ### Criação de uma procedure nomeada incluir_segmercado que insere na tabela de segmento de mercado (SEGMERCADO) os valore (p_ID e p_DESCRICAO), os valores são recebidos pelos parâmetros da procedure.

```bash
CREATE OR REPLACE PROCEDURE incluir_segmercado
(p_ID IN NUMBER, p_DESCRICAO IN VARCHAR2)
IS
BEGIN
   INSERT INTO SEGMERCADO (ID, DESCRICAO) VALUES (p_ID, UPPER(p_DESCRICAO));
   COMMIT;
END;
```

-  ### Executa adicionando valores através dos parametros da procedure

```bash
EXECUTE incluir_segmercado(4,'Farmaceuticos');
```

-  ### Forma de alterar a procedure para que ela receba diretamente nos parâmetros os tipos dos dados que serão recebidos (%TYPE)

```bash
CREATE OR REPLACE PROCEDURE incluir_segmercado
(p_ID IN SEGMERCADO.ID%type, p_DESCRICAO IN SEGMERCADO.DESCRICAO%type)
IS
BEGIN
   INSERT INTO SEGMERCADO (ID, DESCRICAO) VALUES (p_ID, UPPER(p_DESCRICAO));
   COMMIT;
END;
```

-  ### Elimina a procedure desejada

```bash
DROP PROCEDURE incluir_segmercado;
```
