# ===================== 【項目4+5】DB主键校验（精简版） =====================
def check_db_primary_key(df, selected_table_en):
    """
    核心逻辑：仅做DB主键校验（表选择前置已校验，无需重复判断）
    - 項目4：追加(1) → 校验主键是否重复
    - 項目5：更新/删除(0) → 校验主键是否存在
    :param df: 上传的Excel数据（含行号）
    :param selected_table_en: 选中表英文名（前置已校验非空且有效）
    :return: 错误列表（空=通过）
    """
    issues = []

    # 1. 直接获取已定义的主键（无需判断表是否存在，前置已校验）
    primary_keys = TABLE_PRIMARY_KEY_MAP[selected_table_en]

    # 2. 建立DB连接（替换为你项目的真实连接）
    try:
        # ↓↓↓ 替换为你的真实DB连接 ↓↓↓
        db_conn = common.get_db_connection()  # 公共DB连接函数
        # 或测试用：
        # db_conn = pymysql.connect(
        #     host="你的DB地址",
        #     user="你的DB用户名",
        #     password="你的DB密码",
        #     database="你的DB名",
        #     charset="utf8mb4"
        # )
        cursor = db_conn.cursor()
    except Exception as e:
        issues.append(f"DB连接失败：{str(e)}")
        return issues

    # 3. 遍历每行执行DB校验
    for idx, row in df.iterrows():
        row_num = row["行番号"]  # Excel行号
        shori_kbn = str(row.get("SHORI_KBN", "")).strip()

        # 跳过処理区分空的行
        if not shori_kbn:
            issues.append(f"未知エラー：{row_num}行：処理区分が空です。")
            continue

        # 3.1 获取主键值（支持复合主键）
        try:
            key_values = [str(row[pk]).strip() for pk in primary_keys]
            if "" in key_values:  # 主键为空校验
                issues.append(f"未知エラー：{row_num}行：主キーが空です。")
                continue
        except KeyError as e:  # 主键列不存在
            issues.append(f"未知エラー：{row_num}行：主キー列「{e}」が存在しません。")
            continue

        # 3.2 拼接主键查询SQL（自动适配复合主键）
        table_name = selected_table_en
        key_conditions = " AND ".join([f"{pk} = %s" for pk in primary_keys])
        check_sql = f"SELECT COUNT(*) FROM `{table_name}` WHERE {key_conditions}"

        try:
            # 执行DB查询
            cursor.execute(check_sql, key_values)
            count = cursor.fetchone()[0]
            is_key_exist = count > 0

            # --------------------------
            # 項目4：追加(1) → 主键重复校验
            # --------------------------
            if shori_kbn == "1":
                if is_key_exist:
                    issues.append(f"459EBR9959: {row_num}行：処理区分「追加」の場合、同一キーのデータが既に存在します。")

            # --------------------------
            # 項目5：更新/删除(0) → 主键存在校验
            # --------------------------
            elif shori_kbn == "0":
                if not is_key_exist:
                    issues.append(f"459EBR99596: {row_num}行：処理区分「更新/削除」の場合、対象キーのデータが存在しません。")

            # 无效処理区分
            else:
                issues.append(f"未知エラー：{row_num}行：無効な処理区分「{shori_kbn}」です。")

        except Exception as e:
            issues.append(f"DB查询错误：{row_num}行 → {str(e)}")
            continue

    # 关闭连接
    cursor.close()
    db_conn.close()

    return issues



    # --------------------------
                        # 3. 項目4+5：DB主键校验（核心调用，无冗余）
                        # --------------------------
                        db_issues = check_db_primary_key(df, selected_table_en)
                        if db_issues:
                            ss.validation_errors.extend(db_issues)
                            ss.show_error = True
