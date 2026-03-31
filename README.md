# ... 前面的逻辑：获取db_row、生成json_key ...

json_key = "@@@".join(
    str(db_row[pk]).strip() for pk in json_key_columns
)

# 生成 row_dict（已修复为遍历_fields的版本，避免as_dict报错）
row_dict = {}
for col in db_row._fields:
    if col not in json_key_columns:
        val = db_row[col]
        row_dict[str(col).upper()] = str(val).strip() if val is not None else ""

# 👇 关键修改：一个key只对应一条数据，直接赋值
before_data_dict[json_key] = row_dict

# ... 最后生成JSON ...
before_data_json = json.dumps(before_data_dict, ensure_ascii=False)
