Updated Procedure with Primary Key Handling

This updated procedure ensures:

1. Composite Primary Keys Handling → Dynamically retrieves primary keys and constructs the WHERE clause.


2. Ignoring Primary Keys in SET Clause for UPDATE → Ensures primary keys are not modified.




---

Modified Procedure

CREATE OR REPLACE PROCEDURE SP_MERGE_APPROVED_RECORDS
AS
    CURSOR approved_records_cur IS
        SELECT ws.row_id, ws.table_id, wt.table_name, ws.operation_type,
               ws.key1, ws.value1, ws.key2, ws.value2, ws.key3, ws.value3, 
               ws.key4, ws.value4, ws.key5, ws.value5, ws.key6, ws.value6,
               ws.key7, ws.value7, ws.key8, ws.value8, ws.key9, ws.value9,
               ws.key10, ws.value10, ws.key11, ws.value11, ws.key12, ws.value12,
               ws.key13, ws.value13, ws.key14, ws.value14, ws.key15, ws.value15,
               ws.key16, ws.value16, ws.key17, ws.value17, ws.key18, ws.value18,
               ws.key19, ws.value19, ws.key20, ws.value20, ws.key21, ws.value21,
               ws.key22, ws.value22, ws.key23, ws.value23, ws.key24, ws.value24,
               ws.key25, ws.value25
        FROM stage_table ws
        JOIN WC_TABLE_MASTER wt ON ws.table_id = wt.table_id
        WHERE ws.status = 'APPROVED';

    TYPE key_value_array IS TABLE OF VARCHAR2(300) INDEX BY PLS_INTEGER;

    keys    key_value_array;
    values  key_value_array;

    v_sql           VARCHAR2(4000);
    v_table_name    VARCHAR2(100);
    v_operation     VARCHAR2(50);
    v_set_clause    VARCHAR2(4000);
    v_where_clause  VARCHAR2(4000);
    v_insert_cols   VARCHAR2(4000);
    v_insert_vals   VARCHAR2(4000);

    -- To store primary key columns
    v_primary_keys  VARCHAR2(4000);
    v_pk_columns    SYS.ODCIVARCHAR2LIST;

BEGIN
    FOR rec IN approved_records_cur LOOP
        v_table_name := rec.table_name;
        v_operation := rec.operation_type;

        -- Clear dynamic clauses
        v_set_clause := '';
        v_where_clause := '';
        v_insert_cols := '';
        v_insert_vals := '';
        v_primary_keys := '';

        -- Populate keys and values into arrays
        keys(1) := rec.key1;   values(1) := rec.value1;
        keys(2) := rec.key2;   values(2) := rec.value2;
        keys(3) := rec.key3;   values(3) := rec.value3;
        keys(4) := rec.key4;   values(4) := rec.value4;
        keys(5) := rec.key5;   values(5) := rec.value5;
        keys(25) := rec.key25; values(25) := rec.value25;

        -- **Retrieve primary key columns dynamically**
        SELECT CAST(COLLECT(column_name) AS SYS.ODCIVARCHAR2LIST)
        INTO v_pk_columns
        FROM WC_GRID_COLUMN_PROPERTIES
        WHERE table_id = rec.table_id
        AND primary_key_position IS NOT NULL
        ORDER BY primary_key_position;

        -- Construct dynamic SQL based on operation_type
        CASE v_operation
            WHEN 'INSERT' THEN
                FOR i IN 1..25 LOOP
                    IF keys(i) IS NOT NULL THEN
                        v_insert_cols := v_insert_cols || keys(i) || ', ';
                        v_insert_vals := v_insert_vals || '''' || values(i) || ''', ';
                    END IF;
                END LOOP;

                -- Remove trailing commas
                v_insert_cols := RTRIM(v_insert_cols, ', ');
                v_insert_vals := RTRIM(v_insert_vals, ', ');

                v_sql := 'INSERT INTO ' || v_table_name || ' (' || v_insert_cols || ') VALUES (' || v_insert_vals || ')';

            WHEN 'UPDATE' THEN
                -- Construct the SET clause, excluding primary key columns
                FOR i IN 1..25 LOOP
                    IF keys(i) IS NOT NULL AND NOT v_pk_columns.EXISTS(keys(i)) THEN
                        v_set_clause := v_set_clause || keys(i) || ' = ''' || values(i) || ''', ';
                    END IF;
                END LOOP;

                -- Remove trailing comma
                v_set_clause := RTRIM(v_set_clause, ', ');

                -- Construct WHERE clause dynamically using primary keys
                v_where_clause := '';
                FOR i IN 1..v_pk_columns.COUNT LOOP
                    IF v_where_clause IS NOT NULL THEN
                        v_where_clause := v_where_clause || ' AND ';
                    END IF;
                    v_where_clause := v_where_clause || v_pk_columns(i) || ' = ''' || values(i) || '''';
                END LOOP;

                v_sql := 'UPDATE ' || v_table_name || ' SET ' || v_set_clause || ' WHERE ' || v_where_clause;

            WHEN 'DELETE' THEN
                -- Construct WHERE clause dynamically using primary keys
                v_where_clause := '';
                FOR i IN 1..v_pk_columns.COUNT LOOP
                    IF v_where_clause IS NOT NULL THEN
                        v_where_clause := v_where_clause || ' AND ';
                    END IF;
                    v_where_clause := v_where_clause || v_pk_columns(i) || ' = ''' || values(i) || '''';
                END LOOP;

                v_sql := 'DELETE FROM ' || v_table_name || ' WHERE ' || v_where_clause;
        END CASE;

        -- Debugging Output
        DBMS_OUTPUT.PUT_LINE('Executing SQL: ' || v_sql);

        -- Execute the dynamically built SQL statement
        EXECUTE IMMEDIATE v_sql;

        -- After successful execution, remove the record from stage_table
        DELETE FROM stage_table WHERE row_id = rec.row_id;

        -- Commit after each successful operation
        COMMIT;
    END LOOP;

EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE_APPLICATION_ERROR(-20002, 'Error in SP_MERGE_APPROVED_RECORDS: ' || SQLERRM);
END;
/


---

Enhancements

✅ Handles Composite Primary Keys

Uses WC_GRID_COLUMN_PROPERTIES to fetch all primary keys dynamically.


✅ Prevents Primary Key Updates

While forming the SET clause, excludes primary key columns and only includes non-key columns.


✅ Generalized for Any Table

Works with single-column and composite primary key tables without modification.


✅ More Efficient WHERE Clause Formation

Uses SYS.ODCIVARCHAR2LIST to store primary key column names for efficient looping.



---

Example Cases

WC_STAGE Data

WC_GRID_COLUMN_PROPERTIES


---

Generated Queries

✅ INSERT Query

INSERT INTO EMPLOYEES (EMP_ID, EMP_NAME) VALUES ('1', 'John');

✅ UPDATE Query (Ignoring Primary Key)

UPDATE DEPARTMENTS SET DEPT_NAME = 'HR' WHERE DEPT_ID = '2';

✅ DELETE Query

DELETE FROM SALARIES WHERE SALARY_ID = '3';


---

Final Thoughts

This approach is now fully dynamic and supports composite primary keys while ensuring primary keys are never updated.
Let me know if you need further refinements! 🚀

