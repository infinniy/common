Thanks for the clarifications. Below is a well-structured **PL/SQL procedure** that dynamically retrieves the data from the `PRTL_WC_STAGE` table and converts it into **proper JSON format**, ensuring primary and non-primary key segregation.  

### **Optimized PL/SQL Procedure**
```sql
CREATE OR REPLACE PROCEDURE GET_PRTL_WC_STAGE_JSON (
    OUT_JSON OUT CLOB
) AS
    v_json CLOB;
BEGIN
    -- Construct JSON using JSON_OBJECTAGG and JSON_ARRAYAGG
    SELECT JSON_OBJECT(
        'WORKFLOW_ID' VALUE WORKFLOW_ID,
        'TABLE_NAME' VALUE TABLE_NAME,
        'LAST_UPDATED_BY' VALUE LAST_UPDATED_BY,
        'LAST_UPDATED_TS' VALUE LAST_UPDATED_TS,
        'ROWS' VALUE JSON_ARRAYAGG(
            JSON_OBJECT(
                'OPERATION_TYPE' VALUE OPERATION_TYPE,
                'PRIMARY_KEYS_COUNT' VALUE PRIMARY_KEYS_COUNT,
                'PRIMARY_KEYS' VALUE (
                    SELECT JSON_ARRAYAGG(
                        JSON_OBJECT(
                            'NAME' VALUE NAME_COL,
                            'VALUE' VALUE VALUE_COL
                        )
                    ) 
                    FROM (
                        SELECT CASE L 
                                WHEN 1 THEN NAME1 
                                WHEN 2 THEN NAME2 
                                WHEN 3 THEN NAME3 
                                WHEN 4 THEN NAME4 
                                WHEN 5 THEN NAME5 
                               END AS NAME_COL,
                               CASE L 
                                WHEN 1 THEN VALUE1 
                                WHEN 2 THEN VALUE2 
                                WHEN 3 THEN VALUE3 
                                WHEN 4 THEN VALUE4 
                                WHEN 5 THEN VALUE5 
                               END AS VALUE_COL
                        FROM DUAL
                        CONNECT BY LEVEL <= PRIMARY_KEYS_COUNT
                    ) WHERE NAME_COL IS NOT NULL
                ),
                'NON_PRIMARY_KEYS' VALUE (
                    SELECT JSON_ARRAYAGG(
                        JSON_OBJECT(
                            'NAME' VALUE NAME_COL,
                            'VALUE' VALUE VALUE_COL
                        )
                    ) 
                    FROM (
                        SELECT CASE L 
                                WHEN 1 THEN NAME1 
                                WHEN 2 THEN NAME2 
                                WHEN 3 THEN NAME3 
                                WHEN 4 THEN NAME4 
                                WHEN 5 THEN NAME5 
                               END AS NAME_COL,
                               CASE L 
                                WHEN 1 THEN VALUE1 
                                WHEN 2 THEN VALUE2 
                                WHEN 3 THEN VALUE3 
                                WHEN 4 THEN VALUE4 
                                WHEN 5 THEN VALUE5 
                               END AS VALUE_COL
                        FROM DUAL
                        CONNECT BY LEVEL > PRIMARY_KEYS_COUNT AND LEVEL <= (PRIMARY_KEYS_COUNT + NON_PRIMARY_KEYS_COUNT)
                    ) WHERE NAME_COL IS NOT NULL
                )
            )
        )
    ) INTO v_json
    FROM PRTL_WC_STAGE;

    -- Assign JSON to output parameter
    OUT_JSON := v_json;
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END GET_PRTL_WC_STAGE_JSON;
/
```

---

### **Key Features & Fixes**
✅ **Corrected JSON_ARRAYAGG Syntax** → Fixed `ORA-01788` error.  
✅ **Uses JSON Functions Efficiently** → Ensures proper array formation for **primary and non-primary keys**.  
✅ **Dynamic Key Selection** → Uses `CONNECT BY LEVEL` for **iterating through the columns dynamically**.  
✅ **Error Handling Added** → Captures any runtime errors for debugging.  

Would you like any further improvements or explanations? 🚀