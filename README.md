def check_idu_business_rule(df, selected_table, exist_map):
    valid_rows = []
    business_errors = []

    # 英文 -> 日文
    column_dict = get_column_dict(selected_table)
    # 日文 -> 英文
    jp_to_en_map = {jp: en for en, jp in column_dict.items()}

    # 当前表主键
    primary_map = TABLE_PRIMARY_KEY_MAP[selected_table]
    primary_keys_jp = primary_map["pk_jp"]

    for idx, row in df.iterrows():
        row_num = idx + 1
        shori_kbn = str(row.get("処理区分", "")).strip()

        row_has_error = False

        # ===== I：指定字段必须为空（456ERR998）=====
        if shori_kbn == SHORI_KBN_ADD:
            error_cols = []

            for col in PROHIBITED_FIELDS_I:
                val = row.get(col)

                if val is not None and str(val).strip() != "":
                    error_cols.append(col)

            if error_cols:
                business_errors.append(
                    f"レコード[{row_num}]：456ERR998 入力不可項目に値が設定されています（項目：{', '.join(error_cols)}）"
                )
                row_has_error = True

        # ===== U：指定字段不能改（456ERR999）=====
        if shori_kbn == SHORI_KBN_UPDATE:
            immutable_cols = IMMUTABLE_FIELDS_U_MAP.get(selected_table, [])

            full_key = tuple(str(row.get(jp_pk, "")).strip() for jp_pk in primary_keys_jp)
            db_row_dict = exist_map.get(full_key)

            if db_row_dict and immutable_cols:
                changed_cols = [
                    jp_col
                    for jp_col in immutable_cols
                    if jp_to_en_map.get(jp_col)
                    and normalize_compare_value(row.get(jp_col))
                    != normalize_compare_value(db_row_dict.get(jp_to_en_map[jp_col].upper()))
                ]

                if changed_cols:
                    business_errors.append(
                        f"レコード[{row_num}]：456ERR999 対象外項目が変更されています（項目：{', '.join(changed_cols)}）"
                    )
                    row_has_error = True

        # ===== D：除了処理区分外，其它字段都不能改（456ERR999）=====
        if shori_kbn == SHORI_KBN_DEL:
            full_key = tuple(str(row.get(jp_pk, "")).strip() for jp_pk in primary_keys_jp)
            db_row_dict = exist_map.get(full_key)

            if db_row_dict:
                changed_cols = [
                    jp_col
                    for jp_col in row.index
                    if jp_col != "処理区分"
                    and jp_to_en_map.get(jp_col)
                    and normalize_compare_value(row.get(jp_col))
                    != normalize_compare_value(db_row_dict.get(jp_to_en_map[jp_col].upper()))
                ]

                if changed_cols:
                    business_errors.append(
                        f"レコード[{row_num}]：456ERR999 対象外項目が変更されています（項目：{', '.join(changed_cols)}）"
                    )
                    row_has_error = True

        if not row_has_error:
            valid_rows.append(row)

    return pd.DataFrame(valid_rows), business_errors






    IMMUTABLE_FIELDS_U_MAP = {
    "T456SMMA010": [
        # "基準年月",
        # "契約番号",
    ],
    "T456SMMA020": [],
    "T456SMMA030": [],
    "T456SMMA040": [],
    "T456SMMA050": [],
    "T456SMMA060": [],
    "T456SMMA070": [],
    "T456SMMB010": [],
    "T456SMMA080": [],
    "T456SMMC020": [],
    "T456SMMC040": [],
}
