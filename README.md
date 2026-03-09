# app10.py
import streamlit as st
import pandas as pd
import common

# 页面配置
st.set_page_config(layout="wide")

# 自定义样式
st.markdown("""
<style>
div[data-testid="stAlert"] {
    padding: 4px 8px !important;
    margin-top: 2px !important;
    margin-bottom: 2px !important;
}
div[data-testid="stAlert"] > div:first-child {
    display: flex;
    align-items: top;
    gap: 6px;
}
div[data-testid="stAlert"] p {
    font-size: 14px;
    line-height: 1;
    margin: 0.5rem;
}
</style>
""", unsafe_allow_html=True)

# 初始化会话状态
ss = st.session_state
flash = common.flash()

# 初始化状态变量
if "show" not in ss:
    ss.show = False
if "upload" not in ss:
    ss.upload = False
if "show_error" not in ss:
    ss.show_error = False
if "showlog" not in ss:
    ss.showlog = False

# 页面标题
st.subheader("法人DM 基礎データ補正（アップロード）")

# 主容器
with st.container(border=True):
    st.write("")
    st.write("")
    v1, v2 = st.columns([0.2, 1])
    with v1:
        target_table = st.selectbox(
            "補正対象テーブル",
            ["", "粗利明細", "リスクセッ", "回収予定"],
            width=200
        )
    with v2:
        reason = st.text_input("補正事由", width=200)

    st.write("")
    st.write("ドラッグ&ドロップまたは「Browse files」ボタンを押下してファイルを選択してください。※自動でファイル内容のチェック処理が実施されます")
    uploaded_file = st.file_uploader(
        "アップロード",
        type=["xlsx", "xls"],
        label_visibility="collapsed"
    )

    # 文件上传逻辑
    if uploaded_file:
        if not reason:
            flash.push("補正事由は必須です。入力してください。", "error")
        else:
            # 这里可以添加文件验证逻辑
            flash.push("ファイル内容のチェックが完了しました。アップロードデータの登録ボタンを押下してアップロードを実行してください。", "success")
            ss.show_error = True
            ss.upload = True

    # 显示错误信息
    if ss.show_error:
        flash.push("ファイルに誤りがあります。詳細は以下のエラーリストをご確認いただき、修正のうえ再度ファイルの選択をお願いします。\n\n(エラーリスト)", "error")
        errors = [
            "1. レコード: 1, 【契約明細キー】は補正できません。",
            "2. レコード: 3, 【基準年月日】は補正できません。",
            "3. レコード: 5, 【契約明細キー】は補正できません。",
            "4. レコード: 7, 【取引開始日】は必須項目です。",
            "5. レコード: 7, 【取引開始日】の日付形式が不正です。YYYY/MM/DD形式で入力してください。",
            "6. レコード: 9, 【取引終了日】は必須項目です。",
            "7. レコード: 9, 【取引終了日】の日付形式が不正です。YYYY/MM/DD形式で入力してください。",
            "8. レコード: 11, 【契約明細キー】は補正できません。",
            "9. レコード: 13, 【基準年月日】は補正できません。",
            "10. レコード: 15, 【契約明細キー】は補正できません。"
        ]

        # 错误信息展示容器
        error_display = st.container()
        with error_display:
            st.markdown("""
            <style>
            .error_display {
                border: 1px solid #E0E0E0;
                border-radius: 10px;
                padding: 16px;
                height: 300px;
                overflow-y: auto;
                background: #FFF;
            }
            .error_item {
                color: #D32F2F;
                font-size: 13px;
                margin: 2px 0;
                white-space: nowrap;
                overflow: hidden;
                text-overflow: ellipsis;
            }
            </style>
            """, unsafe_allow_html=True)
            parts = [f"<div class='error_item'>{e}</div>" for e in errors]
            item_html = "".join(parts)
            st.markdown(f"<div class='error_display'>{item_html}</div>", unsafe_allow_html=True)

    st.write("")
    st.write("")

    # 确认对话框
    @st.dialog("", dismissible=False, width="small")
    def confirm():
        c1, c2 = st.columns([6, 4])
        with c2:
            st.write("補正データの登録を実行しますか？")
            a1, a2, a3 = st.columns([0.5, 0.4, 0.1])
            with a2:
                if st.button("はい", type="primary"):
                    # 这里可以添加数据注册的逻辑
                    ss.showlog = False
                    st.rerun()
            with a3:
                if st.button("いいえ"):
                    ss.showlog = False
                    st.rerun()

    # 底部按钮
    c1, c2, c3 = st.columns([3, 0.8, 1])
    with c1:
        if st.button("チェック結果ダウンロード", type="primary", disabled=not ss.show_error):
            # 这里可以添加下载错误结果的逻辑
            pass
    with c3:
        if st.button("補正データの登録", type="primary", disabled=False):
            ss.showlog = True

    if ss.showlog:
        confirm()

# 显示所有 Flash 消息
flash.show()
