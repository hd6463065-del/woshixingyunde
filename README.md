row_dict = {}
for col in db_row._fields:
    if col == "IS_EXIST":  # 跳过不需要的字段
        continue
    val = db_row[col]
    col_upper = str(col).upper()

    if val is None:
        row_dict[col_upper] = None
    elif isinstance(val, Decimal):
        # 关键：将 Decimal 转换为 float（如果确定是整数，也可以用 int(val)）
        row_dict[col_upper] = float(val)
    else:
        # 其他类型（str/bool/int/float 等）直接保留
        row_dict[col_upper] = val

# 一个主键对应一条数据
before_data_dict[json_key] = row_dict
