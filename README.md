import json
import pandas as pd

SHORI_KBN_ADD = "I"
SHORI_KBN_UPDATE = "U"
SHORI_KBN_DEL = "D"

def check_db_primary_key(df, selected_table):

    primary_keys = TABLE_PRIMARY_KEY_MAP[selected_table]

    # JSON key 不包含 HOSEI_VER
    json_key_columns = [pk for pk in primary_keys if pk != "HOSEI_VER"]

    session = get_active_session()

    # =========================
    # ① 主键数据写入临时表
    # =========================
    pk_df = df[primary_keys].drop_duplicates().copy()

    for pk in primary_keys:
        pk_df[pk] = pk_df[pk].astype(str).str.strip()

    session.create_dataframe(pk_df).write.save_as_table(
        "tmp_filter",
        table_type="temporary",
        mode="overwrite"
    )

    # =========================
    # ② 执行 JOIN SQL
    # =========================
    sql = load_sql("get_hosei_check")  # 你原来的读取方式

    rows = session.sql(
        sql,
        params={"table_name": selected_table}
    ).collect()

    # =========================
    # ③ 构造存在Map（用于快速判断）
    # =========================
    exist_map = {}

    for r in rows:
        key = tuple(str(r[pk]).strip() for pk in primary_keys)
        exist_map[key] = r

    # =========================
    # ④ 校验 + JSON构建
    # =========================
    valid_rows = []
    before_data_dict = {}

    for idx, row in df.iterrows():

        row_num = row["行番号"]
        shori_kbn = str(row.get("SHORI_KBN", "")).strip()

        full_key = tuple(str(row.get(pk, "")).strip() for pk in primary_keys)

        is_exist = full_key in exist_map

        # ===== I：不能存在 =====
        if shori_kbn == SHORI_KBN_ADD and is_exist:
            mu.push_messages("459EBR9959", row_num)
            continue

        # ===== U / D：必须存在 =====
        if (shori_kbn == SHORI_KBN_UPDATE or shori_kbn == SHORI_KBN_DEL) and not is_exist:
            mu.push_messages("459EBR9956", row_num)
            continue

        valid_rows.append(row)

        # ===== U：生成JSON =====
        if shori_kbn == SHORI_KBN_UPDATE and is_exist:

            db_row = exist_map[full_key]

            json_key = "@@@".join(
                str(db_row[pk]).strip() for pk in json_key_columns
            )

            row_dict = {
                str(col).upper(): str(db_row[col]).strip() if db_row[col] is not None else ""
                for col in db_row.as_dict().keys()
                if col not in json_key_columns  # 这里保留所有字段（含HOSEI_VER）
            }

            # 支持一key多条
            if json_key not in before_data_dict:
                before_data_dict[json_key] = []

            before_data_dict[json_key].append(row_dict)

    # =========================
    # ⑤ 转JSON字符串
    # =========================
    before_data_json = json.dumps(before_data_dict, ensure_ascii=False)

    return pd.DataFrame(valid_rows), before_data_json





    -- name: get_hosei_check
SELECT
    f.*,
    t.*,
    CASE 
        WHEN t.{{primary_keys[0]}} IS NOT NULL THEN 1
        ELSE 0
    END AS IS_EXIST
FROM
    tmp_filter f
LEFT JOIN
    MAIN_H_SILVER_4.{{table_name}} t
ON
    {% for pk in primary_keys %}
        t.{{pk}} = f.{{pk}}
        {% if not loop.last %} AND {% endif %}
    {% endfor %}





    TABLE_CONFIG = {
    "T456SMOF040": {
        "pk_en": ["SHOP_ID", "ITEM_ID", "HOSEI_VER"],
        "pk_jp": ["店舗コード", "商品コード", "補正バージョン"]
    }
}



pk_en = TABLE_CONFIG[selected_table]["pk_en"]
pk_jp = TABLE_CONFIG[selected_table]["pk_jp"]

