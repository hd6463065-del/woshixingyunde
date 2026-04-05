is_exist = full_key in exist_map and exist_map[full_key]["IS_EXIST"] == 1


# 【改2：新增：收集需要拉取DB数据的U/D行】
     if shori_kbn in [SHORI_KBN_UPDATE, SHORI_KBN_DEL] and is_exist:
         valid_rows.append(full_key)  # 记录需要拉取的主键
         before_data_dict[full_key] = None  # 占位，后续填充DB数据

