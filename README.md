column_dict = service.get_column_dict(selected_table)
header_check_dict = {jp: en for en, jp in column_dict.items()}

header_ok = checks.validate_dfheader(df, header_check_dict)
