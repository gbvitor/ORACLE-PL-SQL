```sql
-- 1) Clique sobre o cabeçalho do pacote.

-- 2) Altere o código para:

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

END;

-- 3) Clique no ícone de Compilar para Depuração.

-- 4) Para corrigir o erro, altere o código:

create or replace NONEDITIONABLE PACKAGE BODY CLIENTE_PAC
IS
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
    WHEN OTHERS then
        v_codigo_erro := sqlcode;
        v_mensagem_erro := sqlerrm;
        raise_application_error (-20000,to_char(v_codigo_erro)||v_mensagem_erro);
END;

END;

-- 5) Para passar 5 parâmetros, use o código:

EXECUTE CLIENTE_PAC.INCLUIR_CLIENTE(15,'INCLUIR CLIENTE COM 5 PARAMETROS','99999',2,90000);

-- 6) Mostre o conteúdo da tabela CLIENTE:

SELECT * FROM CLIENTE;

-- 7) Passe somente 3 parâmetros no pacote:

EXECUTE CLIENTE_PAC.INCLUIR_CLIENTE(16,'INCLUIR CLIENTE COM 3 PARAMETROS',2);

-- 8) Veja o que aconteceu com a tabela CLIENTE:

SELECT * FROM CLIENTE;

-- 9) Altere o código para colocar as funções internas ao pacote:

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
    WHEN OTHERS then
        v_codigo_erro := sqlcode;
        v_mensagem_erro := sqlerrm;
        raise_application_error (-20000,to_char(v_codigo_erro)||v_mensagem_erro);
END;

END;

-- 10) Inclua o cliente usando o pacote:

SET SERVEROUTPUT ON;
EXECUTE CLIENTE_PAC.INCLUIR_CLIENTE(18,'INCLUIR CLIENTE PELO PACOTE USANDO PROC INTERNA','22222',2,50000);

-- 11) Execute a procedure INCLUIR_CLIENTE sem chamar o pacote:

EXECUTE INCLUIR_CLIENTE(19,'INCLUIR CLIENTE FORA DO PACOTE','22222',2,50000);

-- 12) Mostre o resultado na tabela CLIENTE:

SELECT * FROM CLIENTE;

-- 13) Crie um novo script associado ao usuário user_dev.

-- 14) Para verificar a dependência de INCLUIR_CLIENTE, execute o comando:

EXECUTE DEPTREE_FILL('procedure','user_dev','INCLUIR_CLIENTE');

-- 15) Veja as dependências com o comando:

SELECT NESTED_LEVEL, SCHEMA, TYPE, NAME FROM DEPTREE ORDER BY SEQ#;

-- 16) Vá na área do user_app e use o comando:

create or replace NONEDITIONABLE PROCEDURE APP_INCLUIR_CLIENTE
(p_ID IN cliente.id%type,
p_RAZAO IN cliente.razao_social%type,
p_CNPJ IN cliente.cnpj%type,
p_SEGMERCADO in cliente.segmercado_id%type,
p_FATURAMENTO IN cliente.faturamento_previsto%type)
IS
BEGIN
    CLIENTE_PAC.INCLUIR_CLIENTE(p_ID, p_RAZAO, p_CNPJ, p_SEGMERCADO, p_FATURAMENTO);
END;

-- 17) Agora, para verificar a dependência de ATUALIZAR_CLI_SEG_MERCADO, execute o comando:

EXECUTE DEPTREE_FILL('procedure','user_dev','ATUALIZAR_CLI_SEG_MERCADO');

-- 18) Veja as dependências da procedure:

SELECT NESTED_LEVEL, SCHEMA, TYPE, NAME FROM DEPTREE ORDER BY SEQ#;

-- 19) Veja as dependências de ATUALIZAR_FATURAMENTO_PREVISTO, executando o comando:

EXECUTE DEPTREE_FILL('procedure','user_dev','ATUALIZAR_FATURAMENTO_PREVISTO');

-- 20) A procedure têm as dependências:

SELECT NESTED_LEVEL, SCHEMA, TYPE, NAME FROM DEPTREE ORDER BY SEQ#;

-- 21) Para verificar as dependências de EXCLUIR_CLIENTE, execute o comando:

EXECUTE DEPTREE_FILL('procedure','user_dev','EXCLUIR_CLIENTE');

-- 22) Veja as dependências, executando o comando:

SELECT NESTED_LEVEL, SCHEMA, TYPE, NAME FROM DEPTREE ORDER BY SEQ#;

-- 23) Para verificar as dependências da procedure FORMAT_CNPJ, execute o comando:

EXECUTE DEPTREE_FILL('procedure','user_dev','FORMAT_CNPJ');

-- 24) Veja as dependências com o comando:

SELECT NESTED_LEVEL, SCHEMA, TYPE, NAME FROM DEPTREE ORDER BY SEQ#;

-- 25) Para verificar as dependências da função OBTER_CATEGORIA_CLIENTE, execute o comando:

EXECUTE DEPTREE_FILL('function','user_dev','OBTER_CATEGORIA_CLIENTE');

-- 26) Veja as dependências com o comando:

SELECT NESTED_LEVEL, SCHEMA, TYPE, NAME FROM DEPTREE ORDER BY SEQ#;

-- 27) Para verificar a dependência da função VERIFICA_SEGMENTO_MERCADO, execute o comando:

EXECUTE DEPTREE_FILL('function','user_dev','VERIFICA_SEGMENTO_MERCADO');

-- 28) Para excluir as procedures, execute os comandos:

DROP PROCEDURE INCLUIR_CLIENTE;
DROP PROCEDURE ATUALIZAR_CLI_SEG_MERCADO;
DROP PROCEDURE ATUALIZAR_FATURAMENTO_PREVISTO;
DROP PROCEDURE EXCLUIR_CLIENTE;

-- 29) Agora, apague o restante:

DROP PROCEDURE FORMAT_CNPJ;
DROP FUNCTION OBTER_CATEGORIA_CLIENTE;
DROP FUNCTION VERIFICA_SEGMENTO_MERCADO;
```
