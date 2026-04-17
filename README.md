"check_ok": False,
"check_success_text": ""


ss.hosei_upload["check_ok"] = True
ss.hosei_upload["check_success_text"] = "..."


if ss.hosei_upload["check_ok"] and ss.hosei_upload["check_success_text"]:
    st.success(ss.hosei_upload["check_success_text"])