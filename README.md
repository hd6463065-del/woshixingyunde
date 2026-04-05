# ==============================================
# 【第1步：轻量主键校验，只查主键，判断存在不存在】
# ==============================================
sp_pk = session.create_dataframe(pk_df)
# 库表只查主键列，给库表列加 _DB 后缀，上传列保留原名
sp_target_pk = session.table(f"SILVER_4.{selected_table}").select(primary_keys)
joined_pk = sp_pk.join(
    sp_target_pk,
    on=primary_keys,
    how="left",
    rsuffix="_DB"  # 仅给库表列加 _DB 后缀，上传列保持原名
)

exist_map = {}
for row in joined_pk.collect():
    # 上传列无后缀，直接用k取值
    key_tuple = tuple(str(row[k]).strip() for k in primary_keys)
    # 库表列加 _DB 后缀，判断是否存在
    is_exist = all(row[f"{k}_DB"] is not None for k in primary_keys)
    exist_map[key_tuple] = {"IS_EXIST": 1 if is_exist else 0}
