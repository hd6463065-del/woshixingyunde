def normalize_compare_value(val):
    """比较时统一转成字符串，None / NaN 按空处理"""
    if val is None:
        return ""

    try:
        if pd.isna(val):
            return ""
    except Exception:
        pass

    if isinstance(val, Decimal):
        return str(val)

    return str(val).strip()




    def get_column_dict(table) -> dict:
    table_info = TblInfos.TABLE_MAP[table].cols
    all_cols = [col for col in table_info.__dict__.values() if isinstance(col, TblCol)]

    physical_list = []
    logical_list = []

    for col in all_cols:
        physical_list.append(col.col_en)
        logical_list.append(col.col_jp)

    column_dict = dict(zip(physical_list, logical_list))
    return column_dict





    def check_idu_business_rule(df, selected_table, exist_map):
    valid_rows = []
    business_errors = []

    for idx, row in df.iterrows():
        row_num = idx + 1
        shori_kbn = str(row.get("処理区分", "")).strip()

        row_has_error = False

        # I：指定字段必须为空（456ERR998）
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

        if not row_has_error:
            valid_rows.append(row)

    return pd.DataFrame(valid_rows), business_errors
