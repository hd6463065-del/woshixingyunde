if "hosei_upload" not in ss:
    ss.hosei_upload = {
        "show": False,                # 对应原来的 ss.show
        "upload": False,              # 对应原来的 ss.upload
        "show_error": False,          # 对应原来的 ss.show_error
        "showlog": False,             # 对应原来的 ss.showlog
        "validation_errors": [],      # 对应原来的 ss.validation_errors
        "valid_data": None,           # 对应原来的 ss.valid_data
        "kijundate": datetime.now().strftime("%Y%m"),  # 对应原来的 ss.kijundate
        # 👇 额外加这2个，解决你当前的报错+后续优化用
        "display_errors": [],         # 解决「display_errors未初始化」的报错
        "btn_submit": False,          # 控制提交按钮状态用
        "messages": {"error": "", "success": ""}  # 统一管理提示消息
    }


ss.show	ss.hosei_upload["show"]
ss.upload	ss.hosei_upload["upload"]
ss.show_error	ss.hosei_upload["show_error"]
ss.showlog	ss.hosei_upload["showlog"]
ss.validation_errors	ss.hosei_upload["validation_errors"]
ss.valid_data	ss.hosei_upload["valid_data"]
ss.kijundate	ss.hosei_upload["kijundate"]
ss.display_errors	ss.hosei_upload["display_errors"]
