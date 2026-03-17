-- name: check_primary_key
SELECT COUNT(1) AS CNT
FROM {table}
WHERE {conditions}

df = service.check_db_primary_key(df, selected_table)


SELECT *
FROM IDENTIFIER(:table_name)

def check_db_primary_key(df, selected_table_en):

    session = get_active_session()

    reader = SqlReader("upload/sql/SC_5_04_01_0.sql")
    runner = SqlRunner(session, reader)

    primary_keys = TABLE_PRIMARY_KEY_MAP[selected_table_en]

    # ---------------------
    # 1 查询 DB 表
    # ---------------------
    db_rows = runner.query(
        "select_table_data",
        {"table_name": selected_table_en}
    ).collect()

    # ---------------------
    # 2 构建 DB 主键集合
    # ---------------------
    db_key_set = set()

    for r in db_rows:
        key_tuple = tuple(str(r[pk]).strip() for pk in primary_keys)
        db_key_set.add(key_tuple)

    # ---------------------
    # 3 Excel 检查
    # ---------------------
    valid_rows = []

    for idx, row in df.iterrows():

        row_num = row["行番号"]
        shori_kbn = str(row.get("SHORI_KBN", "")).strip()

        key_tuple = tuple(str(row.get(pk, "")).strip() for pk in primary_keys)

        is_exist = key_tuple in db_key_set

        # Check4 追加
        if shori_kbn == SHORI_KBN_ADD and is_exist:

            mu.push_messages("459EBR9959", row_num)

            continue

        # Check5 更新 / 削除
        if shori_kbn == SHORI_KBN_UPDATE_DEL and not is_exist:

            mu.push_messages("459EBR9956", row_num)

            continue

        valid_rows.append(row)


        df = service.check_db_primary_key(df, selected_table)

    return pd.DataFrame(valid_rows)
