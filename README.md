# 匹配 整数 或 正小数（0、123、0.025、99.99 全部通过）
if re.fullmatch(r"[0-9]+(\.[0-9]+)?", val_str):
    return True
else:
    return False
