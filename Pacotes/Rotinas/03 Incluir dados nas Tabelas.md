```sql
BEGIN

   -- Incluir segmentos de mercado.

   insert into segmercado values (1,'VAREJISTA');
   insert into segmercado values (2,'ATACADISTA');
   insert into segmercado values (3,'FARMACEUTICO');
   insert into segmercado values (4,'INDUSTRIAL');
   insert into segmercado values (5,'AGROPECUARIA');


   -- incluir clientes.

   insert into cliente values (1 ,'SUPERMERCADO XYZ','12/345',5,sysdate,150000, 'GRANDE' );
   insert into cliente values (2 ,'SUPERMERCADO IJK','67/890',1,sysdate,90000, 'MEDIO GRANDE' );
   insert into cliente values (3 ,'SUPERMERCADO IJK','89/012',3,sysdate,80000, 'MEDIO GRANDE' );
   insert into cliente values (4 ,'FARMACIA AXZ','12/378',3, sysdate,80000,  'MEDIO GRANDE' );

   COMMIT;

END;
```
