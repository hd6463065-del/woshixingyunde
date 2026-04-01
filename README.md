# I时：这些字段禁止填写（必须为空）——字段名写Excel里的日文列名
PROHIBITED_FIELDS_I = [
    # 例子：把下面换成你真实字段
    # "契約番号",
    # "顧客番号",
    # "商品コード",
]

# U时：每个表各自的不可更改字段（字段名写Excel里的日文列名）
IMMUTABLE_FIELDS_U_MAP = {
    "T456SMMA010": [
        # "字段1",
        # "字段2",
    ],
    "T456SMMA020": [],
    "T456SMMA030": [],
    "T456SMMA040": [],
    "T456SMMA050": [],
    "T456SMMA060": [],
    "T456SMMA070": [],
    "T456SMMB010": [],
    "T456SMMA080": [],
    "T456SMMC020": [],
    "T456SMMC040": [],
}

def check_db_primary_key(df, selected_table):
    primary_map = TABLE_PRIMARY_KEY_MAP[selected_table]
    primary_keys = primary_map["pk_en"]
    primary_keys_jp = primary_map["pk_jp"]

    # JSON key 不包含 HOSEI_VER
    json_key_columns = [pk for pk in primary_keys if pk != "HOSEI_VER"]

    session = get_active_session()

    pk_df = df[primary_keys_jp].drop_duplicates().copy()
    pk_df.columns = primary_keys

    for pk in primary_keys:
        pk_df[pk] = pk_df[pk].astype(str).str.strip()

    session.create_dataframe(pk_df).write.save_as_table(
        "tmp_filter",
        table_type="transient",
        mode="overwrite"
    )

    rows = runner.query(
        "get_hosei_check",
        {
            "table_name": selected_table,
            "primary_keys": tuple(primary_keys),
        }
    ).collect()

    exist_map = {}

    for r in rows:
        row_dict = {}
        for col in r._fields:
            row_dict[str(col).upper()] = r[col]

        if row_dict.get("IS_EXIST") == 1:
            key = tuple(str(row_dict.get(pk.upper(), "")).strip() for pk in primary_keys)
            exist_map[key] = row_dict

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
            continue

        # U / D：必须存在
        if (shori_kbn == SHORI_KBN_UPDATE or shori_kbn == SHORI_KBN_DEL) and not is_exist:
            mu.push_messages("456ERR3069", row_num)
            continue

        valid_rows.append(row)

        # U：只要DB存在，就生成 before_data_json
        if shori_kbn == SHORI_KBN_UPDATE and is_exist:
            db_row_dict = exist_map[full_key]

            json_key = "@@@".join(
                str(db_row_dict.get(pk.upper(), "")).strip()
                for pk in json_key_columns
            )

            row_dict = {}
            for col, val in db_row_dict.items():
                if col == "IS_EXIST":
                    continue

                if val is None:
                    row_dict[col] = None
                elif isinstance(val, Decimal):
                    row_dict[col] = float(val)
                else:
                    row_dict[col] = val

            before_data_dict[json_key] = row_dict

    before_data_json = json.dumps(before_data_dict, ensure_ascii=False, sort_keys=False)

    return pd.DataFrame(valid_rows), before_data_json, exist_map



    def check_idu_business_rule(df, selected_table, exist_map):
    valid_rows = []
    business_errors = []

    for idx, row in df.iterrows():
        row_num = idx + 1
        shori_kbn = str(row.get("処理区分", "")).strip()

        # I：指定字段必须为空（456ERR998）
        if shori_kbn == SHORI_KBN_ADD:
            error_cols = []

            for col in PROHIBITED_FIELDS_I:
                val = row.get(col)

                if val is not None and str(val).strip() != "":
                    error_cols.append(col)

            if error_cols:
                business_errors.append(
                    f"レコード[{row_num}]：456ERR998 入力不可項目に値が設定されています（項目：{', '.join(error_cols)}）"
                )
                continue

        valid_rows.append(row)

    return pd.DataFrame(valid_rows), business_errors




    # 第一步：主键检查（错误显示在上面消息区）
df_key_ok, jsonData, exist_map = service.check_db_primary_key(df, selected_table)

# 第二步：I/U/D业务检查（错误显示在下面红框）
df_business_ok, business_errors = service.check_idu_business_rule(
    df_key_ok,
    selected_table,
    exist_map
)

# 第三步：字段规则检查
current_table_rules = validation_rules.get(selected_table, {})
rule_errors = checks.validate_dataframe(df_business_ok, list(current_table_rules.items()))

# 下方错误区
ss.hosei_upload["validation_errors"] = business_errors + rule_errors

# 最终可登录数据
df = df_business_ok
