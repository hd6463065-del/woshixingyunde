def insert_hosei_management(uploaded_file, selected_table, correction_reason, user_info):

    # ① 生成補正ID
    result = runner.query("get_next_hosei_id").collect()
    hosei_id = result[0]["NEXT_ID"]

    # ② 文件转二进制（⚠️用这个）
    file_bytes = uploaded_file.getvalue()

    # ③ 参数
    params = {
        "hosei_id": hosei_id,
        "table_name": selected_table,
        "hosei_kbn": "2",  # アップロード
        "file_data": file_bytes,
        "user_id": user_info.get("user_id"),
        "user_name": user_info.get("user_name"),
        "shop_no": user_info.get("shop_no"),
        "correction_reason": correction_reason
    }

    # ④ 执行insert
    runner.query("insert_hosei_management", params).collect()

    return True, hosei_id






    -- name: insert_hosei_management
INSERT INTO HOSEI_MANAGEMENT
(
    補正ID,
    対象テーブル,
    補正方式,
    補正前情報,
    補正後情報,
    アップロードファイル情報,
    処理区分,
    補正日時,
    補正者ID,
    補正者名,
    補正者管理店番,
    補正理由
)
VALUES
(
    :hosei_id,
    :table_name,
    :hosei_kbn,
    NULL,
    NULL,
    :file_data,
    NULL,
    CURRENT_TIMESTAMP(),
    :user_id,
    :user_name,
    :shop_no,
    :correction_reason
);


-- name: get_next_hosei_id
SELECT 
    TO_CHAR(CURRENT_DATE(), 'YYYYMMDD') ||
    LPAD(COALESCE(MAX(SUBSTR(補正ID, 9, 4)), '0')::INTEGER + 1, 4, '0') AS NEXT_ID
FROM HOSEI_MANAGEMENT
WHERE SUBSTR(補正ID, 1, 8) = TO_CHAR(CURRENT_DATE(), 'YYYYMMDD')
;


success = service.insert_hosei_management(
    uploaded_file=ss.uploaded_file,
    selected_table=selected_table,
    correction_reason=ss.correction_reason,
    user_info=user_info
)
