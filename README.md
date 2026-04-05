# ==============================================
# 【第1步：轻量主键校验，只查主键，判断存在不存在】
# ==============================================
# 1. 上传主键转Snowpark DF（只传主键，数据量极小）
sp_pk = session.create_dataframe(pk_df)
# 2. 库表只查主键列（极致轻量，不拿其他业务列）
sp_target_pk = session.table(f"SILVER_4.{selected_table}").select(primary_keys)
# 3. LEFT JOIN，只关联主键（和原SQL逻辑完全一致）
joined_pk = sp_pk.join(
    sp_target_pk,
    on=primary_keys,
    how="left",
    lsuffix="_UP",  # 上传数据的主键加后缀_UP
    rsuffix="_DB"    # 库表数据的主键加后缀_DB
)
