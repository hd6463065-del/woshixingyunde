import pandas as pd
import streamlit as st

# ======================
# 单个检查函数（分开写）
# ======================

def check4_add_duplicate(row, primary_key_cols, existing_keys):
    """
    Check4：処理区分=1（追加）→ 主键不能已存在
    失败返回 False，成功返回 True
    """
    if row["処理区分"] == "1":
        key = tuple(row[col] for col in primary_key_cols)
        if key in existing_keys:
            return False, "Check4 失败：追加的主键已存在"
    return True, ""

def check5_update_delete_exist(row, primary_key_cols, existing_keys):
    """
    Check5：処理区分=U/0（更新/删除）→ 主键必须存在
    失败返回 False，成功返回 True
    """
    if row["処理区分"] in ["U", "0"]:
        key = tuple(row[col] for col in primary_key_cols)
        if key not in existing_keys:
            return False, "Check5 失败：更新/删除的主键不存在"
    return True, ""

# ======================
# 核心：逐行校验，错一行就跳过这行
# ======================
def validate_and_filter_data(df, primary_key_cols, existing_keys):
    valid_rows = []  # 最终合格的数据
    error_msgs = []  # 错误信息

    for idx, row in df.iterrows():
        # --------------------
        # 第一步：Check4
        # --------------------
        pass4, msg4 = check4_add_duplicate(row, primary_key_cols, existing_keys)
        if not pass4:
            error_msgs.append(f"第{idx+2}行：{msg4}")
            continue  # ❌ 这行直接跳过！Check5 不会再执行

        # --------------------
        # 第二步：Check5（只有Check4过了才会走到这里）
        # --------------------
        pass5, msg5 = check5_update_delete_exist(row, primary_key_cols, existing_keys)
        if not pass5:
            error_msgs.append(f"第{idx+2}行：{msg5}")
            continue  # ❌ 跳过这行

        # ✅ 所有Check都过了，才保留
        valid_rows.append(row)

    # 转成DataFrame
    valid_df = pd.DataFrame(valid_rows)
    return valid_df, error_msgs



# 1. 你的主键列
primary_key_cols = ["你的主键列1", "你的主键列2"]

# 2. 从数据库查出来的已有主键（集合）
existing_keys = {("A001", "X100"), ("A002", "X200")}

# 3. 上传文件
uploaded_file = st.file_uploader("上传文件", type=["xlsx", "csv"])
if uploaded_file:
    df = pd.read_excel(uploaded_file)

    # 4. 执行校验（错一行就跳过这行）
    valid_df, errors = validate_and_filter_data(df, primary_key_cols, existing_keys)

    # 5. 输出结果
    if errors:
        for msg in errors:
            st.error(msg)
    else:
        st.success("全部校验通过")

    # 只有 valid_df 是合格数据，后续只处理这个
    st.dataframe(valid_df)
