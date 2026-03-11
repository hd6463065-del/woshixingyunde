TABLE_PRIMARY_KEY_MAP = {
    "recv_contract_month": ["RECV_KEY"],          # 受信契約明細_月次
    "loan_contract_month": ["LOAN_KEY"],          # 貸出契約明細_月次
    "bond_contract_month": ["BOND_KEY"],          # 債券契約明細_月次
    "swap_contract_month": ["SWAP_KEY"],           # スワップ契約明細_月次
    "stock_asset_contract_month": ["STOCK_KEY"],   # 株・その他資産契約明細_月次
    "option_contract_month": ["OPTION_KEY"],       # オプション契約明細_月次
    "fx_forward_contract_month": ["FX_KEY"],       # 為替予約契約明細_月次
    "gross_profit_detail": ["PROFIT_KEY"],         # 粗利明細_月次
    "risk_set_month": ["RISK_KEY"],                # リスクセット明細_月次
    "market_risk_asset_month": ["MARKET_KEY"],     # マーケットリスクアセット明細_月次
    "other_contract_month": ["OTHER_KEY"],         # その他契約明細_月次（第11张表）
}

# 2. 処理区分定义（按你的实际值调整）
SHORI_KBN_ADD = "1"       # 追加
SHORI_KBN_UPDATE_DEL = "0"# 更新/删除



TABLE_PRIMARY_KEY_MAP = {
    "recv_contract_month": ["RECV_KEY"],          # 受信契約明細_月次
    "loan_contract_month": ["LOAN_KEY"],          # 貸出契約明細_月次
    "bond_contract_month": ["BOND_KEY"],          # 債券契約明細_月次
    "swap_contract_month": ["SWAP_KEY"],           # スワップ契約明細_月次
    "stock_asset_contract_month": ["STOCK_KEY"],   # 株・その他資産契約明細_月次
    "option_contract_month": ["OPTION_KEY"],       # オプション契約明細_月次
    "fx_forward_contract_month": ["FX_KEY"],       # 為替予約契約明細_月次
    "gross_profit_detail": ["PROFIT_KEY"],         # 粗利明細_月次
    "risk_set_month": ["RISK_KEY"],                # リスクセット明細_月次
    "market_risk_asset_month": ["MARKET_KEY"],     # マーケットリスクアセット明細_月次
    "other_contract_month": ["OTHER_KEY"],         # その他契約明細_月次（第11张表）
}

# 2. 処理区分定义（按你的实际值调整）
SHORI_KBN_ADD = "1"       # 追加
SHORI_KBN_UPDATE_DEL = "0"# 更新/删除



# 4.5 項目4+5：上传后立即校验数据库主键（核心）
        if not ss.show_error:
            db_issues = check_db_primary_key_on_upload(df, selected_table_en)
            if db_issues:
                ss.validation_errors.extend(db_issues)
                ss.show_error = True
# ===================== 【項目4+5】上传后数据库主键校验 =====================
def check_db_primary_key_on_upload(df, selected_table_en):
    """
    上传完成后立即校验：
    - 追加(1)：数据库无相同主键 → 通过；有 → 报错459EBR9959
    - 更新/删除(0)：数据库有对应主键 → 通过；无 → 报错459EBR99596
    :param df: 上传的Excel数据（已加行号）
    :param selected_table_en: 选中表英文名
    :return: 错误列表（空=通过）
    """
    issues = []
    
    # 1. 建立DB连接（替换为你的实际配置）
    try:
        db_conn = pymysql.connect(
            host="你的DB地址",
            user="你的DB用户名",
            password="你的DB密码",
            database="你的DB名",
            charset="utf8mb4"
        )
        cursor = db_conn.cursor()
    except Exception as e:
        issues.append(f"DB连接失败：{str(e)}")
        return issues

    # 2. 校验表主键配置
    if selected_table_en not in TABLE_PRIMARY_KEY_MAP:
        issues.append("システムエラー：対象テーブルの主キー設定が存在しません。")
        cursor.close()
        db_conn.close()
        return issues
    
    primary_keys = TABLE_PRIMARY_KEY_MAP[selected_table_en]
    table_name = selected_table_en  # DB表名 = 表英文名

    # 3. 遍历每行校验数据库主键
    for idx, row in df.iterrows():
        row_num = row["行番号"]  # Excel实际行号
        shori_kbn = str(row.get("SHORI_KBN", "")).strip()

        # 跳过処理区分空的行
        if not shori_kbn:
            issues.append(f"未知エラー：{row_num}行：処理区分が空です。")
            continue

        # 3.1 获取主键值（支持复合主键）
        try:
            key_values = [str(row[pk]).strip() for pk in primary_keys]
            if "" in key_values:
                issues.append(f"未知エラー：{row_num}行：主キーが空です。")
                continue
        except KeyError as e:
            issues.append(f"未知エラー：{row_num}行：主キー列「{e}」が存在しません。")
            continue

        # 3.2 拼接主键查询SQL
        key_conditions = " AND ".join([f"{pk} = %s" for pk in primary_keys])
        check_sql = f"SELECT COUNT(*) FROM {table_name} WHERE {key_conditions}"

        # 3.3 追加(1)：检查重复
        if shori_kbn == SHORI_KBN_ADD:
            cursor.execute(check_sql, key_values)
            count = cursor.fetchone()[0]
            if count > 0:
                issues.append(f"459EBR9959: {row_num}行：処理区分「追加」の場合、同一キーのデータが既に存在します。")
                continue  # 跳过该行后续校验

        # 3.4 更新/删除(0)：检查存在
        elif shori_kbn == SHORI_KBN_UPDATE_DEL:
            cursor.execute(check_sql, key_values)
            count = cursor.fetchone()[0]
            if count == 0:
                issues.append(f"459EBR99596: {row_num}行：処理区分「更新/削除」の場合、対象キーのデータが存在しません。")
                continue  # 跳过该行后续校验

        # 3.5 无效処理区分
        else:
            issues.append(f"未知エラー：{row_num}行：無効な処理区分「{shori_kbn}」です。")
            continue

    # 关闭连接
    cursor.close()
    db_conn.close()
    return issues

# ===================== 【項目3】文件列数校验 =====================
def check_file_column_count(df, selected_table_en):
    """項目3：仅校验列数（459EBR99557）"""
    issues = []
    # 补充你的11张表列数配置（示例）
    TABLE_COL_COUNT = {
        "recv_contract_month": 80,
        "loan_contract_month": 75,
        # 其他表列数补充...
    }
    if selected_table_en not in TABLE_COL_COUNT:
        issues.append("459EBR99557: 対象テーブルの項目数設定が存在しません。")
        return issues
    if df.shape[1] != TABLE_COL_COUNT[selected_table_en]:
        issues.append("459EBR99557: アップロードファイルのフォーマット（項目数）が不正です。")
    return issues
