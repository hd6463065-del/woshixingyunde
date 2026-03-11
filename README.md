# validation_rules.py - 按英文key管理各表的校验规则
# 结构：{"英文key": {字段索引/字段名: 校验规则}, ...}
# 可根据实际业务补充/修改规则内容

validation_rules = {
    # 受信契約明細_月次
    "recv_contract_month": {
        0: {"is_required": True, "type": "str", "max_len": 20},  # 契約明細キー
        1: {"is_required": True, "type": "int", "max_len": 6},   # 基準年月
        2: {"is_required": False, "type": "date", "format": "%Y/%m/%d"},  # 取引開始日
        3: {"is_required": False, "type": "float"}  # 受信金額
    },
    # 貸出契約明細_月次
    "loan_contract_month": {
        0: {"is_required": True, "type": "str", "max_len": 20},  # 契約明細キー
        1: {"is_required": True, "type": "int", "max_len": 6},   # 基準年月
        3: {"is_required": True, "type": "float"},  # 貸出金額
        4: {"is_required": False, "type": "str", "max_len": 50}   # 貸出先名
    },
    # 債券契約明細_月次
    "bond_contract_month": {
        0: {"is_required": True, "type": "str", "max_len": 20},  # 契約明細キー
        1: {"is_required": True, "type": "int", "max_len": 6},   # 基準年月
        4: {"is_required": True, "type": "str", "max_len": 50},  # 債券名
        5: {"is_required": False, "type": "float"}  # 債券額
    },
    # 株・その他資産契約明細_月次
    "stock_asset_contract_month": {
        0: {"is_required": True, "type": "str", "max_len": 20},  # 契約明細キー
        1: {"is_required": True, "type": "int", "max_len": 6},   # 基準年月
        6: {"is_required": True, "type": "str", "max_len": 50},  # 銘柄名
        7: {"is_required": False, "type": "int"}  # 保有数量
    },
    # スワップ契約明細_月次
    "swap_contract_month": {
        0: {"is_required": True, "type": "str", "max_len": 20},  # 契約明細キー
        1: {"is_required": True, "type": "int", "max_len": 6},   # 基準年月
        2: {"is_required": False, "type": "date", "format": "%Y/%m/%d"},  # 取引開始日
        8: {"is_required": False, "type": "float"}  # スワップ金額
    },
    # オプション契約明細_月次
    "option_contract_month": {
        0: {"is_required": True, "type": "str", "max_len": 20},  # 契約明細キー
        1: {"is_required": True, "type": "int", "max_len": 6},   # 基準年月
        9: {"is_required": True, "type": "str", "max_len": 20},  # オプション種別
        10: {"is_required": False, "type": "float"}  # 権利金
    },
    # 為替予約契約明細_月次
    "fx_forward_contract_month": {
        0: {"is_required": True, "type": "str", "max_len": 20},  # 契約明細キー
        1: {"is_required": True, "type": "int", "max_len": 6},   # 基準年月
        11: {"is_required": True, "type": "str", "max_len": 3},  # 通貨コード
        12: {"is_required": False, "type": "float"}  # 予約金額
    },
    # 粗利明細
    "gross_profit_detail": {
        0: {"is_required": True, "type": "str", "max_len": 20},  # 契約明細キー
        1: {"is_required": True, "type": "int", "max_len": 6},   # 基準年月
        13: {"is_required": True, "type": "float"}  # 粗利金額
    },
    # リスクセット明細_月次
    "risk_set_month": {
        0: {"is_required": True, "type": "str", "max_len": 20},  # 契約明細キー
        1: {"is_required": True, "type": "int", "max_len": 6},   # 基準年月
        14: {"is_required": True, "type": "float"}  # リスク額
    },
    # マーケットリスクアセット明細_月次
    "market_risk_asset_month": {
        0: {"is_required": True, "type": "str", "max_len": 20},  # 契約明細キー
        1: {"is_required": True, "type": "int", "max_len": 6},   # 基準年月
        15: {"is_required": True, "type": "float"}  # リスクアセット額
    }
}
