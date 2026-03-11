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
