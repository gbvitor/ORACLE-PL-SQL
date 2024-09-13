### Execute o comando abaixo para incluir um novo cliente:

```sql
EXECUTE INCLUIR_CLIENTE(30, 'Loja NNM','56854960854906',2,60000);
```

### Você verá que, após esta inclusão, se executarmos o programa PL/SQL abaixo para atualizar os segmentos, ela não irá funcionar:

```sql
DECLARE
   v_SEGMERCADO CLIENTE.SEGMERCADO_ID%type := 1;
   v_NUMCLI INTEGER;
BEGIN
   SELECT COUNT(*) INTO v_NUMCLI FROM CLIENTE;
   FOR v_ID IN 1..v_NUMCLI LOOP
      ATUALIZAR_SEGMERCADO (v_ID,v_SEGMERCADO);
   END LOOP;
END;
```

### Isso porque os identificadores dos clientes não respeitam uma ordem sequencial. A solução para esse problema é usar um CURSOR.

### O Que é um Cursor?

Um cursor é um mecanismo que proporciona controle sobre o conjunto de registros retornado por uma consulta SQL. Ele permite:

Iterar Sobre Resultados: Navegar linha por linha nos resultados de uma consulta.
Manusear Dados: Ler e processar dados um de cada vez, o que é útil quando você precisa realizar operações complexas em cada linha.
Tipos de Cursors

-  Cursors Explícitos:

Definidos pelo desenvolvedor e oferecem controle detalhado sobre o processo de recuperação e manipulação dos dados.
São declarados explicitamente no código PL/SQL e precisam ser abertos, buscados e fechados manualmente.

-  Cursors Implícitos:

Criados automaticamente pelo Oracle para operações SQL DML (INSERT, UPDATE, DELETE) e são gerenciados pelo sistema.
Não requerem declaração ou controle explícito pelo desenvolvedor.

```sql
--Execute os comandos abaixo para ver como funciona a estrutura de CURSOR:

SET SERVEROUTPUT ON;
DECLARE
   v_ID CLIENTE.ID%type;
   v_RAZAO CLIENTE.RAZAO_SOCIAL%type;
   CURSOR cur_CLIENTE IS SELECT ID, RAZAO_SOCIAL FROM CLIENTE;
BEGIN
   OPEN cur_CLIENTE;
   LOOP
      FETCH cur_CLIENTE INTO v_ID, v_RAZAO;
   EXIT WHEN cur_CLIENTE%NOTFOUND;
      dbms_output.put_line('ID = ' || v_ID || ', RAZAO = ' || v_RAZAO);
   END LOOP;
END;
```

### Edite a procedure de atualização dos segmentos para levar em consideração o uso do CURSOR e o problema de atualização ser solucionado quando os identificadores dos clientes não respeitarem uma ordem sequencial:

```sql
DECLARE
   v_SEGMERCADO CLIENTE.SEGMERCADO_ID%type := 3;
   v_ID CLIENTE.ID%type;
   CURSOR cur_CLIENTE IS SELECT ID FROM CLIENTE;
BEGIN
   OPEN cur_CLIENTE;
   LOOP
      FETCH cur_CLIENTE INTO v_ID;
   EXIT WHEN cur_CLIENTE%NOTFOUND;
      ATUALIZAR_SEGMERCADO(v_ID, v_SEGMERCADO);
   END LOOP;
   CLOSE cur_CLIENTE;
END;
```
