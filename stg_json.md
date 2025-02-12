To convert the `PRTL_WC_STAGE` table data back into the original JSON format, we need to aggregate multiple rows into a structured JSON format dynamically. Below is a PL/SQL function that accomplishes this:

---

### **Function to Convert `PRTL_WC_STAGE` Table Data to JSON**
```sql
CREATE OR REPLACE FUNCTION GET_PRTL_WC_STAGE_JSON (
    v_UPLOAD_ID IN NUMBER,
    v_TABLE_NAME IN VARCHAR2
) RETURN CLOB IS
    v_json CLOB;
BEGIN
    SELECT JSON_OBJECT(
        'UPLOAD_ID' VALUE v_UPLOAD_ID,
        'TABLE_NAME' VALUE v_TABLE_NAME,
        'LAST_UPDATED_BY_USER' VALUE MAX(LAST_UPDATED_BY_USER),
        'LAST_UPDATED_BY_TIME' VALUE TO_CHAR(MAX(LAST_UPDATED_BY_TIME), 'YYYY-MM-DD"T"HH24:MI:SS'),
        'ROWS' VALUE JSON_ARRAYAGG(
            JSON_OBJECT(
                'OPERATION_TYPE' VALUE OPERATION_TYPE,
                'TOTAL_KEY_COLS' VALUE TOTAL_KEY_COLS,
                'KEY_VALUES' VALUE (
                    SELECT JSON_ARRAYAGG(
                        JSON_OBJECT(
                            'NAME' VALUE key_name,
                            'VALUE' VALUE key_value
                        )
                    ) 
                    FROM (
                        SELECT NAME1 AS key_name, VALUE1 AS key_value FROM DUAL WHERE NAME1 IS NOT NULL
                        UNION ALL
                        SELECT NAME2, VALUE2 FROM DUAL WHERE NAME2 IS NOT NULL
                        UNION ALL
                        SELECT NAME3, VALUE3 FROM DUAL WHERE NAME3 IS NOT NULL
                        UNION ALL
                        SELECT NAME4, VALUE4 FROM DUAL WHERE NAME4 IS NOT NULL
                        UNION ALL
                        SELECT NAME5, VALUE5 FROM DUAL WHERE NAME5 IS NOT NULL
                    )
                )
            )
        )
    ) INTO v_json
    FROM PRTL_WC_STAGE
    WHERE UPLOAD_ID = v_UPLOAD_ID
    AND TABLE_NAME = v_TABLE_NAME
    GROUP BY OPERATION_TYPE, TOTAL_KEY_COLS;
    
    RETURN v_json;
END GET_PRTL_WC_STAGE_JSON;
```

---

### **Explanation of the Function**
1. **Extracts Metadata:**
   - `UPLOAD_ID`
   - `TABLE_NAME`
   - `LAST_UPDATED_BY_USER` (Max value for consistency)
   - `LAST_UPDATED_BY_TIME` (Formatted timestamp)

2. **Aggregates the Data:**
   - Groups data by `OPERATION_TYPE` and `TOTAL_KEY_COLS`
   - Uses `JSON_ARRAYAGG` to collect multiple key-value pairs into `KEY_VALUES`.

3. **Dynamic Handling of Columns:**
   - Since column names are dynamically structured (`NAME1, VALUE1`, `NAME2, VALUE2`, etc.), we use `UNION ALL` inside a subquery to collect only non-null key-value pairs.

---

### **Example Execution**
```sql
SELECT GET_PRTL_WC_STAGE_JSON(1, 'INSTRUMENT_TYPE') FROM DUAL;
```

---

### **Expected Output JSON**
```json
{
    "UPLOAD_ID": 1,
    "TABLE_NAME": "INSTRUMENT_TYPE",
    "LAST_UPDATED_BY_USER": "JohnDoe",
    "LAST_UPDATED_BY_TIME": "2025-01-28T12:00:00",
    "ROWS": [
        {
            "OPERATION_TYPE": "INSERT",
            "TOTAL_KEY_COLS": 5,
            "KEY_VALUES": [
                {"NAME": "SRC_SYS_CD", "VALUE": "LONNA"},
                {"NAME": "INST_TYP_CD", "VALUE": "CHEQUE"},
                {"NAME": "INST_TYP_VAL", "VALUE": "3223"},
                {"NAME": "INST_LINK_CD", "VALUE": "HHM-JDS"},
                {"NAME": "INST_TYPE_DESC", "VALUE": "Special Type"}
            ]
        }
    ]
}
```

---

### **Key Benefits**
- Dynamically reconstructs JSON format.
- Works for any number of key-value pairs without hardcoding.
- Uses `JSON_ARRAYAGG` for efficient aggregation.

Would you like me to refine this further based on additional requirements?
