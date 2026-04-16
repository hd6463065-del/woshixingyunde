ss.hosei_upload = {
    "show_error": False,
    "showlog": False,
    "validation_errors": [],
    "valid_data": None,
    "display_errors": [],
    "success_msg_shown": False,
}



if flag and len(ss.hosei_upload["validation_errors"]) == 0:
    ss.hosei_upload["valid_data"] = df
    if not ss.hosei_upload["success_msg_shown"]:
        mu.push_messages("456INF0012")




if success:
    ss.hosei_upload["showlog"] = False
    ss.hosei_upload["display_errors"] = []
    ss.hosei_upload["show_error"] = False
    ss.hosei_upload["validation_errors"] = []
    ss.hosei_upload["valid_data"] = None
    ss.hosei_upload["success_msg_shown"] = False
    ss["correction_reason"] = ""
    st.rerun()


else:
    ss.hosei_upload["showlog"] = False
    ss.hosei_upload["success_msg_shown"] = True
    st.rerun()





        ss.hosei_upload["success_msg_shown"] = True