# 4. 收集主键结果，生成exist_map（只拉主键，数据量极小，内存无压力）
exist_map = {}
for row in joined_pk.collect():
    # 生成和原来完全一样的key_tuple，后续校验逻辑完全兼容
    key_tuple = tuple(str(row[f"{k}_UP"]).strip() for k in primary_keys)
    # 判断是否存在：所有库表主键列都不为None，和原IS_EXIST=1逻辑完全一致
    is_exist = all(row[f"{k}_DB"] is not None for k in primary_keys)
    # 兼容原代码结构，封装成rowdict格式
    exist_map[key_tuple] = {"IS_EXIST": 1 if is_exist else 0}
