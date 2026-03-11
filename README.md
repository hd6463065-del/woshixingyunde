import streamlit as st
import pandas as pd
import common
import checks  # 导入你的校验核心模块
import message_util  # 导入你的消息工具模块
import io
import re
from datetime import datetime

# 页面基础配置
st.set_page_config(layout="wide")
st.markdown("""
<style>
        div[data-testid="stAlert"] {
        padding: 4px 8px !important;
        margin-top:2px !important;
        margin-bottom:2px !important;}

        div[data-testid = "stAlert"]> div:first-child {
        display:flex;
        align-items:top;
        gap:6px;}
        div[data-testid="stAlert"] p {
        font-size:14px;
        line-height:1;
        margin:0.50;}
        
        .error_display{
        border:1px solid #E0E0E0; border-radius:10px; padding:10px; height:300px;
        overflow-y: auto; background:#fff;
        }
        .error_item{
        color:#d32f2f;
        font-size:14px;
        line-height:1.5;
        margin:2px 0;
        white-space:pre-wrap;
        }
</style>
""", unsafe_allow_html=True)

# 初始化会话状态
ss = st.session_state
flash = common.flash()

# 初始化必要的会话状态变量
if "show" not in ss:
    ss.show = False
if "upload" not in ss:
    ss.upload = False
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

# ===================== 导入按英文key分类的校验规则 =====================
from validation_rules import validation_rules as all_table_rules

# ===================== 核心：按式样要求拆分错误信息 =====================
def create_error_excel(errors):
    """
    严格按「レコード X：错误内容」拆分列
    输出列：エラー行番号 / エラー内容
    格式不匹配时：エラー行番号为空，エラー内容保留原始值
    """
    error_rows = []
    # 正则匹配「レコード + 数字 + ：」的格式（兼容空格）
    pattern = re.compile(r'レコード\s*(\d+)\s*：')
    
    for error_msg in errors:
        # 匹配行番号
        match = pattern.search(error_msg)
        if match:
            row_num = match.group(1)  # 提取数字行号
            # 提取错误内容（去掉「レコードX：」前缀）
            error_content = pattern.sub('', error_msg).strip()
        else:
            # 格式不匹配时：行番号为空，内容保留原始值
            row_num = ""
            error_content = error_msg
        
        error_rows.append({
            "エラー行番号": row_num,
            "エラー内容": error_content
        })
    
    # 生成Excel
    error_df = pd.DataFrame(error_rows)
    output = io.BytesIO()
    with pd.ExcelWriter(output, engine="openpyxl") as writer:
        error_df.to_excel(writer, sheet_name="エラーリスト", index=False)
    output.seek(0)
    return output

# ===================== 页面UI =====================
st.subheader("法人DM 基礎データ補正（アップロード）")
with st.container(border=True):
    st.write("")
    st.write("")
    w1, w2 = st.columns([0.2, 1])
    with w1:
        # 核心修改：下拉框配置（日文显示 + 英文key）
        table_options = [
            ("選択してください", ""),  # 默认选项
            ("受信契約明細_月次", "recv_contract_month"),
            ("貸出契約明細_月次", "loan_contract_month"),
            ("債券契約明細_月次", "bond_contract_month"),
            ("株・その他資産契約明細_月次", "stock_asset_contract_month"),
            ("スワップ契約明細_月次", "swap_contract_month"),
            ("オプション契約明細_月次", "option_contract_month"),
            ("為替予約契約明細_月次", "fx_forward_contract_month"),
            ("粗利明細", "gross_profit_detail"),
            ("リスクセット明細_月次", "risk_set_month"),
            ("マーケットリスクアセット明細_月次", "market_risk_asset_month"),
        ]
        
        # 下拉框：画面显示日文，内部获取英文key
        selected_option = st.selectbox(
            "補正対象テーブル",
            options=table_options,
            format_func=lambda x: x[0],  # 仅显示日文部分
            index=0,  # 默认选中第一个（選択してください）
            width=280
        )
        # 提取英文key（后续代码用这个）
        selected_table_en = selected_option[1]
    with w2:
        correction_reason = st.text_input("補正事由", width=280)

st.write("")
st.write("")
st.write("ドラッグ&ドロップまたは、「Browse files」ボタンを押下してファイルを選択してください。※自動でファイル内容のチェック処理が開始されます。")
upload = st.file_uploader("アップロード", type=["xlsx", "xls"], label_visibility="collapsed")

# ===================== 核心逻辑：上传Excel → 读取 → 校验 =====================
if upload:
    # 重置状态
    ss.show_error = False
    ss.validation_errors = []
    flash.session_state[flash.key].clear()

    # 1. 前置校验：対象テーブル + 補正事由 必須チェック
    if not selected_table_en:  # 检查英文key是否为空（即未选表）
        flash.push("対象テーブルは必須です、選択してください。", "error")
    elif not correction_reason:
        flash.push("補正事由は必須です、入力してください。", "error")
    else:
        try:
            # 读取Excel文件
            df = pd.read_excel(upload)
            df = df.reset_index(drop=True)

            # 2. 动态获取当前选中表的英文key对应的规则
            current_table_rules = all_table_rules.get(selected_table_en, {})
            # 直接调用校验函数
            ss.validation_errors = checks.validate_dataframe(df, list(current_table_rules.items()))

            # 处理校验结果
            if len(ss.validation_errors) == 0:
                ss.valid_data = df
                flash.push("ファイル内容のチェックが完了しました、全てのデータが正常です。", "success")
                ss.show_error = False
            else:
                ss.show_error = True
                display_errors = ss.validation_errors[:10]
                if len(ss.validation_errors) > 10:
                    display_errors.append(f"※合計{len(ss.validation_errors)}件のエラーがあります。「チェック結果ダウンロード」で全件確認してください。")
                ss.display_errors = display_errors
                flash.push("ファイルに誤りがあります。詳細をご確認のうえ、修正後に再アップロードしてください。", "error")

        except Exception as e:
            flash.push(f"システムエラー：{str(e)}", "error")
            ss.show_error = True

# 显示flash消息
flash.show()

# ===================== 错误列表展示 =====================
if ss.show_error and "display_errors" in ss:
    error_display = st.container()
    with error_display:
        parts = [f"<div class='error_item'>{e}</div>" for e in ss.display_errors]
        item_html = "".join(parts)
        st.markdown(f"<div class='error_display'>{item_html}</div>", unsafe_allow_html=True)

# ===================== 按钮区域 & 确认对话框 =====================
st.write("")
st.write("")

@st.dialog(" ", dismissible=False, width="small")
def confirm():
    e1, e2 = st.columns([0.4, 2])
    with e2:
        st.write("\u3000\u3000\u3000\u3000補正データの登録を行いますか？")
    a1, a2, a3 = st.columns([0.5, 0.4, 1])
    with a2:
        if st.button("はい", type="primary"):
            ss.showlog = False
            st.success("補正データの登録が完了しました。精査者へ精査依頼をしてください。")
            st.rerun()
    with a3:
        if st.button("いいえ"):
            ss.showlog = False
            st.rerun()

# 按钮布局
c1, c2, c3 = st.columns([3, 0.8, 1])
with c1:
    # 下载完整错误列表
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
    # 登録按钮启用规则
    upload_condition = upload is not None
    reason_condition = bool(correction_reason)
    valid_condition = len(ss.validation_errors) == 0
    can_submit = upload_condition and reason_condition and valid_condition

    if st.button("補正データの登録", type="primary", disabled=not can_submit):
        ss.showlog = True

# 显示确认对话框
if ss.showlog:
    confirm()
