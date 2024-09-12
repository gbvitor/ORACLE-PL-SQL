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

### No código acima, é preciso conhecer, de antemão, o número de clientes existentes na tabela, neste momento. Mas isso pode ser feito pelo próprio programa PL/SQL:

```sql
DECLARE
   v_SEGMERCADO CLIENTE.SEGMERCADO_ID%type := 3;
   v_ID CLIENTE.ID%type := 1;
   v_NUMCLI INTEGER;
BEGIN
   SELECT COUNT(*) INTO v_NUMCLI FROM CLIENTE;
   LOOP
      ATUALIZAR_SEGMERCADO (v_ID,v_SEGMERCADO);
      v_ID := v_ID + 1;
   EXIT WHEN v_ID > v_NUMCLI;
   END LOOP;
END;
```

### Não temos à disposição apenas o comando de repetição LOOP ... END LOOP. Essa repetição pode ser feita com a estrutura de repetição FOR. Inclusive, é a mais apropriada quando temos o número de interações previamente definido:

```sql
DECLARE
   v_SEGMERCADO CLIENTE.SEGMERCADO_ID%type := 4;
   v_NUMCLI INTEGER;
BEGIN
   SELECT COUNT(*) INTO v_NUMCLI FROM CLIENTE;
   FOR v_ID IN 1..v_NUMCLI LOOP
      ATUALIZAR_SEGMERCADO (v_ID,v_SEGMERCADO);
   END LOOP;
END;
```

```sql
--É possível passar os parâmetros de forma nomeada para a procedure:
DECLARE
   v_SEGMERCADO CLIENTE.SEGMERCADO_ID%type := 1;
   v_NUMCLI INTEGER;
BEGIN
   SELECT COUNT(*) INTO v_NUMCLI FROM CLIENTE;
   FOR v_ID IN 1..v_NUMCLI LOOP
      ATUALIZAR_SEGMERCADO ( p_SEGMERCADO_ID => v_SEGMERCADO, p_ID => v_ID);
   END LOOP;
END;
```

### O comando de saída do LOOP ... END LOOP pode ser chamado em qualquer linha do bloco de comandos do programa PL/SQL, desde que dentro da estrutura de repetição:

```sql
DECLARE
   v_SEGMERCADO CLIENTE.SEGMERCADO_ID%type := 2;
   v_ID CLIENTE.ID%type := 1;
   v_NUMCLI INTEGER;
BEGIN
   SELECT COUNT(*) INTO v_NUMCLI FROM CLIENTE;
   LOOP
      IF v_ID <= v_NUMCLI THEN
         ATUALIZAR_SEGMERCADO (v_ID,v_SEGMERCADO);
         v_ID := v_ID + 1;
      ELSE
         EXIT;
      END IF;
   END LOOP;
END;
```

### Outra estrutura de repetição o WHILE ... LOOP:

```sql
DECLARE
   v_SEGMERCADO CLIENTE.SEGMERCADO_ID%type := 3;
   v_ID CLIENTE.ID%type := 1;
   v_NUMCLI INTEGER;
BEGIN
   SELECT COUNT(*) INTO v_NUMCLI FROM CLIENTE;
   WHILE v_ID <= v_NUMCLI LOOP
      ATUALIZAR_SEGMERCADO (v_ID,v_SEGMERCADO);
      v_ID := v_ID + 1;
   END LOOP;
END;
```
