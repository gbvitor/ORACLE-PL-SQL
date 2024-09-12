```sql
SET SERVEROUTPUT ON;
```

Explicação do comando acima
O comando SET SERVEROUTPUT ON; é usado em ferramentas de linha de comando como SQL\*Plus e SQLcl, e em alguns outros ambientes de execução de PL/SQL. Aqui está o que ele faz:

-  Ativa a Saída de Servidor: Quando você ativa a saída do servidor, isso permite que as mensagens que você gera usando DBMS_OUTPUT.PUT_LINE em seu código PL/SQL sejam exibidas na tela.

#### Contexto e Uso

No ambiente Oracle, você pode usar o pacote DBMS_OUTPUT para enviar mensagens para a saída do console. Isso é útil para depuração e rastreamento do que seu código PL/SQL está fazendo. Por padrão, a saída do servidor pode estar desativada, então você precisa usar SET SERVEROUTPUT ON; para vê-la.

### Bloco de código anônimo em PL/SQL

```SQL
--Programa para cria v_DESCRICAO
DECLARE
   v_ID SEGMERCADO.ID%type := 1;
   v_IDSaida SEGMERCADO.ID%type;
   v_DESCRICAO SEGMERCADO.DESCRICAO%type;

BEGIN
   SELECT DESCRICAO INTO v_DESCRICAO FROM SEGMERCADO WHERE ID = v_ID;
   SELECT ID INTO v_IDSaida FROM SEGMERCADO WHERE ID = v_ID;
   dbms_output.put_line(v_DESCRICAO);
   dbms_output.put_line(v_IDSaida);

END;
```

### Criação de uma função para obter descrição do segmento de mercado através do recebimento do ID pelo parâmetro da função.

```sql
CREATE OR REPLACE FUNCTION obter_descricao_segmercado
(p_ID IN SEGMERCADO.ID%type)
RETURN SEGMERCADO.DESCRICAO%type
IS
   v_DESCRICAO SEGMERCADO.DESCRICAO%type;
BEGIN
   SELECT DESCRICAO INTO v_DESCRICAO FROM SEGMERCADO WHERE ID = p_ID;
   RETURN v_DESCRICAO;
END;

--Mostra o conteudo ultilizando a função criada
SELECT ID, obter_descricao_segmercado(ID), DESCRICAO, LOWER(DESCRICAO) FROM SEGMERCADO;
```
