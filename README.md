if "hosei_upload" not in ss:
    ss.hosei_upload = {
        "show_error": False,
        "showlog": False,
        "validation_errors": [],
        "valid_data": None,
        "display_errors": [],
        "last_check_key": None,
        "json_data": None,
        "upload_file_name": None
    }


ss.hosei_upload["json_data"] = json_data
ss.hosei_upload["upload_file_name"] = upload.name
