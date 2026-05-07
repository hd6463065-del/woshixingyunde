if not e.startswith("11件以上")  # 不是“11件以上”的，正常加序号
    else f"<div class='error_item'>{e}</div>"  # 是“11件以上”的，不加序号