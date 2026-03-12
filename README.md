import streamlit as st
import pandas as pd
import common
import checks  # 导入你的校验核心模块
import message_util as mu  # 导入你的消息工具模块
import excel_file_util
import io
import re
import random
from datetime import datetime
from validation_rules import validation_rules  # 导入字段的校验规则

# 页面基础配置
st.set_page_config(layout="wide")
st.markdown("""
<style>
    div[data-testid="stAlert"] {
        padding: 4px 8px !important;
        margin-top: 2px !important;
        margin-bottom: 2px !important;
    }
    div[data-testid="stAlert"] div:first-child {
        display: flex;
        align-items: top;
        gap: 6px;
    }
    div[data-testid="stAlert"] p {
        font-size: 14px;
        line-height: 1;
        margin: 0.5em 0;
    }
    .error_display {
        border: 1px solid #E0E0E0;
        border-radius: 10px;
        padding: 10px;
        height: 300px;
        overflow-y: auto;
        background: #fff;
    }
    .error_item {
        color: #d32f2f;
        font-size: 14px;
        line-height: 1.5;
        margin: 2px 0;
        white-space: pre-wrap;
    }
</style>
""", unsafe_allow_html=True)

# 初始化会话状态
ss = st.session_state

# 初始化必要的会话状态变量
if "show" not in ss:
    ss.show = False
if "upload" not in ss:
    ss.upload = None
if "show_error" not in ss:
    ss.show_error = False
if "showlog" not in ss:
    ss.showlog = False
if "validation_errors" not in ss:
    ss.validation_errors = []
if "valid_data" not in ss:
    ss.valid_data = None
if "kijundate" not in ss:
    ss.kijundate = datetime.now().strftime("%Y%m")
if "display_errors" not in ss:
    ss.display_errors = []
if "correction_reason" not in ss:
    ss.correction_reason = ""

# ====================== 【核心配置】各表对应列数常量 ======================
# 1. 定义各表的列数常量 (命名规则: const_<表英文名>_total_col)
const_recv_contract_month_total_col = 157      # 受信契約明細_月次 (4表)
const_loan_contract_month_total_col = 146       # 貸出契約明細_月次
const_bond_contract_month_total_col = 111      # 債券契約明細_月次
const_stock_asset_contract_month_total_col = 78 # 株・その他資産契約明細_月次
const_swap_contract_month_total_col = 80       # スワップ契約明細_月次
const_option_contract_month_total_col = 63      # オプション契約明細_月次
const_fx_forward_contract_month_total_col = 69 # 為替予約契約明細_月次
const_gross_profit_detail_total_col = 78        # 粗利明細
const_frame_contract_month_total_col = 87      # 枠契約明細_月次
const_risk_set_month_total_col = 46             # リスクセット明細_月次
const_market_risk_asset_month_total_col = 49   # マーケットリスクアセット明細_月次

# 2. 映射字典: 表英文名 -> 对应的列数常量
TABLE_COL_CONST_MAP = {
    "recv_contract_month": const_recv_contract_month_total_col,
    "loan_contract_month": const_loan_contract_month_total_col,
    "bond_contract_month": const_bond_contract_month_total_col,
    "stock_asset_contract_month": const_stock_asset_contract_month_total_col,
    "swap_contract_month": const_swap_contract_month_total_col,
    "option_contract_month": const_option_contract_month_total_col,
    "fx_forward_contract_month": const_fx_forward_contract_month_total_col,
    "gross_profit_detail": const_gross_profit_detail_total_col,
    "frame_contract_month": const_frame_contract_month_total_col,
    "risk_set_month": const_risk_set_month_total_col,
    "market_risk_asset_month": const_market_risk_asset_month_total_col
}

# 3. 定义各表的主键 (修正拼写错误: NEISAI → MEISAI)
TABLE_PRIMARY_KEY_MAP = {
    "recv_contract_month": ["KEIJUN_YM", "JUSHIN_KYK_MEISAI_KEY", "HOSEI_VER"],
    "loan_contract_month": ["KEIJUN_YM", "KASHIDASHI_KYK_MEISAI_KEY", "HOSEI_VER"],
    "bond_contract_month": ["KEIJUN_YM", "SAIKEN_KYK_MEISAI_KEY", "HOSEI_VER"],
    "stock_asset_contract_month": ["KEIJUN_YM", "KABU_SONOTASHISAN_KYK_MEISAI_KEY", "HOSEI_VER"],
    "swap_contract_month": ["KEIJUN_YM", "SWAP_KYK_MEISAI_KEY", "HOSEI_VER"],
    "option_contract_month": ["KEIJUN_YM", "OP_KYK_MEISAI_KEY", "HOSEI_VER"],
    "fx_forward_contract_month": ["KEIJUN_YM", "FX_KYK_MEISAI_KEY", "HOSEI_VER"],
    "gross_profit_detail": ["ARARI_MEISAI_KEY", "HOSEI_VER"],
    "frame_contract_month": ["KEIJUN_YM", "WAKU_MEISAI_KEY", "HOSEI_VER"],
    "risk_set_month": ["SANTEI_KEIJUN_YM", "ANKEN_NO", "KOKYAKU_NO", "HOSHONIN_KOKYAKU_NO", "ON_OFF_KBN", "HOSEI_VER"],
    "market_risk_asset_month": ["SANTEI_KEIJUN_YM", "JIKO_CO_CD", "KOKYAKU_NO", "SA_CCR_KEISANYO_TRHKMEISAI_NO", "HOSHONIN_KOKYAKU_NO", "HOSEI_VER"]
}

# 4. 処理区分定義
SHORI_KBN_ADD = "1"    # 追加
SHORI_KBN_UPDATE_DEL = "0" # 更新/削除

# ====================== 核心: 按式样要求拆分错误信息 ======================
def create_error_excel(errors):
    """
    严格按「レコード x: 错误内容」拆分列
    输出列: エラー行番号 / エラー内容
    """
    error_rows = []
    pattern = re.compile(r"レコード\s*(\d+)\s*: ")

    for error_msg in errors:
        match = pattern.search(error_msg)
        if match:
            row_num = match.group(1)
            error_content = pattern.sub("", error_msg).strip()
        else:
            row_num = ""
            error_content = error_msg

        error_rows.append({
            "エラー行番号": row_num,
            "エラー内容": error_content
        })

    error_df = pd.DataFrame(error_rows)
    output = io.BytesIO()
    with pd.ExcelWriter(output, engine="openpyxl") as writer:
        error_df.to_excel(writer, sheet_name="エラーリスト", index=False)
    output.seek(0)
    return output

# ====================== 【项目4+5】DB主键校验 ======================
def check_db_primary_key(df, selected_table_en):
    """
    核心逻辑: 仅做DB主键校验
    - 项目4: 追加(1) → 校验主键是否重复
    - 项目5: 更新/删除(0) → 校验主键是否存在
    """
    issues = []
    primary_keys = TABLE_PRIMARY_KEY_MAP[selected_table_en]

    try:
        db_conn = common.get_db_connection()
        cursor = db_conn.cursor()
    except Exception as e:
        mu.push_messages("456ERR9999", f"DB连接失败: {str(e)}")
        return issues

    for idx, row in df.iterrows():
        row_num = row.get("行番号", idx+1)  # 兼容无行番号的情况
        shori_kbn = str(row.get("SHORI_KBN", "")).strip()

        if not shori_kbn:
            mu.push_messages("456ERR0003", row_num)
            continue

        try:
            key_values = [str(row[pk]).strip() for pk in primary_keys]
            if "" in key_values:
                mu.push_messages("456ERR0004", row_num, ",".join(primary_keys))
                continue
        except KeyError as e:
            mu.push_messages("456ERR0005", row_num, str(e))
            continue

        key_conditions = " AND ".join([f"{pk} = %s" for pk in primary_keys])
        check_sql = f"SELECT COUNT(*) FROM `{selected_table_en}` WHERE {key_conditions}"

        try:
            cursor.execute(check_sql, key_values)
            count = cursor.fetchone()[0]
            is_key_exist = count > 0

            if shori_kbn == SHORI_KBN_ADD:
                if is_key_exist:
                    mu.push_messages("459EBR9959", row_num)
            elif shori_kbn == SHORI_KBN_UPDATE_DEL:
                if not is_key_exist:
                    mu.push_messages("459EBR99596", row_num)
            else:
                mu.push_messages("456ERR0006", row_num, shori_kbn)
        except Exception as e:
            mu.push_messages("456ERR9999", f"{row_num}行 → {str(e)}")
            continue

    cursor.close()
    db_conn.close()
    return issues

# ====================== 【登録処理】基礎データ補正管理テーブル書込 ======================
def insert_hosei_management_data(df, selected_table_en, correction_reason, user_info):
    """写入基础データ補正管理テーブル"""
    HOSEI_TABLE = "基礎データ補正管理"

    try:
        db_conn = common.get_db_connection()
        cursor = db_conn.cursor()
        db_conn.autocommit = False
    except Exception as e:
        return False, mu.push_messages("456ERR9999", f"DB连接失败: {str(e)}")

    try:
        hosei_id = f"{datetime.now().strftime('%Y%m%d%H%M%S')}{random.randint(1000, 9999)}"

        table_ja_name = {
            "recv_contract_month": "受信契約明細_月次",
            "loan_contract_month": "貸出契約明細_月次",
            "bond_contract_month": "債券契約明細_月次",
            "stock_asset_contract_month": "株・その他資産契約明細_月次",
            "swap_contract_month": "スワップ契約明細_月次",
            "option_contract_month": "オプション契約明細_月次",
            "fx_forward_contract_month": "為替予約契約明細_月次",
            "gross_profit_detail": "粗利明細",
            "frame_contract_month": "枠契約明細_月次",
            "risk_set_month": "リスクセット明細_月次",
            "market_risk_asset_month": "マーケットリスクアセット明細_月次"
        }.get(selected_table_en, selected_table_en)

        for idx, row in df.iterrows():
            shori_kbn = str(row.get("SHORI_KBN", "")).strip()
            hosei_hoshiki = "追加" if shori_kbn == "1" else "更新/削除" if shori_kbn == "0" else "不明"
            primary_keys = TABLE_PRIMARY_KEY_MAP[selected_table_en]
            key_values = "_".join([str(row[pk]).strip() for pk in primary_keys])

            insert_sql = f"""
            INSERT INTO {HOSEI_TABLE} (
                補正ID, 対象テーブル英名, 対象テーブル和名, 補正方式,
                レコードキー値, 処理区分, 補正事由,
                補正者ID, 補正者名, 補正者管理店番,
                補正日時, 削除フラグ
            ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
            """
            cursor.execute(insert_sql, [
                hosei_id,
                selected_table_en,
                table_ja_name,
                hosei_hoshiki,
                key_values,
                shori_kbn,
                correction_reason,
                user_info["user_id"],
                user_info["user_name"],
                user_info["shop_no"],
                datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                "0"
            ])

        db_conn.commit()
        mu.push_messages("456INF0001", hosei_id)
        return True, "登録成功"
    except Exception as e:
        db_conn.rollback()
        mu.push_messages("456EBR0070", str(e))
        return False, "登録失敗"
    finally:
        cursor.close()
        db_conn.close()

# ====================== 列数校验函数 ======================
def check_file_column_count(df, selected_table_en):
    """动态校验Excel列数"""
    issues = []
    col_count = df.shape[1]
    const_total_col = TABLE_COL_CONST_MAP[selected_table_en]

    if col_count != const_total_col:
        mu.push_messages("459EBR99557", const_total_col, col_count)
        issues.append(mu.get_message("459EBR99557", const_total_col, col_count))
    return issues

# ====================== 弹窗定义 ======================
@st.dialog("", dismissible=False, width="small")
def confirm():
    e1, e2 = st.columns([0.4, 2])
    with e2:
        st.write("\u3000\u3000\u3000\u3000補正データの登録を行いますか？")
    a1, a2, a3 = st.columns([0.5, 0.4, 1])
    with a2:
        if st.button("はい", type="primary"):
            user_info = {
                "user_id": ss.get("user_id", "UNKNOWN"),
                "user_name": ss.get("user_name", "匿名"),
                "shop_no": ss.get("shop_no", "0000")
            }
            success, msg = insert_hosei_management_data(
                df=ss.valid_data,
                selected_table_en=selected_table_en,
                correction_reason=correction_reason,
                user_info=user_info
            )

            if success:
                ss.showlog = False
                mu.push_messages("456INF0002")
                ss.valid_data = None
                ss.validation_errors = []
                ss.correction_reason = ""
                st.rerun()
            else:
                ss.showlog = False
                st.rerun()
    with a3:
        if st.button("いいえ"):
            ss.showlog = False
            st.rerun()

# ====================== 页面UI ======================
st.subheader("法人DN 基礎データ補正 (アップロード)")
with st.container(border=True):
    st.write("")
    st.write("")
    w1, w2 = st.columns([0.2, 1])
    with w1:
        table_options = [
            ("選択してください", ""),
            ("受信契約明細_月次", "recv_contract_month"),
            ("貸出契約明細_月次", "loan_contract_month"),
            ("債券契約明細_月次", "bond_contract_month"),
            ("株・その他資産契約明細_月次", "stock_asset_contract_month"),
            ("スワップ契約明細_月次", "swap_contract_month"),
            ("オプション契約明細_月次", "option_contract_month"),
            ("為替予約契約明細_月次", "fx_forward_contract_month"),
            ("粗利明細", "gross_profit_detail"),
            ("枠契約明細_月次", "frame_contract_month"),
            ("リスクセット明細_月次", "risk_set_month"),
            ("マーケットリスクアセット明細_月次", "market_risk_asset_month"),
        ]

        selected_option = st.selectbox(
            "補正対象テーブル",
            options=table_options,
            format_func=lambda x: x[0],
            index=0,
            width=280
        )
        selected_table_en = selected_option[1]
    with w2:
        correction_reason = st.text_input("補正事由", value=ss.correction_reason, width=280)
        ss.correction_reason = correction_reason  # 保存到会话状态

    st.write("")
    st.write("ドラッグ&ドロップまたは、「Browse files」ボタンを押下してファイルを選択してください。※自動でファイル内容のチェック処理が開始されます。")
    upload = st.file_uploader("アップロード", type=["xlsx", "xls"], label_visibility="collapsed")

# ====================== 核心逻辑: 校验流程 ======================
if upload:
    ss.show_error = False
    ss.validation_errors = []
    mu.clear_messages()  # 清空历史消息

    # 1. 前置校验
    if not selected_table_en:
        mu.push_messages("456ERR0001")
        ss.show_error = True
    elif not correction_reason:
        mu.push_messages("456ERR0002")
        ss.show_error = True
    else:
        try:
            # 2. 读取Excel文件 (修正拼写错误: read_vb_as_df → read_wb_as_df)
            df = excel_file_util.read_wb_as_df(upload)
            
            # 3. 処理区分列存在性校验
            if "SHORI_KBN" not in df.columns:
                mu.push_messages("459ERR0002")
                ss.show_error = True
            else:
                # 4. 列数校验
                col_issues = check_file_column_count(df, selected_table_en)
                if col_issues:
                    ss.validation_errors.extend(col_issues)
                    ss.show_error = True
                else:
                    # 5. DB主键校验
                    db_issues = check_db_primary_key(df, selected_table_en)
                    if db_issues:
                        ss.validation_errors.extend(db_issues)
                        ss.show_error = True
                    else:
                        # 6. 公共字段校验
                        df = df.reset_index(drop=True)
                        current_table_rules = validation_rules.get(selected_table_en, {})
                        ss.validation_errors = checks.validate_dataframe(df, list(current_table_rules.items()))

            # 7. 处理校验结果
            if len(ss.validation_errors) == 0:
                ss.valid_data = df
                mu.push_messages("456INF0003")
                ss.show_error = False
            else:
                ss.show_error = True
                display_errors = ss.validation_errors[:10]
                if len(ss.validation_errors) > 10:
                    display_errors.append(mu.get_message("456INF0004", len(ss.validation_errors)))
                ss.display_errors = display_errors
                mu.push_messages("456ERR0007")
        except Exception as e:
            mu.push_messages("456ERR9999", str(e))
            ss.show_error = True

# 显示公共消息
mu.show_messages()

# ====================== 错误列表展示 ======================
if ss.show_error and ss.display_errors:
    error_display = st.container()
    with error_display:
        parts = [f"<div class='error_item'>{e}</div>" for e in ss.display_errors]
        item_html = "".join(parts)
        st.markdown(f"<div class='error_display'>{item_html}</div>", unsafe_allow_html=True)

# ====================== 按钮区域 ======================
st.write("")
st.write("")

c1, c2, c3 = st.columns([3, 0.8, 1])
with c1:
    if ss.show_error and len(ss.validation_errors) > 0:
        error_file = create_error_excel(ss.validation_errors)
        st.download_button(
            label="チェック結果ダウンロード",
            data=error_file,
            file_name=f"データチェックエラー_{datetime.now().strftime('%Y%m%d%H%M%S')}.xlsx",
            mime="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
            type="primary"
        )
    else:
        st.button("チェック結果ダウンロード", type="primary", disabled=True)

with c3:
    upload_condition = upload is not None
    reason_condition = bool(correction_reason)
    valid_condition = len(ss.validation_errors) == 0
    can_submit = upload_condition and reason_condition and valid_condition

    if st.button("補正データの登録", type="primary", disabled=not can_submit):
        ss.showlog = True

# 显示确认弹窗 (补充缺失的调用逻辑)
if ss.showlog:
    confirm()
