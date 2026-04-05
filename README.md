def check_db_primary_key(df, selected_table):
    primary_map = TABLE_PRIMARY_KEY_MAP[selected_table]
    primary_keys = primary_map["pk_en"]      # DB物理主键
    primary_keys_jp = primary_map["pk_jp"]   # Excel日文主键
    flag = True

    # JSON key 不包含 HOSEI_VER
    json_key_columns = [pk for pk in primary_keys if pk != "HOSEI_VER"]

    session = get_active_session()

    # 1. 从Excel取主键并去重
    pk_df = df[primary_keys_jp].drop_duplicates().copy()
    pk_df.columns = primary_keys

    # 全部转字符串并去空格，避免join不上
    for pk in primary_keys:
        pk_df[pk] = pk_df[pk].astype(str).str.strip()

    # 2. 转 Snowpark DataFrame
    filter_spdf = session.create_dataframe(pk_df)

    # 3. 目标表 Snowpark DataFrame
    target_spdf = session.table(f"SILVER_4.{selected_table}")

    # 4. 构造 join 条件
    join_condition = None
    for pk in primary_keys:
        cond = target_spdf[pk] == filter_spdf[pk]
        if join_condition is None:
            join_condition = cond
        else:
            join_condition = join_condition & cond

    # 5. LEFT JOIN
    joined_spdf = filter_spdf.join(
        target_spdf,
        join_condition,
        how="left"
    )

    # 6. 判断是否存在
    # 用第一个主键列是否为null判断
    first_pk = primary_keys[0]
    joined_spdf = joined_spdf.with_column(
        "IS_EXIST",
        (target_spdf[first_pk].is_not_null()).cast("int")
    )

    # 7. collect 结果
    rows = joined_spdf.collect()

    # 8. 构造 exist_map
    exist_map = {}

    for r in rows:
        row_dict = {}
        for col in r.as_dict().keys():
            row_dict[str(col).upper()] = r[col]

        if row_dict.get("IS_EXIST") == 1:
            key = tuple(str(row_dict.get(pk.upper(), "")).strip() for pk in primary_keys)
            exist_map[key] = row_dict

    # 9. 原Excel逐行检查
    valid_rows = []
    before_data_dict = {}

    for idx, row in df.iterrows():
        row_num = idx + 1
        shori_kbn = str(row.get("処理区分", "")).strip()

        full_key = tuple(str(row.get(jp_pk, "")).strip() for jp_pk in primary_keys_jp)
        is_exist = full_key in exist_map

        # I：不能存在
        if shori_kbn == SHORI_KBN_ADD and is_exist:
            mu.push_messages("456ERR3068", row_num)
            flag = False
            continue

        # U / D：必须存在
        if (shori_kbn == SHORI_KBN_UPDATE or shori_kbn == SHORI_KBN_DEL) and not is_exist:
            mu.push_messages("456ERR3069", row_num)
            flag = False
            continue

        valid_rows.append(row)

        # U / D：生成旧数据JSON
        if shori_kbn in [SHORI_KBN_UPDATE, SHORI_KBN_DEL] and is_exist:
            db_row_dict = exist_map[full_key]

            json_key = "@@@".join(
                str(db_row_dict.get(pk.upper(), "")).strip()
                for pk in json_key_columns
            )

            row_dict = {}
            for col, val in db_row_dict.items():
                if col == "IS_EXIST":
                    continue

                col_upper = str(col).upper()

                if val is None:
                    row_dict[col_upper] = None
                elif isinstance(val, Decimal):
                    row_dict[col_upper] = float(val)
                else:
                    row_dict[col_upper] = val

            before_data_dict[json_key] = row_dict

    before_data_json = json.dumps(before_data_dict, ensure_ascii=False, sort_keys=False)

    return pd.DataFrame(valid_rows), before_data_json, exist_map, flag
