def clear_check_success():
    ss = st.session_state
    ss.hosei_upload["check_ok"] = False
    ss.hosei_upload["check_success_text"] = ""