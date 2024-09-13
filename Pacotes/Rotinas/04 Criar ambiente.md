```sql
-- No Oracle SQL Developer ir para: Menu Superior -> Ferramentas -> Preferencias
-- Banco de Dados -> Planilha
-- Em SELECIONAR CAMINHO PADRÃO PARA PROCURA DE SCRIPTS colocar o local do script.

CREATE OR REPLACE FUNCTION OBTER_CATEGORIA_CLIENTE
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
/

CREATE OR REPLACE FUNCTION OBTER_DESCRICAO_SEGMENTO
    (p_id in segmercado.id%type)
    RETURN segmercado.descricao%type
IS
    v_descricao segmercado.descricao%type;
BEGIN
    SELECT descricao INTO v_descricao
       FROM segmercado
       WHERE id = p_id;
    RETURN v_descricao;
EXCEPTION
        WHEN no_data_found then
            raise_application_error(-20002,'Segmento de Mercado	Inexistente');
END;
/

CREATE OR REPLACE FUNCTION VERIFICA_SEGMENTO_MERCADO
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
/


CREATE OR REPLACE PROCEDURE FORMAT_CNPJ
			(p_cnpj IN OUT varchar2)
IS
BEGIN
	p_cnpj := substr(p_cnpj,1,2) ||'/'|| substr(p_cnpj,3);
END;
/

CREATE OR REPLACE PROCEDURE ATUALIZAR_CLI_SEG_MERCADO
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
/

CREATE OR REPLACE PROCEDURE ATUALIZAR_FATURAMENTO_PREVISTO
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
/

CREATE OR REPLACE PROCEDURE EXCLUIR_CLIENTE
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
/

CREATE OR REPLACE PROCEDURE INCLUIR_CLIENTE
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
    v_verifica_segmento := 	verifica_segmento_mercado(p_segmercado_id);
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
        raise_application_error(-20010,'Cliente j� cadastrado');
    WHEN e_segmento then
        raise_application_error (-20011,'Segmento de mercado inexistente');
    WHEN OTHERS then
        v_codigo_erro := sqlcode;
        v_mensagem_erro := sqlerrm;
        raise_application_error (-20000,to_char(v_codigo_erro)||v_mensagem_erro);
END;
/

CREATE OR REPLACE PROCEDURE INCLUIR_SEGMERCADO
    (p_id in segmercado.id%type,
     p_descricao in segmercado.descricao%type)
IS
 BEGIN
    INSERT into segmercado
       values(p_id, UPPER(p_descricao));
    COMMIT;
EXCEPTION
    WHEN dup_val_on_index THEN
        raise_application_error(-20001,'Segmento de Mercado j� Cadastrado');
END;
/



CREATE OR REPLACE FUNCTION VERIFICA_SEGMENTO_MERCADO
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
/
```
