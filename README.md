# エラーメッセージをまとめて出力
if err3068_rows:
    mu.push_messages("456ERR3068", "、".join(err3068_rows))

if err3069_rows:
    mu.push_messages("456ERR3069", "、".join(err3069_rows))

return pd.DataFrame(valid_rows), before_data_json, exist_map_full, flag