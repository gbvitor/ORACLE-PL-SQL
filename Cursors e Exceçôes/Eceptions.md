### Você pode tratar um erro com uma mensagem amigável, modificando a procedure conforme os comandos abaixo:

```sql
CREATE OR REPLACE PROCEDURE incluir_cliente
(
p_ID CLIENTE.ID%type,
p_RAZAO CLIENTE.RAZAO_SOCIAL%type,
p_CNPJ CLIENTE.CNPJ%type,
p_SEGMERCADO CLIENTE.SEGMERCADO_ID%type,
p_FATURAMENTO CLIENTE.FATURAMENTO_PREVISTO%type
)
IS
   v_CATEGORIA CLIENTE.CATEGORIA%type;
   v_CNPJ CLIENTE.CNPJ%type;
BEGIN

   v_CATEGORIA := categoria_cliente(p_FATURAMENTO);
   FORMATA_CNPJ(p_CNPJ, v_CNPJ);

   INSERT INTO CLIENTE
   VALUES
   (p_ID, p_RAZAO, v_CNPJ, p_SEGMERCADO, SYSDATE, p_FATURAMENTO, v_CATEGORIA);
   COMMIT;

EXCEPTION
   WHEN DUP_VAL_ON_INDEX THEN
      dbms_output.put_line('******************************************');
      dbms_output.put_line('*************** CLIENTE JÁ CADASTRADO !!!!');
      dbms_output.put_line('******************************************');
END;
```

### Aqui, você tratará o erro DUP_VAL_ON_INDEX que está relacionado a uma lista de exceções já mapeadas pelo Oracle. Esta, no caso, acontecerá quando o erro de chave primária ocorrer.

### Mas se você ajustar a procedure como abaixo, o erro virá pela interface nativa Oracle, com um número customizado por você:

```sql
CREATE OR REPLACE PROCEDURE incluir_cliente
(
p_ID CLIENTE.ID%type,
p_RAZAO CLIENTE.RAZAO_SOCIAL%type,
p_CNPJ CLIENTE.CNPJ%type,
p_SEGMERCADO CLIENTE.SEGMERCADO_ID%type,
p_FATURAMENTO CLIENTE.FATURAMENTO_PREVISTO%type
)
IS
   v_CATEGORIA CLIENTE.CATEGORIA%type;
   v_CNPJ CLIENTE.CNPJ%type;
BEGIN

   v_CATEGORIA := categoria_cliente(p_FATURAMENTO);
   FORMATA_CNPJ(p_CNPJ, v_CNPJ);

   INSERT INTO CLIENTE
   VALUES
   (p_ID, p_RAZAO, v_CNPJ, p_SEGMERCADO, SYSDATE, p_FATURAMENTO, v_CATEGORIA);
   COMMIT;

EXCEPTION
   WHEN DUP_VAL_ON_INDEX THEN
      raise_application_error(-20010,'CLIENTE JÁ CADASTRADO !!!!');
END;
```

### Nem sempre todos os erros Oracle estão mapeados nas exceções nominadas. Mas você pode associar um erro Oracle usando seu número interno e direcioná-lo para um texto customizado:

```sql
CREATE OR REPLACE PROCEDURE incluir_cliente
(
p_ID CLIENTE.ID%type,
p_RAZAO CLIENTE.RAZAO_SOCIAL%type,
p_CNPJ CLIENTE.CNPJ%type,
p_SEGMERCADO CLIENTE.SEGMERCADO_ID%type,
p_FATURAMENTO CLIENTE.FATURAMENTO_PREVISTO%type
)
IS
   v_CATEGORIA CLIENTE.CATEGORIA%type;
   v_CNPJ CLIENTE.CNPJ%type;
   e_IDNULO exception;
   pragma exception_init(e_IDNULO,-1400);
BEGIN

   v_CATEGORIA := categoria_cliente(p_FATURAMENTO);
   FORMATA_CNPJ(p_CNPJ, v_CNPJ);

   INSERT INTO CLIENTE
   VALUES
   (p_ID, p_RAZAO, v_CNPJ, p_SEGMERCADO, SYSDATE, p_FATURAMENTO, v_CATEGORIA);
   COMMIT;

EXCEPTION
   WHEN DUP_VAL_ON_INDEX THEN
      raise_application_error(-20010,'CLIENTE JÁ CADASTRADO !!!!');
   WHEN e_IDNULO THEN
      raise_application_error(-20015,'IDENTIFICADOR DO CLIENTE NULO !!!!');
END;
```

### Para evitar de ter que mapear todos os erros que possam ocorrer, podemos criar um erro genérico, como no script abaixo:

```sql
CREATE OR REPLACE PROCEDURE incluir_cliente
(
p_ID CLIENTE.ID%type,
p_RAZAO CLIENTE.RAZAO_SOCIAL%type,
p_CNPJ CLIENTE.CNPJ%type,
p_SEGMERCADO CLIENTE.SEGMERCADO_ID%type,
p_FATURAMENTO CLIENTE.FATURAMENTO_PREVISTO%type
)
IS
   v_CATEGORIA CLIENTE.CATEGORIA%type;
   v_CNPJ CLIENTE.CNPJ%type;
   e_IDNULO exception;
   pragma exception_init(e_IDNULO,-1400);
BEGIN

   v_CATEGORIA := categoria_cliente(p_FATURAMENTO);
   FORMATA_CNPJ(p_CNPJ, v_CNPJ);

   INSERT INTO CLIENTE
   VALUES
   (p_ID, p_RAZAO, v_CNPJ, p_SEGMERCADO, SYSDATE, p_FATURAMENTO, v_CATEGORIA);
   COMMIT;

EXCEPTION
   WHEN DUP_VAL_ON_INDEX THEN
      raise_application_error(-20010,'CLIENTE JÁ CADASTRADO !!!!');
   WHEN e_IDNULO THEN
      raise_application_error(-20015,'IDENTIFICADOR DO CLIENTE NULO !!!!');
   WHEN others THEN
      raise_application_error(-20020,'ERRO NÃO ESPERADO !!!!');
END;
```

### Mas é possível exibir o texto original do Oracle quando ocorrer um erro não mapeado. Para isso faça:

```sql
CREATE OR REPLACE PROCEDURE incluir_cliente
(
p_ID CLIENTE.ID%type,
p_RAZAO CLIENTE.RAZAO_SOCIAL%type,
p_CNPJ CLIENTE.CNPJ%type,
p_SEGMERCADO CLIENTE.SEGMERCADO_ID%type,
p_FATURAMENTO CLIENTE.FATURAMENTO_PREVISTO%type
)
IS
   v_CATEGORIA CLIENTE.CATEGORIA%type;
   v_CNPJ CLIENTE.CNPJ%type;
   e_IDNULO exception;
   pragma exception_init(e_IDNULO,-1400);
BEGIN

   v_CATEGORIA := categoria_cliente(p_FATURAMENTO);
   FORMATA_CNPJ(p_CNPJ, v_CNPJ);

   INSERT INTO CLIENTE
   VALUES
   (p_ID, p_RAZAO, v_CNPJ, p_SEGMERCADO, SYSDATE, p_FATURAMENTO, v_CATEGORIA);
   COMMIT;

EXCEPTION
   WHEN DUP_VAL_ON_INDEX THEN
      raise_application_error(-20010,'CLIENTE JÁ CADASTRADO !!!!');
   WHEN e_IDNULO THEN
      raise_application_error(-20015,'IDENTIFICADOR DO CLIENTE NULO !!!!');
   WHEN others THEN
      raise_application_error(-20020,'ERRO NÃO ESPERADO !!!! - TEXTO ORIGINAL DO ERRO: ' || sqlerrm());
END;
```

### Muitas vezes, um erro cometido pelo usuário não é considerado um erro para o Oracle. Estes erros de negócio também podem ser customizados:

```sql
CREATE OR REPLACE PROCEDURE incluir_cliente
(
p_ID CLIENTE.ID%type,
p_RAZAO CLIENTE.RAZAO_SOCIAL%type,
p_CNPJ CLIENTE.CNPJ%type,
p_SEGMERCADO CLIENTE.SEGMERCADO_ID%type,
p_FATURAMENTO CLIENTE.FATURAMENTO_PREVISTO%type
)
IS
   v_CATEGORIA CLIENTE.CATEGORIA%type;
   v_CNPJ CLIENTE.CNPJ%type;
   e_IDNULO exception;
   pragma exception_init(e_IDNULO,-1400);
   e_FATURAMENTO_NULO exception;
BEGIN

   IF p_FATURAMENTO IS NULL THEN
      RAISE e_FATURAMENTO_NULO;
   END IF;

   v_CATEGORIA := categoria_cliente(p_FATURAMENTO);
   FORMATA_CNPJ(p_CNPJ, v_CNPJ);

   INSERT INTO CLIENTE
   VALUES
   (p_ID, p_RAZAO, v_CNPJ, p_SEGMERCADO, SYSDATE, p_FATURAMENTO, v_CATEGORIA);
   COMMIT;

EXCEPTION
   WHEN DUP_VAL_ON_INDEX THEN
      raise_application_error(-20010,'CLIENTE JÁ CADASTRADO !!!!');
   WHEN e_IDNULO THEN
      raise_application_error(-20015,'IDENTIFICADOR DO CLIENTE NULO !!!!');
   WHEN e_FATURAMENTO_NULO THEN
      raise_application_error(-20025,'FATURAMENTO FOI INCLUIDO COM VALOR NULO !!!!');
   WHEN others THEN
      raise_application_error(-20020,'ERRO NÃO ESPERADO !!!! - TEXTO ORIGINAL DO ERRO: ' || sqlerrm());
END;
```
