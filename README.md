for jp_col in immutable_cols:
    en_col = jp_to_en_map.get(jp_col)
    if not en_col:
        continue
    
    # 👇 【关键：在这里打日志，看两个值到底是什么】
    excel_val = row.get(jp_col)
    db_val = db_row_dict.get(en_col.upper())
    # 用st.write打印到前端，或print打日志，二选一
    st.write(f"【レコード{row_num} 項目{jp_col}】Excel入力値: {excel_val} | DB元値: {db_val}")
    # 也可以用print打后台日志：print(f"レコード{row_num} {jp_col}: Excel={excel_val}, DB={db_val}")

    # 标准化比较（原逻辑不变）
    norm_excel = normalize_compare_value(excel_val)
    norm_db = normalize_compare_value(db_val)
    if norm_excel != norm_db:
        changed_cols.append(jp_col)
        st.write(f"【レコード{row_num}】{jp_col} 不一致！Excel={norm_excel}, DB={norm_db}")
