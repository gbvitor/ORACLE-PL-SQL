```sql
-- Crie uma conexão usando o user_dev.

-- Na janela de script, escreva:

CREATE OR REPLACE PACKAGE EXCEPTION_PAC
IS
    e_NULL EXCEPTION;
    pragma exception_init(e_NULL, -1400);
    e_FK EXCEPTION;
    pragma exception_init(e_FK, -2291);
END;
```

```sql
-- Execute as inserções, forçando os erros:

INSERT INTO CLIENTE VALUES (NULL, 'TESTE','22222',2,SYSDATE,10000,'AAA');
INSERT INTO CLIENTE VALUES (30, 'TESTE','22222',10,SYSDATE,10000,'AAA');
```

```sql
-- Mostre a tabela SEGMERCADO:

SELECT * FROM SEGMERCADO;
```

```sql
-- Abra o código do pacote e inclua a exceção de erro:

CREATE OR REPLACE NONEDITIONABLE PACKAGE BODY CLIENTE_PAC
IS
FUNCTION VERIFICA_SEGMENTO_MERCADO
    (p_id in segmercado.id%type)
        RETURN boolean
IS
    v_dummy number(1);
BEGIN
    SELECT 1 into v_dummy
        FROM segmercado
        WHERE id = p_id;
    RETURN true;
EXCEPTION
    WHEN no_data_found then
        RETURN false;
END;

FUNCTION OBTER_CATEGORIA_CLIENTE
    (p_faturamento_previsto IN cliente.faturamento_previsto%type)
    RETURN cliente.categoria%type
IS
BEGIN
    IF p_faturamento_previsto <= 10000 THEN
        RETURN 'PEQUENO';
    ELSIF p_faturamento_previsto <= 50000 THEN
        RETURN 'MEDIO';
    ELSIF p_faturamento_previsto <= 100000  THEN
        RETURN 'MEDIO GRANDE';
    ELSE
        RETURN 'GRANDE';
    END IF;
END;

PROCEDURE FORMAT_CNPJ
    (p_cnpj IN OUT varchar2)
IS
BEGIN
p_cnpj := substr(p_cnpj,1,2) ||'/'|| substr(p_cnpj,3);
    DBMS_OUTPUT.PUT_LINE('CHAMEI A ROTINA FORMAT_CNPJ DO PACOTE !!!!');
END;

PROCEDURE INCLUIR_CLIENTE
    (p_id in cliente.id%type,
    p_razao_social in cliente.razao_social%type,
    p_CNPJ cliente.CNPJ%type ,
    p_segmercado_id cliente.segmercado_id%type,
    p_faturamento_previsto cliente.faturamento_previsto%type)
IS
    v_categoria cliente.categoria%type;
    v_CNPJ cliente.cnpj%type := p_CNPJ;
    v_codigo_erro number(5);
    v_mensagem_erro varchar2(200);
    v_dummy number;
    v_verifica_segmento boolean;
    e_segmento exception;
BEGIN
    v_verifica_segmento :=     verifica_segmento_mercado(p_segmercado_id);
    IF v_verifica_segmento = false THEN
        RAISE e_segmento;
    END IF;
    v_categoria := obter_categoria_cliente(p_faturamento_previsto);
    format_cnpj (v_cnpj);
    INSERT INTO cliente
        VALUES (p_id, UPPER(p_razao_social), v_CNPJ, p_segmercado_id
            ,SYSDATE, p_faturamento_previsto, v_categoria);
    COMMIT;
EXCEPTION
    WHEN dup_val_on_index then
        raise_application_error(-20010,'Cliente já cadastrado');
    WHEN e_segmento then
        raise_application_error (-20011,'Segmento de mercado inexistente');
    WHEN EXCEPTION_PAC.e_NULL then
        raise_application_error (-20020,'Preenchimento do valor do campo é obrigatório. Favor incluir este valor !!!');
    WHEN OTHERS then
        v_codigo_erro := sqlcode;
        v_mensagem_erro := sqlerrm;
        raise_application_error (-20000,to_char(v_codigo_erro)||v_mensagem_erro);
END;

PROCEDURE ATUALIZAR_CLI_SEG_MERCADO
    (p_id cliente.id%type,
    p_segmercado_id cliente.segmercado_id%type)
IS
        e_fk exception;
        pragma exception_init(e_fk, -2291);
        e_no_update exception;
BEGIN
    UPDATE cliente
        SET segmercado_id = p_segmercado_id
        WHERE id = p_id;
    IF sql%NOTFOUND then
        RAISE e_no_update;
    END IF;
    COMMIT;
EXCEPTION
    WHEN e_fk then
        RAISE_APPLICATION_ERROR (-20001,'Segmento de Mercado Inexistente');
    WHEN e_no_update then
        RAISE_APPLICATION_ERROR (-20002,'Cliente Inexistente');
END;

PROCEDURE ATUALIZAR_FATURAMENTO_PREVISTO
    (p_id in cliente.id%type,
    p_faturamento_previsto in cliente.faturamento_previsto%type)
IS
    v_categoria cliente.categoria%type;
    e_error_id exception;
BEGIN
    v_categoria := obter_categoria_cliente(p_faturamento_previsto);
    UPDATE cliente
        SET categoria = v_categoria,
            faturamento_previsto = p_faturamento_previsto
        WHERE id = p_id;
    IF sql%NOTFOUND THEN
        RAISE e_error_id;
    END IF;
    COMMIT;
EXCEPTION
    WHEN e_error_id then
        raise_application_error(-20010,'Cliente inexistente');
END;

PROCEDURE EXCLUIR_CLIENTE
    (p_id in cliente.id%type)
IS
    e_error_id exception;
BEGIN
    DELETE FROM cliente
        WHERE id = p_id;
    IF sql%NOTFOUND THEN
        RAISE e_error_id;
    END IF;
    COMMIT;
EXCEPTION
    WHEN e_error_id then
        raise_application_error(-20010,'Cliente inexistente');
END;

PROCEDURE INCLUIR_CLIENTE
    (p_id in cliente.id%type,
    p_razao_social in cliente.razao_social%type,
    p_segmercado_id cliente.segmercado_id%type)
IS
    v_codigo_erro number(5);
    v_mensagem_erro varchar2(200);
    v_dummy number;
    v_verifica_segmento boolean;
    e_segmento exception;
BEGIN
    v_verifica_segmento :=     verifica_segmento_mercado(p_segmercado_id);
    IF v_verifica_segmento = false THEN
        RAISE e_segmento;
    END IF;
    INSERT INTO cliente (ID, RAZAO_SOCIAL, SEGMERCADO_ID, DATA_INCLUSAO)
        VALUES (p_id, UPPER(p_razao_social), p_segmercado_id
            ,SYSDATE);
    COMMIT;
EXCEPTION
    WHEN dup_val_on_index then
        raise_application_error(-20010,'Cliente já cadastrado');
    WHEN e_segmento then
        raise_application_error (-20011,'Segmento de mercado inexistente');
    WHEN EXCEPTION_PAC.e_NULL then
        raise_application_error (-20020,'Preenchimento do valor do campo é obrigatório. Favor incluir este valor !!!');
    WHEN OTHERS then
        v_codigo_erro := sqlcode;
        v_mensagem_erro := sqlerrm;
        raise_application_error (-20000,to_char(v_codigo_erro)||v_mensagem_erro);
END;

END;
```

```sql
-- Execute o pacote como user_app:

EXECUTE APP_INCLUIR_CLIENTE (NULL, 'INCLUIDO POR USER_APP PARA TESTAR VALOR NULO','12222',1,100);

-- Foi apresentado o erro customizado.
```

```sql
-- Execute o código para criar a procedure:

CREATE OR REPLACE PROCEDURE INCLUIR_SEGMENTO
(p_ID SEGMERCADO.ID%type, p_DESCRICAO SEGMERCADO.DESCRICAO%type)
IS
BEGIN
    INSERT INTO SEGMERCADO(ID, DESCRICAO) VALUES (p_ID, p_DESCRICAO);
END;
```

```sql
-- Dê o GRANT e crie o sinônimo:

GRANT SELECT, INSERT ON SEGMERCADO TO user_app;

CREATE PUBLIC SYNONYM SEGMERCADO FOR user_dev.SEGMERCADO;
```

```sql
-- Para incluir um novo segmento de mercado, execute:

EXECUTE INCLUIR_SEGMENTO (7, 'COMERCIAL');
```

```sql
-- Mostre SEGMERCADO:

SELECT * FROM SEGMERCADO;

```

```sql
-- Inclua um segmento com NULL:

EXECUTE INCLUIR_SEGMENTO (NULL, 'TRANSPORTES');
```

```sql
-- Aparece um erro, como era esperado. Altere a procedure para tratar o erro quando a entrada for NULL:

CREATE OR REPLACE PROCEDURE INCLUIR_SEGMENTO
(p_ID SEGMERCADO.ID%type, p_DESCRICAO SEGMERCADO.DESCRICAO%type)
IS
BEGIN
    INSERT INTO SEGMERCADO(ID, DESCRICAO) VALUES (p_ID, p_DESCRICAO);
EXCEPTION
    WHEN EXCEPTION_PAC.e_NULL THEN
        raise_application_error (-20030, 'CAMPO DO SEGMENTO COM PREENCHIMENTO OBRIGATÓRIO');
END;
```

```sql
-- Em user_dev, dê privilégios e associe à PUBLIC:

GRANT EXECUTE ON EXCEPTION_PAC TO PUBLIC;

CREATE PUBLIC SYNONYM EXCEPTION_PAC FOR user_dev.EXCEPTION_PAC;

```

```sql
-- Tente incluir o segmento com NULL:

EXECUTE INCLUIR_SEGMENTO (NULL, 'TRANSPORTES');

--Agora, aparece o erro tratado como exceção.
```

```sql
-- Dê um duplo clique na função OBTER_CATEGORIA_CLIENTE e observe o código fonte para --determinar a classificação do cliente.

-- Crie o pacote com as constantes:

create or replace NONEDITIONABLE PACKAGE CLIENTE_PAC
IS
PROCEDURE INCLUIR_CLIENTE
    (p_id in cliente.id%type,
    p_razao_social in cliente.razao_social%type,
    p_CNPJ cliente.CNPJ%type ,
    p_segmercado_id cliente.segmercado_id%type,
    p_faturamento_previsto cliente.faturamento_previsto%type);

PROCEDURE ATUALIZAR_CLI_SEG_MERCADO
    (p_id cliente.id%type,
        p_segmercado_id cliente.segmercado_id%type);

PROCEDURE ATUALIZAR_FATURAMENTO_PREVISTO
    (p_id in cliente.id%type,
        p_faturamento_previsto in cliente.faturamento_previsto%type);

PROCEDURE EXCLUIR_CLIENTE
    (p_id in cliente.id%type);

PROCEDURE INCLUIR_CLIENTE
    (p_id in cliente.id%type,
    p_razao_social in cliente.razao_social%type,
    p_segmercado_id cliente.segmercado_id%type);

c_PEQUENO NUMBER(10) := 10000;
c_MEDIO NUMBER(10) := 50000;
c_MEDIO_GRANDE NUMBER(10) := 90000;

END;

-- Compile o cabeçalho do pacote.
```

```sql
-- Substitua no corpo do pacote:

create or replace NONEDITIONABLE PACKAGE BODY CLIENTE_PAC
IS
FUNCTION VERIFICA_SEGMENTO_MERCADO
    (p_id in segmercado.id%type)
        RETURN boolean
IS
    v_dummy number(1);
BEGIN
    SELECT 1 into v_dummy
        FROM segmercado
        WHERE id = p_id;
    RETURN true;
EXCEPTION
    WHEN no_data_found then
        RETURN false;
END;

FUNCTION OBTER_CATEGORIA_CLIENTE
    (p_faturamento_previsto IN cliente.faturamento_previsto%type)
    RETURN cliente.categoria%type
IS
BEGIN
    IF p_faturamento_previsto <= c_PEQUENO THEN
        RETURN 'PEQUENO';
    ELSIF p_faturamento_previsto <= c_MEDIO THEN
        RETURN 'MEDIO';
    ELSIF p_faturamento_previsto <= c_MEDIO_GRANDE THEN
        RETURN 'MEDIO GRANDE';
    ELSE
        RETURN 'GRANDE';
    END IF;
END;

PROCEDURE FORMAT_CNPJ
    (p_cnpj IN OUT varchar2)
IS
BEGIN
    p_cnpj := substr(p_cnpj,1,2) ||'/'|| substr(p_cnpj,3);
    DBMS_OUTPUT.PUT_LINE('CHAMEI A ROTINA FORMAT_CNPJ DO PACOTE !!!!');
END;

PROCEDURE INCLUIR_CLIENTE
    (p_id in cliente.id%type,
    p_razao_social in cliente.razao_social%type,
    p_CNPJ cliente.CNPJ%type ,
    p_segmercado_id cliente.segmercado_id%type,
    p_faturamento_previsto cliente.faturamento_previsto%type)
IS
    v_categoria cliente.categoria%type;
    v_CNPJ cliente.cnpj%type := p_CNPJ;
    v_codigo_erro number(5);
    v_mensagem_erro varchar2(200);
    v_dummy number;
    v_verifica_segmento boolean;
    e_segmento exception;
BEGIN
    v_verifica_segmento :=     verifica_segmento_mercado(p_segmercado_id);
    IF v_verifica_segmento = false THEN
        RAISE e_segmento;
    END IF;
    v_categoria := obter_categoria_cliente(p_faturamento_previsto);
    format_cnpj (v_cnpj);
    INSERT INTO cliente
        VALUES (p_id, UPPER(p_razao_social), v_CNPJ, p_segmercado_id
            ,SYSDATE, p_faturamento_previsto, v_categoria);
    COMMIT;
EXCEPTION
    WHEN dup_val_on_index then
        raise_application_error(-20010,'Cliente já cadastrado');
    WHEN e_segmento then
        raise_application_error (-20011,'Segmento de mercado inexistente');
    WHEN EXCEPTION_PAC.e_NULL then
        raise_application_error (-20020,'Preenchimento do valor do campo é obrigatório. Favor incluir este valor !!!');
    WHEN OTHERS then
        v_codigo_erro := sqlcode;
        v_mensagem_erro := sqlerrm;
        raise_application_error (-20000,to_char(v_codigo_erro)||v_mensagem_erro);
END;

PROCEDURE ATUALIZAR_CLI_SEG_MERCADO
    (p_id cliente.id%type,
        p_segmercado_id cliente.segmercado_id%type)
IS
        e_fk exception;
        pragma exception_init(e_fk, -2291);
        e_no_update exception;
BEGIN
    UPDATE cliente
        SET segmercado_id = p_segmercado_id
        WHERE id = p_id;
    IF SQL%NOTFOUND then
        RAISE e_no_update;
    END IF;
    COMMIT;
EXCEPTION
    WHEN e_fk then
        RAISE_APPLICATION_ERROR (-20001,'Segmento de Mercado Inexistente');
    WHEN e_no_update then
        RAISE_APPLICATION_ERROR (-20002,'Cliente Inexistente');
END;

PROCEDURE ATUALIZAR_FATURAMENTO_PREVISTO
    (p_id in cliente.id%type,
        p_faturamento_previsto in cliente.faturamento_previsto%type)
IS
    v_categoria cliente.categoria%type;
    e_error_id exception;
BEGIN
    v_categoria := obter_categoria_cliente(p_faturamento_previsto);
    UPDATE cliente
        SET categoria = v_categoria,
            faturamento_previsto = p_faturamento_previsto
        WHERE id = p_id;
    IF SQL%NOTFOUND THEN
        RAISE e_error_id;
    END IF;
    COMMIT;
EXCEPTION
    WHEN e_error_id then
        raise_application_error(-20010,'Cliente inexistente');
END;

PROCEDURE EXCLUIR_CLIENTE
    (p_id in cliente.id%type)
IS
    e_error_id exception;
BEGIN
    DELETE FROM cliente
        WHERE id = p_id;
    IF SQL%NOTFOUND THEN
        RAISE e_error_id;
    END IF;
    COMMIT;
EXCEPTION
    WHEN e_error_id then
        raise_application_error(-20010,'Cliente inexistente');
END;

PROCEDURE INCLUIR_CLIENTE
    (p_id in cliente.id%type,
    p_razao_social in cliente.razao_social%type,
    p_segmercado_id cliente.segmercado_id%type)
IS
    v_codigo_erro number(5);
    v_mensagem_erro varchar2(200);
    v_dummy number;
    v_verifica_segmento boolean;
    e_segmento exception;
BEGIN
    v_verifica_segmento :=     verifica_segmento_mercado(p_segmercado_id);
    IF v_verifica_segmento = false THEN
        RAISE e_segmento;
    END IF;
    INSERT INTO cliente (ID, RAZAO_SOCIAL, SEGMERCADO_ID, DATA_INCLUSAO)
        VALUES (p_id, UPPER(p_razao_social), p_segmercado_id
            ,SYSDATE);
    COMMIT;
EXCEPTION
    WHEN dup_val_on_index then
        raise_application_error(-20010,'Cliente já cadastrado');
    WHEN e_segmento then
        raise_application_error (-20011,'Segmento de mercado inexistente');
    WHEN EXCEPTION_PAC.e_NULL then
        raise_application_error (-20020,'Preenchimento do valor do campo é obrigatório. Favor incluir este valor !!!');
    WHEN OTHERS then
        v_codigo_erro := sqlcode;
        v_mensagem_erro := sqlerrm;
        raise_application_error (-20000,to_char(v_codigo_erro)||v_mensagem_erro);
END;

END;
```

```sql
-- Crie um novo script associado ao user_app.

-- Mostre o conteúdo de CLIENTE:

SELECT * FROM CLIENTE;
```

```sql
-- Execute o código para incluir um novo cliente:

EXECUTE APP_INCLUIR_CLIENTE (31, 'CLIENTE INCLUIDO PELO PACOTE', '23333', 2, 100000);
```

```sql
-- Mostre o cliente com ID 31:

SELECT * FROM CLIENTE WHERE ID = 31;
```

```sql
-- Inclua o novo cliente:

EXECUTE APP_INCLUIR_CLIENTE (32, 'CLIENTE INCLUIDO PELO PACOTE', '23333', 2, 100000);
```

```sql
-- Mostre como foi a classificação dos clientes:

SELECT * FROM CLIENTE WHERE ID = 31 OR ID = 32;
```

```sql
-- Para saber o limite das categorias, execute os comandos:

SET SERVEROUTPUT ON;
EXEC dbms_output.put_line(CLIENTE_PAC.c_MEDIO_GRANDE);
```
