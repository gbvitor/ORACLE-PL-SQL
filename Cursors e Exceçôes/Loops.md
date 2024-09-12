### O objetivo agora é atualizar todos os segmentos de mercados de todos os clientes, é possível ultilizando um looping.

```sql
--usando a estrutura LOOP ... END LOOP:
DECLARE
   v_SEGMERCADO CLIENTE.SEGMERCADO_ID%type := 3;
   v_ID CLIENTE.ID%type := 1;
BEGIN
   LOOP
      ATUALIZAR_SEGMERCADO (v_ID,v_SEGMERCADO);
      v_ID := v_ID + 1;
   EXIT WHEN v_ID > 9;
   END LOOP;
END;
```
