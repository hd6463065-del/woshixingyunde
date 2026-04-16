if "hosei_upload" not in ss:
    ss.hosei_upload = {
        "show_error": False,
        "showlog": False,
        "validation_errors": [],
        "valid_data": None,
        "display_errors": [],
        "success_msg_shown": False,
        "last_upload_obj_id": None,
    }



if upload:
    current_upload_obj_id = id(upload)

    if ss.hosei_upload["last_upload_obj_id"] != current_upload_obj_id:
        ss.hosei_upload["success_msg_shown"] = False
        ss.hosei_upload["last_upload_obj_id"] = current_upload_obj_id

    upload_file_name = upload.name
    ...



if success:
    ss.hosei_upload["showlog"] = False
    ss.hosei_upload["display_errors"] = []
    ss.hosei_upload["show_error"] = False
    ss.hosei_upload["validation_errors"] = []
    ss.hosei_upload["valid_data"] = None
    ss.hosei_upload["success_msg_shown"] = False
    ss.hosei_upload["last_upload_obj_id"] = None
    ss["correction_reason"] = ""
    st.rerun()