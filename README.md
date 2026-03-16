def check_record_existence(session, df, selected_table, primary_keys):

    table_df = session.table(selected_table)

    valid_rows = []
    skipped_rows = []

    for idx, row in df.iterrows():

        row_no = idx + 2
        shori_kbn = str(row["SHORI_KBN"]).strip()

        # 构造主键查询条件
        condition = None

        for pk in primary_keys:

            value = row[pk]

            if condition is None:
                condition = table_df[pk] == value
            else:
                condition = condition & (table_df[pk] == value)

        # Snowpark 查询
        exists = table_df.filter(condition).count() > 0

        # --------------------------
        # Check4 : 追加
        # --------------------------
        if shori_kbn == SHORI_KBN_ADD:

            if exists:

                skipped_rows.append(
                    f"レコード{row_no}: 主キー重複のためスキップ"
                )

                continue

            valid_rows.append(row)
            continue

        # --------------------------
        # Check5 : 更新 / 削除
        # --------------------------
        if shori_kbn in (SHORI_KBN_UPDATE, SHORI_KBN_DELETE):

            if not exists:

                skipped_rows.append(
                    f"レコード{row_no}: 主キー不存在のためスキップ"
                )

                continue

            valid_rows.append(row)

    return valid_rows, skipped_rows



valid_rows, skipped_rows = check_record_existence(
    session,
    df,
    selected_table,
    primary_keys
)

valid_df = pd.DataFrame(valid_rows)




SELECT COUNT(*)
FROM {table}
WHERE primary_key = ?


INSERT INTO table (...)
VALUES (...)
