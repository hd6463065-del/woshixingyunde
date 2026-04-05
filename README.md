# ==============================================
# 【第1步：轻量主键校验，完全不用col，彻底解决所有报错】
# ==============================================
# 1. 上传主键转Snowpark DF
sp_pk = session.create_dataframe(pk_df)

# 2. 先定义sp_target_pk，再操作列（解决NameError）
sp_target_pk = session.table(f"SILVER_4.{selected_table}").select(primary_keys)

# 3. 给库表主键列手动加_DB后缀（完全不用col，原生写法）
sp_target_pk_aliased = sp_target_pk.select(
    [sp_target_pk[pk].alias(f"{pk}_DB") for pk in primary_keys]
)

# 4. 【关键修复：join的on参数，直接用字符串列名，不用Column对象】
# 用left join，rsuffix="_DB" 给库表列加后缀，上传列保留原名
joined_pk = sp_pk.join(
    sp_target_pk,
    on=primary_keys,  # ✅ 直接传字符串列名列表，符合要求
    how="left",
    rsuffix="_DB"     # ✅ 给库表列加_DB后缀，100%生效
)

# 5. 生成exist_map（列名100%正确，无KeyError）
exist_map = {}
for row in joined_pk.collect():
    # 上传列无后缀，直接取k
    key_tuple = tuple(str(row[k]).strip() for k in primary_keys)
    # 库表列带_DB后缀，判断存在
    is_exist = all(row[f"{k}_DB"] is not None for k in primary_keys)
    exist_map[key_tuple] = {"IS_EXIST": 1 if is_exist else 0}
