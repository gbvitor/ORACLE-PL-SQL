### Em PL/SQL, um package é uma estrutura que agrupa um conjunto de procedimentos, funções, variáveis e tipos de dados relacionados de forma organizada e modular. Utilizar packages em PL/SQL oferece várias vantagens, incluindo encapsulamento, reutilização e controle de acesso.

Um package em PL/SQL é dividido em duas partes principais:

### Especificação do Package (Package Specification):

-  Define a interface pública do package.
-  Contém declarações de procedimentos, funções, tipos de dados e variáveis globais que serão acessíveis fora do package.
-  Serve como um contrato ou uma API para os usuários do package.

### Corpo do Package (Package Body):

-  Implementa os procedimentos e funções definidos na especificação do package.
-  Contém o código real para as rotinas e pode também declarar variáveis e tipos de dados privados que são usados apenas dentro do corpo do package.
-  Pode ter implementações de procedimentos e funções que não são expostos na especificação (ou seja, são privados).

### Vantagens de Usar Packages

#### Encapsulamento:

-  Permite esconder a implementação interna de uma aplicação, expondo apenas o que é necessário.

#### Reusabilidade:

-  Facilita a reutilização de código em diferentes partes da aplicação.

#### Controle de Acesso:

-  Permite definir o acesso a dados e procedimentos, protegendo partes da lógica de negócios.

#### Organização:

-  Ajuda a organizar o código de forma modular, facilitando a manutenção e a compreensão.

#### Desempenho:

-  A PL/SQL pode armazenar o código do package em memória, melhorando o desempenho das chamadas repetidas a procedimentos e funções.

Em resumo, packages são uma ferramenta poderosa em PL/SQL para estruturar e gerenciar código de maneira eficiente e segura.

```sql
-- Crie um script através do user_dev.

-- Use os comandos para criar o pacote:

CREATE OR REPLACE PACKAGE CLIENTE_PAC
IS
PROCEDURE INCLUIR_CLIENTE
    (p_id in cliente.id%type,
    p_razao_social in cliente.razao_social%type,
    p_CNPJ cliente.CNPJ%type ,
    p_segmercado_id cliente.segmercado_id%type,
    p_faturamento_previsto cliente.faturamento_previsto%type);

END;
```

```sql
-- Para criar o corpo do pacote, execute os comandos:

CREATE OR REPLACE PACKAGE BODY CLIENTE_PAC
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

END;
```

```sql
-- Para garantir o privilégio de execução ao user_app, execute:

GRANT EXECUTE ON CLIENTE_PAC TO user_app;
```

```sql
-- Crie um script como user_app e execute o comando:

EXECUTE user_dev.CLIENTE_PAC.INCLUIR_CLIENTE(10, 'PRIMEIRO CLIENTE INCLUIDO POR USER_APP VIA PACKAGE', '455564', 2, 120000);
```

```sql
-- Mostre o conteúdo da tabela:

SELECT * FROM CLIENTE;
```

```sql
-- Crie um sinônimo para o pacote na conexão user_dev:

CREATE PUBLIC SYNONYM CLIENTE_PAC FOR user_dev.CLIENTE_PAC;
```

```sql
-- Em user_app, rode o comando:

EXECUTE CLIENTE_PAC.INCLUIR_CLIENTE(11, 'SEGUNDO CLIENTE INCLUIDO POR USER_APP VIA PACKAGE', '455564', 2, 120000);
```

```sql
-- Crie um novo script associado à user_dev e execute os comandos para criar o cabeçalho do pacote:

create or replace PACKAGE CLIENTE_PAC
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

END;
```

```sql
-- Agora, execute os comandos:

-- ================================================

create or replace PACKAGE BODY CLIENTE_PAC
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

END;
```

```sql
-- Crie um novo script com o user_app e verifique o conteúdo da tabela CLIENTE:

SELECT * FROM CLIENTE;
```

```sql
-- Para excluir o cliente, execute o comando:

EXECUTE CLIENTE_PAC.EXCLUIR_CLIENTE(10);
```
