row_dict = {}
# 严格按数据库字段顺序遍历，跳过 IS_EXIST
for col in db_row._fields:
    if col == "IS_EXIST":
        continue
    val = db_row[col]
    col_upper = str(col).upper()
    # 直接保留原类型：None → JSON null，其他类型原样保留
    row_dict[col_upper] = val if val is not None else None

# 一个主键对应一条数据
before_data_dict[json_key] = row_dict

# 生成 JSON（锁死顺序、不转码日文）
before_data_json = json.dumps(
    before_data_dict,
    ensure_ascii=False,
    sort_keys=False  # 关键：保证字段顺序和数据库完全一致
)
