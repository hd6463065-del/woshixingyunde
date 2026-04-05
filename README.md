if db_row_dict and immutable_cols:
    changed_cols = []
    for jp_col in immutable_cols:
        en_col = jp_to_en_map.get(jp_col)
        if not en_col:
            continue

        # ↓↓↓ ↓↓↓ 【就插在这！！！】 ↓↓↓ ↓↓↓
        # 打印Excel值和DB值，不用任何key，直接打
        st.write(f"行{row_num} {jp_col} → Excel={row.get(jp_col)} | DB={db_row_dict.get(en_col.upper())}")
        # ↑↑↑ ↑↑↑ 【插完了！下面原代码一行不动！】 ↑↑↑ ↑↑↑

        if jp_to_en_map.get(jp_col) and normalize_compare_value(row.get(jp_col)) != normalize_compare_value(db_row_dict.get(jp_to_en_map[jp_col].upper())):
            changed_cols.append(jp_col)
