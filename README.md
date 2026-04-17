import hashlib


"last_check_key": None



def make_check_key(upload, selected_table):
    if upload is None:
        return None

    file_hash = hashlib.md5(upload.getvalue()).hexdigest()
    return f"{selected_table}:{upload.name}:{file_hash}"




current_check_key = make_check_key(upload, selected_table)



if upload and current_check_key != ss.hosei_upload["last_check_key"]:
    ss.hosei_upload["last_check_key"] = current_check_key

    # 先清空旧结果
    ss.hosei_upload["valid_data"] = None
    ss.hosei_upload["validation_errors"] = []
    ss.hosei_upload["display_errors"] = []
    ss.hosei_upload["show_error"] = False

    upload_file_name = upload.name

    # 対象テーブル必須チェック
    if not selected_table:
        mu.push_messages("456ERR3053", "対象テーブル")
    else:
        df = excel_file_util.read_wb_as_df(upload)

        # 処理区分列存在チェック
        if "処理区分" not in df.columns:
            mu.push_messages("456ERR3056", "処理区分")
        else:
            # 列数チェック
            inssus = service.check_file_column_count(df, selected_table)
            if inssus is False:
                mu.push_messages("456ERR3057")
            else:
                column_dict = service.get_column_dict(selected_table)
                header_check_dict = {jp: en for en, jp in column_dict.items()}
                df_check = df.drop(columns=["処理区分"])

                # フィールド名チェック
                header_ok = checks.validate_dfheader(df_check, header_check_dict)
                if header_ok is False:
                    mu.push_messages("456ERR4093")
                else:
                    # DB主キー存在チェック
                    df_pk, json_data, exist_map, flag = service.check_db_primary_key(df, selected_table)

                    # 業務ルールチェック
                    business_errors = service.check_idu_business_rule(df_pk, selected_table, exist_map)
                    current_table_rules = validation_rules_map.get(selected_table, {})

                    # 入力チェック実施
                    ss.hosei_upload["validation_errors"] = checks.validate_uplode(
                        df_pk, list(current_table_rules.items())
                    ) + business_errors

                    if flag and len(ss.hosei_upload["validation_errors"]) == 0:
                        ss.hosei_upload["valid_data"] = df

                    if len(ss.hosei_upload["validation_errors"]) > 0:
                        ss.hosei_upload["show_error"] = True
                        display_errors = ss.hosei_upload["validation_errors"][:10]
                        if len(ss.hosei_upload["validation_errors"]) > 10:
                            display_errors.append("11件以上あり")
                        ss.hosei_upload["display_errors"] = display_errors
                        mu.push_messages("456ERR4058")



