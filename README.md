import pandas as pd

def check_db_primary_key(df, selected_table_en):
    issues = []
    valid_rows = []  # 存校验通过的行
    primary_keys = TABLE_PRIMARY_KEY_MAP[selected_table_en]
    
    # --------------------------
    # 👇 假DB：模拟已存在的主键集合（你随便改这里测试）
    # --------------------------
    # 格式：{ (主键1值, 主键2值, ...), ... }  对应你的复合主键
    MOCK_EXISTING_KEYS = {
        ("TEST001", "202401"),  # 示例1：已存在的复合主键
        ("TEST002", "202402"),  # 示例2：已存在的复合主键
        # 你可以多加几个来测试不同场景
    }

    # 原DB连接代码暂时保留（之后替换真查询时用）
    # db_conn = common.get_db_connection()
    # cursor = db_conn.cursor()

    for row_num, row in df.iterrows():
        # 1. 处理区分校验（保持你原来的逻辑）
        shori_kbn = str(row.get("SHORI_KBN", "")).strip()
        if not shori_kbn:
            issues.append(f"未知エラー：{row_num}行：処理区分が空です。")
            continue

        # 2. 主键值获取 + 非空校验（保持你原来的逻辑）
        try:
            key_values = []
            for pk in primary_keys:
                val = str(row[pk]).strip()
                if not val:
                    issues.append(f"未知エラー：{row_num}行：主キー「{pk}」が空です。")
                    key_values = []
                    break
                key_values.append(val)
            if not key_values:
                continue
        except KeyError as e:
            issues.append(f"未知エラー：{row_num}行：主キー列「{e}」が存在しません。")
            continue

        # --------------------------
        # 👇 假DB查询核心：直接用mock集合判断
        # --------------------------
        # 把key_values转成tuple，和MOCK_EXISTING_KEYS里的元素类型一致
        key_tuple = tuple(key_values)
        is_key_exist = key_tuple in MOCK_EXISTING_KEYS  # ✅ 假结果

        # 👇 原来的真DB查询代码暂时注释掉，之后恢复
        # table_name = selected_table_en
        # key_conditions = " AND ".join([f"`{pk}` = %s" for pk in primary_keys])
        # check_sql = f"SELECT COUNT(*) FROM `{table_name}` WHERE {key_conditions}"
        # try:
        #     cursor.execute(check_sql, key_values)
        #     count = cursor.fetchone()[0]
        #     is_key_exist = count > 0
        # except Exception as e:
        #     issues.append(f"DBクエリエラー：{row_num}行 → {str(e)}")
        #     continue

        # 3. Check4：追加(1) → 主键重复校验
        if shori_kbn == "1":
            if is_key_exist:
                issues.append(f"459EBR99596：{row_num}行：処理区分「追加」の場合、同一キーのデータが既に存在します。")
                continue  # 跳过整行

        # 4. Check5：更新/删除(0) → 主键存在校验
        elif shori_kbn == "0":
            if not is_key_exist:
                issues.append(f"459EBR99596：{row_num}行：処理区分「更新/削除」の場合、対象キーのデータが存在しません。")
                continue  # 跳过整行

        # 5. 无效处理区分
        else:
            issues.append(f"未知エラー：{row_num}行：無効な処理区分「{shori_kbn}」です。")
            continue  # 跳过整行

        # ✅ 所有校验通过 → 加入合格数据
        valid_rows.append(row)

    # 原DB关闭代码暂时保留
    # cursor.close()
    # db_conn.close()

    # 返回合格数据 + 错误列表
    valid_df = pd.DataFrame(valid_rows)
    return valid_df, issues
