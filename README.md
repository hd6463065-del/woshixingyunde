# ==============================================
# 【第1步：轻量主键校验，100%解决_DB后缀问题，col导入不影响】
# ==============================================
sp_pk = session.create_dataframe(pk_df)

# 1. 给库表主键列【手动加_DB后缀】，保证列名100%存在
sp_target_pk = session.table(f"SILVER_4.{selected_table}").select(
    [sp_target_pk[pk].alias(f"{pk}_DB") for pk in primary_keys]
)

# 2. 手动关联：上传列k = 库表列k_DB
join_conditions = [sp_pk[pk] == sp_target_pk[f"{pk}_DB"] for pk in primary_keys]
joined_pk = sp_pk.join(sp_target_pk, on=join_conditions, how="left")

exist_map = {}
for row in joined_pk.collect():
    # 上传列无后缀，直接取k
    key_tuple = tuple(str(row[k]).strip() for k in primary_keys)
    # 库表列带_DB后缀，判断存在
    is_exist = all(row[f"{k}_DB"] is not None for k in primary_keys)
    exist_map[key_tuple] = {"IS_EXIST": 1 if is_exist else 0}
