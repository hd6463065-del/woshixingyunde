import pandas as pd

def check_db_primary_key(df, selected_table_en):
    issues = []
    valid_rows = []  # 新增：存校验通过的行
    primary_keys = TABLE_PRIMARY_KEY_MAP[selected_table_en]
    db_conn = common.get_db_connection()
    cursor = db_conn.cursor()

    # 【修改2】：加 row_num 对应原文件行号
    for row_num, row in df.iterrows():
        # 【修改3】：严谨获取处理区分
        shori_kbn = str(row.get("SHORI_KBN", "")).strip()
        if not shori_kbn:
            issues.append(f"未知エラー：{row_num}行：処理区分が空です。")
            continue

        # 【修改4】：修复主键为空校验
        try:
            key_values = []
            for pk in primary_keys:
                val = str(row[pk]).strip()
                if not val:
                    issues.append(f"未知エラー：{row_num}行：主キー「{pk}」が空です。")
                    key_values = []
                    break
            if not key_values:
                continue
        except KeyError as e:
            issues.append(f"未知エラー：{row_num}行：主キー列「{e}」が存在しません。")
            continue

        # DB查询（保留你原逻辑）
        table_name = selected_table_en
        key_conditions = " AND ".join([f"`{pk}` = %s" for pk in primary_keys])
        check_sql = f"SELECT COUNT(*) FROM `{table_name}` WHERE {key_conditions}"
        try:
            cursor.execute(check_sql, key_values)
            count = cursor.fetchone()[0]
            is_key_exist = count > 0
        except Exception as e:
            issues.append(f"DBクエリエラー：{row_num}行 → {str(e)}")
            continue

        # Check4：追加(1) - 主键重复
        if shori_kbn == "1":
            if is_key_exist:
                issues.append(f"459EBR99596：{row_num}行：処理区分「追加」の場合、同一キーのデータが既に存在します。")
                continue  # 【修改5-1】：跳过整行

        # Check5：更新/删除(0) - 主键存在
        elif shori_kbn == "0":
            if not is_key_exist:
                issues.append(f"459EBR99596：{row_num}行：処理区分「更新/削除」の場合、対象キーのデータが存在しません。")
                continue  # 【修改5-2】：跳过整行

        # 无效处理区分
        else:
            issues.append(f"未知エラー：{row_num}行：無効な処理区分「{shori_kbn}」です。")
            continue  # 【修改5-3】：跳过整行

        # 【修改6】：所有校验通过才加入合格数据
        valid_rows.append(row)

    cursor.close()
    db_conn.close()
    # 转换合格数据为DataFrame并返回
    valid_df = pd.DataFrame(valid_rows)
    return valid_df, issues  # 【修改1】：返回合格数据+错误
