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
flash = common.flash()

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

# 3. 定义各表的主键
TABLE_PRIMARY_KEY_MAP = {
    "recv_contract_month": ["KEIJUN_YM", "JUSHIN_KYK_NEISAI_KEY", "HOSEI_VER"],
    "loan_contract_month": ["KEIJUN_YM", "KASHIDASHI_KYK_NEISAI_KEY", "HOSEI_VER"],
    "bond_contract_month": ["KEIJUN_YM", "SAIKEN_KYK_NEISAI_KEY", "HOSEI_VER"],
    "stock_asset_contract_month": ["KEIJUN_YM", "KABU_SONOTASHISAN_KYK_NEISAI_KEY", "HOSEI_VER"],
    "swap_contract_month": ["KEIJUN_YM", "SWAP_KYK_NEISAI_KEY", "HOSEI_VER"],
    "option_contract_month": ["KEIJUN_YM", "OP_KYK_NEISAI_KEY", "HOSEI_VER"],
    "fx_forward_contract_month": ["KEIJUN_YM", "FX_KYK_NEISAI_KEY", "HOSEI_VER"],
    "gross_profit_detail": ["ARARI_NEISAI_KEY", "HOSEI_VER"],
    "risk_set_month": ["KEIJUN_YM", "WARU_NEISAI_KEY", "HOSEI_VER"],
    "market_risk_asset_month": ["SANTEI_KEIJUN_YM", "ANKEN_NO", "KOKYAKU_NO", "HOSHONIN_KOKYAKU_NO", "ON_OFF_KBN", "HOSEI_VER"],
    "other_contract_month": ["SANTEI_KEIJUN_YM", "JIKO_CO_CD", "KOKYAKU_NO", "SA_CCR_KEISANYO_TRHKMEISAI_NO", "KOSHONIN_KOKYAKU_NO", "HOSEI_VER"]
}

# 2. 処理区分定義 (按你的实际调整)
SHORI_KBN_ADD = "1"    # 追加
SHORI_KBN_UPDATE_DEL = "0" # 更新/削除

# # Snowpark セッション取得
# session = get_active_session()
# # SQL Reader
# reader = SqlReader("pages/test.sql")
# # 存入session (给后续登录/业务逻辑用), 不画面显示
# ss.user_name = reader["user_name"]
# ss.shop_no = reader["shop_no"]

# ====================== 核心: 按式样要求拆分错误信息 (移除「不明」) ======================
def create_error_excel(errors):
    """
    严格按「レコード x: 错误内容」拆分列
    输出列: エラー行番号 / エラー内容
    """
    error_rows = []
    # 正则匹配「レコード + 数字 + :」的格式 (兼容空格)
    pattern = re.compile(r"レコード\s*(\d+)\s*: ")

    for error_msg in errors:
        # 匹配行番号
        match = pattern.search(error_msg)
        if match:
            row_num = match.group(1)  # 提取数字行号
            # 提取错误内容 (去掉「レコード x: 」前缀)
            error_content = pattern.sub("", error_msg).strip()
        else:
            # 格式不匹配时: 行番号为空, 内容保留原始值 (不填「不明」)
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

# ====================== 【项目4+5】DB主键校验 (精简版) ======================
def check_db_primary_key(df, selected_table_en):
    """
    核心逻辑: 仅做DB主键校验 (表选择前置已校验,无需重复半判断)
    - 项目4: 追加(1) → 校验主键是否重复
    - 项目5: 更新/删除(0) → 校验主键是否存在
    :param df: 上传的Excel数据 (含行号)
    :param selected_table_en: 选中表英文名 (前置已校验非空且有效)
    :return: 错误列表 (空=通过)
    """
    issues = []

    # 1. 直接获取已定义的主键 (无需判断表是否存在, 前置已校验)
    primary_keys = TABLE_PRIMARY_KEY_MAP[selected_table_en]

    # 2. 建立DB连接 (替换为你项目的真实连接)
    try:
        # ↓↓↓ 替换为你的真实DB连接 ↓↓↓
        db_conn = common.get_db_connection()  # 公共DB连接函数
        # 或测试用:
        # db_conn = pymysql.connect(
        #     host="你的DB地址",
        #     port=你的DB端口,
        #     user="你的DB用户名",
        #     password="你的DB密码",
        #     database="你的DB名",
        #     charset="utf8mb4"
        # )
        cursor = db_conn.cursor()
    except Exception as e:
        issues.append(f"DB连接失败: {str(e)}")
        return issues

    # 3. 遍历每行执行DB校验
    for idx, row in df.iterrows():
        row_num = row["行番号"]  # Excel行号
        shori_kbn = str(row.get("SHORI_KBN", "")).strip()

        # 跳过処理区分为空的行
        if not shori_kbn:
            issues.append(f"未知エラー: {row_num}行: 処理区分が空です。")
            continue

        # 3.1 获取主键值 (支持复合主键)
        try:
            key_values = [str(row[pk]).strip() for pk in primary_keys]
            if "" in key_values:  # 主键为空校验
                issues.append(f"未知エラー: {row_num}行: 主キーが空です。")
                continue
        except KeyError as e:  # 主键列不存在
            issues.append(f"未知エラー: {row_num}行: 主キー列「{e}」が存在しません。")
            continue

        # 3.2 拼接主键查询SQL (自动适配复合主键)
        table_name = selected_table_en
        key_conditions = " AND ".join([f"{pk} = %s" for pk in primary_keys])
        check_sql = f"SELECT COUNT(*) FROM `{table_name}` WHERE {key_conditions}"

        try:
            # 执行DB查询
            cursor.execute(check_sql, key_values)
            count = cursor.fetchone()[0]
            is_key_exist = count > 0

            # 项目4: 追加(1) → 主键重复校验
            if shori_kbn == SHORI_KBN_ADD:
                if is_key_exist:
                    issues.append(f"459EBR9959: {row_num}行: 処理区分「追加」の場合、同一キーのデータが既に存在します。")

            # 项目5: 更新/删除(0) → 主键存在校验
            elif shori_kbn == SHORI_KBN_UPDATE_DEL:
                if not is_key_exist:
                    issues.append(f"459EBR99596: {row_num}行: 処理区分「更新/削除」の場合、対象キーのデータが存在しません。")

            # 无效処理区分
            else:
                issues.append(f"未知エラー: {row_num}行: 無効な処理区分「{shori_kbn}」です。")
        except Exception as e:
            issues.append(f"DB查询错误: {row_num}行 → {str(e)}")
            continue

    # 关闭连接
    cursor.close()
    db_conn.close()

    return issues

# ====================== 【登録処理】基礎データ補正管理テーブル書込 ======================
def insert_hosei_management_data(df, selected_table_en, correction_reason, user_info):
    """
    写入基础データ補正管理テーブル (设计书要求)
    :param df: 校验通过的Excel数据
    :param selected_table_en: 选中表英文名
    :param correction_reason: 補正事由
    :param user_info: 用户信息 (dict, 包含user_id, user_name, shop_no等)
    :return: (是否成功, 消息/错误)
    """
    # 1. 定义補正管理テーブル字段 (按设计书)
    HOSEI_TABLE = "基礎データ補正管理"  # 替换为你的实际表名 (英文/日文)

    # 2. 建立DB连接
    try:
        db_conn = common.get_db_connection()
        cursor = db_conn.cursor()
        db_conn.autocommit = False  # 开启事务
    except Exception as e:
        return False, f"DB连接失败: {str(e)}"

    try:
        # 3. 生成補正ID (示例: 年月日+随机数, 可替换为你的规则)
        hosei_id = f"{datetime.now().strftime('%Y%m%d%H%M%S')}{random.randint(1000, 9999)}"

        # 4. 遍历数据批量插入
        table_ja_name = {  # 表英文名→日文名映射 (按你的实际配置)
            "recv_contract_month": "受信契約明細_月次",
            "loan_contract_month": "貸出契約明細_月次",
            # ... 补充其他表的日文名 ...
        }.get(selected_table_en, selected_table_en)

        for idx, row in df.iterrows():
            # 4.1 组装登録数数据 (按设计书字段)
            shori_kbn = str(row.get("SHORI_KBN", "")).strip()
            # 補正方式 (追加/更新/削除)
            hosei_hoshiki = "追加" if shori_kbn == "1" else "更新/削除" if shori_kbn == "0" else "不明"
            # 主键值 (拼接复合主键)
            primary_keys = TABLE_PRIMARY_KEY_MAP[selected_table_en]
            key_values = "_".join([str(row[pk]).strip() for pk in primary_keys])

            # 4.2 插入SQL (按你的实际表字段调整)
            insert_sql = f"""
            INSERT INTO {HOSEI_TABLE} (
                補正ID, 対象テーブル英名, 対象テーブル和名, 補正方式,
                レコードキー値, 処理区分, 補正事由,
                補正者ID, 補正者名, 補正者管理店番,
                補正日時, 削除フラグ
            ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
            """
            # 4.3 执行插入
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
                "0"  # 0=未删除, 1=已删除
            ])

        # 5. 提交事务
        db_conn.commit()
        return True, f"登録成功! 補正ID: {hosei_id}"
    except Exception as e:
        # 失败回滚
        db_conn.rollback()
        error_msg = f"456EBR0070: 登録失敗 → {str(e)}"  # 设计书错误ID
        return False, error_msg
    finally:
        # 关闭连接
        cursor.close()
        db_conn.close()

# ====================== 列数校验函数 ======================
def check_file_column_count(df, selected_table_en):
    """
    动态校验Excel列数: 根据选中的表获取对应列数常量, 列数不一致时添加456ERR4041错误
    :param df: 读取的Excel数据
    :param selected_table_en: 选中表的英文名
    :return: (issues: 错误列表, df: 原始数据)
    """
    issues = []
    col_count = df.shape[1]  # Excel实际列数
    const_total_col = TABLE_COL_CONST_MAP[selected_table_en]

    # 2. 列数校验 (和你图里逻辑完全一致)
    if col_count != const_total_col:
        # 错误文案完全复用你图里的格式: 456ERR4041
        issues.append(f"456ERR4041: アップロードファイルの項目数が正しくありません。本フォーマットの規定項目数: {const_total_col} 項目を設定してください。(col_count: {col_count})")
    return issues

# ====================== 页面UI ======================
st.subheader("法人DN 基礎データ補正 (アップロード)")
with st.container(border=True):
    st.write("")
    st.write("")
    w1, w2 = st.columns([0.2, 1])
    with w1:
        # 核心修改: 下拉框配置 (日文显示 + 英文key)
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
            ("枠契約明細_月次", "frame_contract_month"),
            ("リスクセット明細_月次", "risk_set_month"),
            ("マーケットリスクアセット明細_月次", "market_risk_asset_month"),
        ]

        # 下拉框: 画面显示日文, 内部获取英文key
        selected_option = st.selectbox(
            "補正対象テーブル",
            options=table_options,
            format_func=lambda x: x[0],  # 仅显示日文部分
            index=0,  # 默认选中第一个 (選択してください)
            width=280
        )
        # 提取英文key (后续代码用这个)
        selected_table_en = selected_option[1]
    with w2:
        correction_reason = st.text_input("補正事由", width=280)

    st.write("")
    st.write("ドラッグ&ドロップまたは、「Browse files」ボタンを押下してファイルを選択してください。※自動でファイル内容のチェック処理が開始されます。")
    upload = st.file_uploader("アップロード", type=["xlsx", "xls"], label_visibility="collapsed")

# ====================== 核心逻辑: 调用checks.validate_dataframe ======================
if upload:
    # 重置状态
    ss.show_error = False
    ss.validation_errors = []
    st.session_state["flash"].clear()

    # 1. 前置校验: 対象テーブル + 補正事由 必須チェック
    if not selected_table_en:  # 检查英文key是否为空 (即未选表)
        mu.push_messages("456ERR0001")
    elif not correction_reason:
        mu.push_messages("456ERR0002")
    else:
        try:
            # 2. 读取Excel文件
            df = excel_file_util.read_vb_as_df(upload)
            # 【補充】処理区分 (SHORI_KBN) 列存在性チェック
            if "SHORI_KBN" not in df.columns:
                mu.push_messages("459ERR0002")
                ss.show_error = True
            else:
                inssus = check_file_column_count(df, selected_table_en)
                if inssus:
                    ss.validation_errors.extend(inssus)
                    ss.show_error = True
                else:
                    db_issues = check_db_primary_key(df, selected_table_en)
                    if db_issues:
                        ss.validation_errors.extend(db_issues)
                        ss.show_error = True
                    else:
                        df = df.reset_index(drop=True)
                        # 2. 动态获取当前选中表的英文key对应的规则
                        current_table_rules = validation_rules.get(selected_table_en, {})
                        # 直接调用校验函数
                        ss.validation_errors = checks.validate_dataframe(df, list(current_table_rules.items()))

            # 4. 处理校验结果
            if len(ss.validation_errors) == 0:
                ss.valid_data = df
                flash.push("ファイル内容のチェックが完了しました。全てのデータが正常です。", "success")
                ss.show_error = False
            else:
                ss.show_error = True
                display_errors = ss.validation_errors[:10]
                if len(ss.validation_errors) > 10:
                    display_errors.append(f"※合計{len(ss.validation_errors)}件のエラーがあります。「チェック結果ダウンロード」で全件確認してください。")
                ss.display_errors = display_errors
                flash.push("ファイルに誤りがあります。詳細をご確認のうえ、修正後に再アップロードしてください。", "error")
        except Exception as e:
            flash.push(f"システムエラー: {str(e)}", "error")
            ss.show_error = True

# 显示flash消息
mu.show_messages()

# ====================== 错误列表展示 ======================
if ss.show_error and "display_errors" in ss:
    error_display = st.container()
    with error_display:
        parts = [f"<div class='error_item'>{e}</div>" for e in ss.display_errors]
        item_html = "".join(parts)
        st.markdown(f"<div class='error_display'>{item_html}</div>", unsafe_allow_html=True)

# ====================== 按钮区域 & 确认对话框 ======================
st.write("")
st.write("")

@st.dialog("補正データ登録確認", dismissible=False, width="small")
def confirm():
    e1, e2 = st.columns([0.4, 2])
    with e2:
        st.write("\u3000\u3000\u3000\u3000補正データの登録を行いますか？")
    a1, a2, a3 = st.columns([0.5, 0.4, 1])
    with a2:
        if st.button("はい", type="primary"):
            # 新增: 调用登録函数 (核心)
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
                # 登録成功: 保持你原来的成功提示
                ss.showlog = False
                st.success("補正データの登録が完了しました。精査者へ精査依頼をしてください。")
                # 重置状态 (清空数据)
                ss.valid_data = None
                ss.validation_errors = []
                ss.correction_reason = ""
                st.rerun()
            else:
                # 登録失敗: 显示错误
                st.error(f"登録失敗: {msg}")
                ss.showlog = False  # 关闭弹窗
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
