validation_rules = {
    # 0: 処理区分
    0: {"is_required": True, "max_len": 1},
    # 1: 基準年月
    1: {"is_required": True, "max_len": 6},
    # 2: スワップ契約明細キー
    2: {"is_required": True, "max_len": 100},
    # 3: データ基準年月
    3: {"max_len": 6},
    # 4: BS店
    4: {"max_len": 3},
    # 5: 法人番号
    5: {"max_len": 6},
    # 6: 統合顧客ID
    6: {"max_len": 10},
    # 7: 取引番号
    7: {"max_len": 30},
    # 8: 管理店舗
    8: {"max_len": 3},
    # 9: 取引先分類
    9: {"max_len": 3},
    # 10: 確定コード
    10: {"max_len": 4},
    # 11: BT区分
    11: {"max_len": 3},
    # 12: 取引拠点
    12: {"max_len": 4},
    # 13: 法人DM科目コード
    13: {"max_len": 50},
    # 14: 種別
    14: {"max_len": 50},
    # 15: 約定日
    15: {"max_len": 8},
    # 16: 取引開始日
    16: {"max_len": 8},
    # 17: 取引終了日
    17: {"max_len": 8},
    # 18: Payサイト_通貨コード
    18: {"max_len": 3},
    # 19: Payサイト_金利タイプ
    19: {"max_len": 4},
    # 20: Payサイト_金利サイクル
    20: {"max_len": 2},
    # 21: Payサイト_次回利払日
    21: {"max_len": 8},
    # 22: Payサイト_当初想定元本
    22: {"type": "number", "common_check": "number_20_3"},
    # 23: Payサイト_現在想定元本
    23: {"type": "number", "common_check": "number_20_3"},
    # 24: Payサイト_最大想定元本
    24: {"type": "number", "common_check": "number_20_3"},
    # 25: Payサイト_満期時想定元本
    25: {"type": "number", "common_check": "number_20_3"},
    # 26: Payサイト_計算根拠
    26: {"max_len": 8},
    # 27: Payサイト_現在適用金利
    27: {"type": "number", "common_check": "number_14_7"},
    # 28: Payサイト_為替レート
    28: {"type": "number", "common_check": "number_14_7"},
    # 29: Payサイト_現在スプレッド
    29: {"type": "number", "common_check": "number_14_7"},
    # 30: Revサイト_通貨コード
    30: {"max_len": 3},
    # 31: Revサイト_金利タイプ
    31: {"max_len": 4},
    # 32: Revサイト_金利サイクル
    32: {"max_len": 2},
    # 33: Revサイト_次回利払日
    33: {"max_len": 23},
    # 34: Revサイト_当初想定元本
    34: {"type": "number", "common_check": "number_20_3"},
    # 35: Revサイト_現在想定元本
    35: {"type": "number", "common_check": "number_20_3"},
    # 36: Revサイト_最大想定元本
    36: {"type": "number", "common_check": "number_20_3"},
    # 37: Revサイト_満期時想定元本
    37: {"type": "number", "common_check": "number_20_3"},
    # 38: Revサイト_計算根拠
    38: {"max_len": 8},
    # 39: Revサイト_現在適用金利
    39: {"type": "number", "common_check": "number_14_7"},
    # 40: Revサイト_為替レート
    40: {"type": "number", "common_check": "number_14_7"},
    # 41: Revサイト_現在スプレッド
    41: {"type": "number", "common_check": "number_14_7"},
    # 42: 内貨換算レート
    42: {"type": "number", "common_check": "number_28_19"},
    # 43: Payサイト_時価原通貨
    43: {"type": "number", "common_check": "number_20_3"},
    # 44: Payサイト_時価HC
    44: {"type": "number", "common_check": "number_20_3"},
    # 45: Revサイト_時価原通貨
    45: {"type": "number", "common_check": "number_20_3"},
    # 46: Revサイト_時価HC
    46: {"type": "number", "common_check": "number_20_3"},
    # 47: 評価通貨コード
    47: {"max_len": 50},
    # 48: 評価額_HC
    48: {"type": "number", "common_check": "number_20_3"},
    # 49: 評価額_円
    49: {"type": "number", "common_check": "number_17_0"},
    # 50: FamilyID
    50: {"max_len": 14},
    # 51: 名寄法人番号
    51: {"max_len": 6},
    # 52: Payサイト_月中想定元本積算_原通貨
    52: {"type": "number", "common_check": "number_20_3"},
    # 53: Revサイト_月中想定元本積算_原通貨
    53: {"type": "number", "common_check": "number_20_3"},
    # 54: Payサイト_月中マージン積算_原通貨
    54: {"type": "number", "common_check": "number_20_3"},
    # 55: Revサイト_月中マージン積算_原通貨
    55: {"type": "number", "common_check": "number_20_3"},
    # 56: Payサイト_月中平均想定元本_原通貨
    56: {"type": "number", "common_check": "number_20_3"},
    # 57: Revサイト_月中平均想定元本_原通貨
    57: {"type": "number", "common_check": "number_20_3"},
    # 58: Payサイト_月中平均マージ_原通貨
    58: {"type": "number", "common_check": "number_20_3"},
    # 59: Revサイト_月中平均マージ_原通貨
    59: {"type": "number", "common_check": "number_20_3"},
    # 60: 繰延解除フラグ
    60: {"max_len": 1},
    # 61: 補正バージョン
    61: {"max_len": 2},
    # 62: 補正事由
    62: {"max_len": 50},
    # 63: 補正日
    63: {"max_len": 8},
    # 64: 補正者
    64: {"max_len": 100},
    # 65: 補正確認者
    65: {"max_len": 100},
    # 66: 登録日
    66: {"max_len": 8},
    # 67: 登録者
    67: {"max_len": 100},
    # 68: 登録確認者
    68: {"max_len": 100},
    # 69: データ原分類コード
    69: {"max_len": 50},
    # 70: データ原テーブルファイルコード
    70: {"max_len": 50},
    # 71: データ原テーブルファイル内明細番号
    71: {"max_len": 58},
    # 72: 元システムキー_TradingAreaID
    72: {"max_len": 4},
    # 73: 元システムキー_InstrumentID
    73: {"max_len": 13},
    # 74: 元システムキー_OriginalID
    74: {"max_len": 30},
    # 75: 元システムキー_DataSource
    75: {"max_len": 20},
    # 76: 元システムキー_TradeID
    76: {"max_len": 23},
    # 77: Bronzeデータベースパラメータ_panorama索引_検査
    77: {"max_len": 1024},
    # 78: Bronzeデータベースパラメータ_スキーマ_データソース
    78: {"max_len": 1024},
    # 79: Bronzeデータベースパラメータ_Calypso_全利回り
    79: {"max_len": 1024}
}
