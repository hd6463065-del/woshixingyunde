import pandas as pd
import os
import webbrowser

# -------------------------- 核心配置（无需修改） --------------------------
EXCEL_FILE_NAME = "別紙_9_9_メッセージ一覧.xlsx"
TARGET_SHEETS = ["情報メッセージ一覧", "ワーニング・エラーメッセージ一覧"]
# 表头行（第8行是列名行，对应Excel行号），数据从第9行开始
HEADER_ROW = 7  # pandas索引从0开始，Excel第8行=索引7
# -------------------------------------------------------------------------

MESSAGE_DATA = {}

# 读取Excel
xlsx = pd.ExcelFile(EXCEL_FILE_NAME)
print(f"Excel中所有Sheet: {xlsx.sheet_names}")

for sheet_name in TARGET_SHEETS:
    if sheet_name not in xlsx.sheet_names:
        print(f"⚠️  跳过不存在的Sheet: {sheet_name}")
        continue

    print(f"🔄 正在处理Sheet: {sheet_name}")
    # 关键修复：读取表头，按列名获取数据，避免索引错误
    df = pd.read_excel(
        EXCEL_FILE_NAME,
        sheet_name=sheet_name,
        header=HEADER_ROW,  # 用第8行做列名
        usecols=["メッセージID", "メッセージ区分", "表示メッセージ"]  # 按列名读取，最可靠
    )

    # 过滤空行（只保留消息ID和内容都不为空的行）
    df = df.dropna(subset=["メッセージID", "表示メッセージ"])

    # 逐行处理（按列名取值，彻底解决KeyError）
    for _, row in df.iterrows():
        msg_id = str(row["メッセージID"]).strip()
        msg_type = str(row["メッセージ区分"]).strip()
        content = str(row["表示メッセージ"]).strip()

        # 过滤无效值
        if msg_id == "nan" or content == "nan":
            continue

        # 映射Level
        if "エラー" in msg_type:
            level = "ERR"
        elif "ワーニング" in msg_type:
            level = "WAR"
        elif "インフォメーション" in msg_type:
            level = "INF"
        else:
            level = "INF"

        MESSAGE_DATA[msg_id] = {
            "content": content.replace('"', '\\"'),  # 提前转义双引号
            "level": level
        }

# 生成最终文件
output_path = os.path.join(os.getcwd(), "messages.py")
with open(output_path, "w", encoding="utf-8") as f:
    f.write("# -*- coding: utf-8 -*-\n")
    f.write("# 自动生成：仅包含指定Sheet的消息字典\n")
    f.write("MESSAGE_DATA = {\n")
    for msg_id in sorted(MESSAGE_DATA.keys()):
        data = MESSAGE_DATA[msg_id]
        f.write(f'    "{msg_id}": {{\n')
        f.write(f'        "content": "{data["content"]}",\n')
        f.write(f'        "level": "{data["level"]}"\n')
        f.write('    },\n')
    f.write("}\n")

# 结果提示
print(f"✅ 生成成功！共处理 {len(MESSAGE_DATA)} 条消息")
print(f"📂 文件路径：{output_path}")
webbrowser.open(os.path.dirname(output_path))
