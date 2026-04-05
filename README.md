# 4. 【关键修复：直接用字符串列名JOIN，不手动拼条件，避免列名污染】
joined_spdf = filter_spdf.join(
    target_spdf,
    on=primary_keys,  # ✅ 直接传字符串列名列表
    how="left"
)

# 5. 加IS_EXIST列（用原始列名，无前缀）
joined_spdf = joined_spdf.with_column(
    "IS_EXIST",
    joined_spdf[primary_keys[0]].is_not_null().cast("int")
)




# 用JOIN后的列名，无前缀，判断存在
joined_spdf = joined_spdf.with_column(
    "IS_EXIST",
    joined_spdf[first_pk].is_not_null().cast("int")
)
