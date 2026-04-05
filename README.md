# ==============================================
# 【第1步：轻量主键校验，变量顺序正确，彻底解决NameError】
# ==============================================
sp_pk = session.create_dataframe(pk_df)

# 1. 【先定义sp_target_pk，再操作列！】
sp_target_pk = session.table(f"SILVER_4.{selected_table}").select(primary_keys)

# 2. 给库表主键列手动加_DB后缀（现在sp_target_pk已定义，可正常访问）
sp_target_pk_aliased = sp_target_pk.select(
    [col(pk).alias(f"{pk}_DB") for pk in primary_keys]
)

# 3. 手动构建关联条件
join_conditions = [col(k) == col(f"{k}_DB") for k in primary_keys]
joined_pk = sp_pk.join(sp_target_pk_aliased, on=join_conditions, how="left")

# 4. 生成exist_map（列名100%正确，无KeyError）
exist_map = {}
for row in joined_pk.collect():
    key_tuple = tuple(str(row[k]).strip() for k in primary_keys)
    is_exist = all(row[f"{k}_DB"] is not None for k in primary_keys)
    exist_map[key_tuple] = {"IS_EXIST": 1 if is_exist else 0}
