def check_db_primary_key(df, selected_table_en):

    issues = []

    primary_keys = TABLE_PRIMARY_KEY_MAP[selected_table_en]

    db_conn = common.get_db_connection()
    cursor = db_conn.cursor()

    for idx, row in df.iterrows():

        row_num = row["行番号"]
        shori_kbn = str(row.get("SHORI_KBN", "")).strip()

        if not shori_kbn:
            issues.append(f"未知エラー: {row_num}行: 処理区分が空です。")
            continue

        # 主键值
        try:
            key_values = [str(row[pk]).strip() for pk in primary_keys]
        except KeyError as e:
            issues.append(f"未知エラー: {row_num}行: 主キー列 [{e}] が存在しません。")
            continue

        if "" in key_values:
            issues.append(f"未知エラー: {row_num}行: 主キーが空です。")
            continue

        # 生成 where 条件
        key_conditions = " AND ".join([f"{pk}=%s" for pk in primary_keys])

        check_sql = f"""
        SELECT COUNT(1)
        FROM {selected_table_en}
        WHERE {key_conditions}
        """

        cursor.execute(check_sql, key_values)

        count = cursor.fetchone()[0]

        is_key_exist = count > 0

        # -----------------------------
        # Check4 追加（1）
        # -----------------------------
        if shori_kbn == SHORI_KBN_ADD:

            if is_key_exist:
                issues.append(
                    f"459EBR9959: {row_num}行: 処理区分「追加」の場合、同一キーのデータが既に存在します。"
                )

        # -----------------------------
        # Check5 更新 / 削除（0）
        # -----------------------------
        elif shori_kbn == SHORI_KBN_UPDATE_DEL:

            if not is_key_exist:
                issues.append(
                    f"459EBR9956: {row_num}行: 処理区分「更新/削除」の場合、対象キーのデータが存在しません。"
                )

        else:
            issues.append(
                f"未知エラー: {row_num}行: 無効な処理区分「{shori_kbn}」です。"
            )

    cursor.close()
    db_conn.close()

    return issues




    issues = service.check_db_primary_key(df, selected_table_en)
