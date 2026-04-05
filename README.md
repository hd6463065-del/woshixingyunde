for jp_col in immutable_cols:
    en_col = jp_to_en_map.get(jp_col)
    if not en_col:
        continue

    # ↓↓↓ 【只加这4行，其他全不动！】
    st.write(f"【行{row_num} {jp_col}】")
    st.write(f"jp_to_en_map映射: {jp_col} → {en_col}")
    st.write(f"en_col.upper(): {en_col.upper()}")
    st.write(f"db_row_dict的key列表: {list(db_row_dict.keys())}")
    st.write(f"最终取到的DB值: {db_row_dict.get(en_col.upper())}")
    # ↑↑↑ 【插完了】
