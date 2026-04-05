@st.cache_data
def get_colomn_dict(table)-> dict:
    table_info = TblInfos.TABLE_MAP[table].cols
    all_cols = [col for col in table_info.__dict__.values() if isinstance(col, TblCol)]

    #物理名リスト
    phsical_list = []
    #論理名リスト
    logical_list = []

    for col in all_cols:
        phsical_list.append(col.col_en)
        logical_list.append(col.col_jp)

    colomndict = dict(zip(phsical_list,logical_list))

    # ✅ 【直接覆盖，所有表生效，不用判断表名！】
    # 把正确的英文列名写进去，覆盖原有错误映射
    colomndict["HOSEIBI"] = "登録日"
    colomndict["HOSEISHA"] = "登録者"
    colomndict["HOSEI_KAKUNINSHA"] = "登録確認者"

    return colomndict
