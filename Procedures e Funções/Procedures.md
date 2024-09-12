### Criação de uma procedure nomeada incluir_segmercado que insere na tabela de segmento de mercado (SEGMERCADO) os valores (p_ID e p_DESCRICAO), os valores são recebidos pelos parâmetros da procedure.

```sql
CREATE OR REPLACE PROCEDURE incluir_segmercado
(p_ID IN NUMBER, p_DESCRICAO IN VARCHAR2)
IS
BEGIN
   INSERT INTO SEGMERCADO (ID, DESCRICAO) VALUES (p_ID, UPPER(p_DESCRICAO));
   COMMIT;
END;
```

### Executa a procedure adicionando valores através dos parâmetros da procedure.

```sql
EXECUTE incluir_segmercado(4,'Farmaceuticos');
```

### Forma de alterar a procedure para receber diretamente nos parâmetros os tipos dos dados que serão recebidos (%TYPE).

```sql
CREATE OR REPLACE PROCEDURE incluir_segmercado
(p_ID IN SEGMERCADO.ID%type, p_DESCRICAO IN SEGMERCADO.DESCRICAO%type)
IS
BEGIN
   INSERT INTO SEGMERCADO (ID, DESCRICAO) VALUES (p_ID, UPPER(p_DESCRICAO));
   COMMIT;
END;
```

### Elimina a procedure desejada.

```sql
DROP PROCEDURE incluir_segmercado;
```
