# ==============================================
# 【第1步：轻量主键校验，手动重命名，彻底解决_DB后缀问题】
# ==============================================
from snowflake.snowpark.functions import col  # 确保顶部导入了col

sp_pk = session.create_dataframe(pk_df)
# 1. 给库表主键列【手动加_DB后缀】，100%保证列名存在
sp_target_pk = session.table(f"SILVER_4.{selected_table}").select(
    [col(pk).alias(f"{pk}_DB") for pk in primary_keys]
)
# 2. 手动构建关联条件：上传列k = 库表列k_DB
join_conditions = [col(k) == col(f"{k}_DB") for k in primary_keys]
joined_pk = sp_pk.join(sp_target_pk, on=join_conditions, how="left")

exist_map = {}
for row in joined_pk.collect():
    # 上传列无后缀，直接取k
    key_tuple = tuple(str(row[k]).strip() for k in primary_keys)
    # 库表列带_DB后缀，判断存在
    is_exist = all(row[f"{k}_DB"] is not None for k in primary_keys)
    exist_map[key_tuple] = {"IS_EXIST": 1 if is_exist else 0}



