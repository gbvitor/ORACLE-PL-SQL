### Alteração da sessão.

```bash
ALTER SESSION SET "_ORACLE_SCRIPT" = true;
```

Este comando altera a configuração da sessão atual para permitir o uso de recursos que não são geralmente permitidos em uma sessão padrão. A variável \_ORACLE_SCRIPT é uma configuração interna usada para permitir operações administrativas específicas e pode estar desativada por padrão por motivos de segurança. Defini-la como true pode permitir criar ou modificar objetos que exigem permissões especiais, como criar usuários ou alterar a configuração de sistema.

### Criação de um úsuario.

```bash
CREATE USER user IDENTIFIED BY pass DEFAULT TABLESPACE USERS;
```

Este comando cria um novo usuário no banco de dados com o nome user e define a senha como pass. Além disso, especifica que o tablespace padrão para este usuário será USERS. Um tablespace é uma estrutura lógica no banco de dados que agrupa os objetos de banco de dados.

### Alterando as garantias do úsuario.

```bash
GRANT connect, resource TO user
```

Aqui, são concedidas duas permissões ao usuário criado:

-  CONNECT: Permite que o usuário se conecte ao banco de dados.
-  RESOURCE: Permite que o usuário crie objetos de banco de dados, como tabelas, índices e outros. Esta permissão dá ao usuário a capacidade de criar e manipular seus próprios objetos dentro do banco de dados.

### Alterando o usuário.

```bash
ALTER USER sgvitor QUOTA UNLIMITED ON USERS;
```

Este comando modifica a cota de espaço em disco para o usuário sgvitor, permitindo-lhe usar uma quantidade ilimitada de espaço no tablespace USERS. Em outras palavras, o usuário sgvitor pode armazenar um número ilimitado de dados no tablespace USERS, sem restrições de espaço.
