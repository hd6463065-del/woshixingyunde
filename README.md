# ===================== 【核心配置】各表对应的列数常量（和你图里const_Fmt1_total_col格式一致） =====================
# 1. 定义各表的列数常量（命名规则：const_<表英文名>_total_col）
const_recv_contract_month_total_col = 80    # 受信契约明细_月度（A表）：80列
const_loan_contract_month_total_col = 75    # 贷出契约明细_月度：75列
const_bond_contract_month_total_col = 68    # 债券契约明细_月度：68列
const_swap_contract_month_total_col = 72    # 掉期契约明细_月度：72列

# 2. 映射字典：表英文名 → 对应的列数常量（动态获取用）
TABLE_COL_CONST_MAP = {
    "recv_contract_month": const_recv_contract_month_total_col,
    "loan_contract_month": const_loan_contract_month_total_col,
    "bond_contract_month": const_bond_contract_month_total_col,
    "swap_contract_month": const_swap_contract_month_total_col,
}
# ===================== 【补充8】动态列数校验函数（复用456ERR4041错误码） =====================
def check_file_column_count(df, selected_table_en):
    """
    动态校验Excel列数：根据选中的表，获取对应列数常量，列数不一致时添加456ERR4041错误
    :param df: 读取的Excel数据
    :param selected_table_en: 选中表的英文名
    :return: (issues: 错误列表, df: 原始数据)
    """
    issues = []
    col_count = df.shape[1]  # Excel实际列数

    # 1. 获取选中表对应的列数常量
    if selected_table_en not in TABLE_COL_CONST_MAP:
        issues.append(f"456ERR4041: 対象テーブル「{selected_table_en}」の列数設定が存在しません。")
        return issues, df
    
    const_total_col = TABLE_COL_CONST_MAP[selected_table_en]

    # 2. 列数校验（和你图里逻辑完全一致）
    if col_count != const_total_col:
        # 错误文案完全复用你图里的格式：456ERR4041: アップロードファイルの項目数が正しくありません...
        issues.append(
            f"456ERR4041: アップロードファイルの項目数が正しくありません。"
            f"本フォーマットの規定項目数は {const_total_col} 項目です。"
            f"現在のファイル項目数は {col_count} 項目です。"
        )

    # 3. 返回错误列表 + 原始数据
    return issues, df








     # 【核心】调用动态列数校验函数（和你图里逻辑一致）
                issues, df = check_file_column_count(df, selected_table_en)
                
                # 如果有列数错误，把issues加入到ss.validation_errors里
                if issues:
                    ss.validation_errors.extend(issues)
                    ss.show_error = True
                    flash.push("456ERR4041: アップロードファイルの項目数が正しくありません。", "error")
                    return  # 中断后续流程
