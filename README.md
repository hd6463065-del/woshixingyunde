# ==============================================
# 【第2步：只对有效的U/D，去拿DB旧整条数据】
# ==============================================
# 无U/D行时自动跳过，不执行
if valid_rows:
    # 1. 把需要的主键转成Snowpark DF
    sp_need = session.create_dataframe(valid_rows, schema=primary_keys_jp)
    # 2. 关联库表，只拉取这些主键对应的全量数据
    sp_target_full = session.table(f"SILVER_4.{selected_table}")
    joined_full = sp_need.join(
        sp_target_full,
        on=primary_keys_jp,  # 用jp列名关联，和full_key完全对应
        how="inner"  # 只拿存在的，U/D已校验过存在，更高效
    )

    # 3. 收集全量数据，填充到before_data_dict
    for row in joined_full.collect():
        # 生成和原代码完全一致的full_key
        key_tuple = tuple(str(row[k]).strip() for k in primary_keys_jp)
        # 存储整条DB数据，用法和原before_data_dict完全一致
        before_data_dict[key_tuple] = row.as_dict()
