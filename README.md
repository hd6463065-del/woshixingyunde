if "hosei_upload" not in ss:
    ss.hosei_upload = {
        "show_error": False,
        "showlog": False,
        "validation_errors": [],
        "valid_data": None,
        "display_errors": [],
        "last_check_key": None,
        "json_data": None,
        "upload_file_name": None,
        "upload_seq": 0
    }

def on_upload_change():
    st.session_state.hosei_upload["upload_seq"] += 1




def make_check_key(upload, selected_table, upload_seq):
    if upload is None:
        return None

    file_hash = hashlib.md5(upload.getvalue()).hexdigest()
    return f"{selected_table}:{upload_seq}:{upload.name}:{file_hash}"
upload = st.file_uploader(
    "アップロード",
    type=["xlsx", "xls"],
    label_visibility="collapsed",
    key="hosei_upload_file",
    on_change=on_upload_change
)



