ss.hosei_upload = {
    "show_error": False,
    "showlog": False,
    "validation_errors": [],
    "valid_data": None,
    "display_errors": [],
    "is_already_checked": False  # 👉 新增：是否已经校验完成
}
# 原来：if upload:
# 改成👇 只有【文件存在 + 还没校验过】才跑校验
if upload and not ss.hosei_upload["is_already_checked"]:

    # ========== 你里面所有原有校验代码 一字不动 ==========

    # 校验全部走完最后，加上这行：标记已经校验完成
    ss.hosei_upload["is_already_checked"] = True




selected_option = st.selectbox(
    "補正対象テーブル",
    options=table_options,
    format_func=lambda x: x[0],
    index=0,
    width=280,
    on_change=lambda : setattr(ss.hosei_upload, "is_already_checked", False)
)

upload = st.file_uploader(...)
# 文件没了 → 重置校验状态
if upload is None:
    ss.hosei_upload["is_already_checked"] = False
