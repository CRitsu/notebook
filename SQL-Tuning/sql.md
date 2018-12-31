```sql
CREATE TABLE customer (
	id NUMBER NOT NULL,
	name VARCHAR2 ( 256 ),
	info1 VARCHAR2 ( 256 ),
	info2 VARCHAR2 ( 256 ),
	info3 VARCHAR2 ( 256 )
);
```

```sql
DECLARE
  v_num NUMBER := 1;
BEGIN
	LOOP
    INSERT INTO customer
    VALUES (
      v_num,
      'name' || v_num,
      'info1_' || v_num,
      'info2_' || v_num,
      'info3_' || v_num
    );
    IF v_num >= 1000 THEN
        EXIT;
    END IF;
    v_num := v_num + 1;
  END LOOP;
END;
```
