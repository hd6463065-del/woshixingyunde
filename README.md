if upload:
    ss.hosei_upload["show_error"] = False
    ss.hosei_upload["validation_errors"] = []

    if not selected_table:
        mu.push_messages("456ERR3053", "対象テーブル")
    else:
        try:
            df = excel_file_util.read_wb_as_df(upload)
            if "処理区分" not in df.columns:
                mu.push_messages("456ERR3056", "処理区分")
                ss.hosei_upload["validation_errors"] = ["error_msg"]
            else:
                # ✅ 核心：获取文件二进制并转十六进制
                upload.seek(0)  # 重置文件指针
                file_binary = upload.read()
                hex_str = binascii.hexlify(file_binary).decode('utf-8')
                
                # ✅ 构造 SQL 时传入 hex_str
                sql = f"""
                INSERT INTO ...
                UPLOAD_FILE_JOHO = HEX_DECODE_BINARY('{hex_str}')
                ...
                """
                # 执行 SQL...
        except Exception as e:
            mu.push_messages("456ERR9999", str(e))
