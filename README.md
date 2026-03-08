import pandas as pd
import os

# -------------------------- 核心配置 --------------------------
EXCEL_FILE = "別紙_9_9_メッセージ一覧.xlsx"
TARGET_SHEETS = [
    "情報メッセージ一覧",
    "ワーニング・エラーメッセージ一覧"
]
HEADER_ROW = 7  # 列名在第8行（索引从0开始）
# -----------------------------------------------------------

MESSAGE_DATA = {}

for sheet_name in TARGET_SHEETS:
    print(f"正在处理Sheet: {sheet_name}")
    df = pd.read_excel(
        EXCEL_FILE,
        sheet_name=sheet_name,
        header=HEADER_ROW,
        usecols=["メッセージID", "メッセージ区分", "表示メッセージ"]
    )
    df = df.dropna(subset=["メッセージID", "表示メッセージ"])

    for _, row in df.iterrows():
        msg_id = str(row["メッセージID"]).strip()
        msg_type = str(row["メッセージ区分"]).strip()
        content = str(row["表示メッセージ"]).strip()

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
            "content": content,
            "level": level
        }

# 生成文件时，统一处理所有双引号转义
with open("messages.py", "w", encoding="utf-8") as f:
    f.write("# -*- coding: utf-8 -*-\n")
    f.write("# 自动生成：仅包含指定Sheet的消息字典\n")
    f.write("MESSAGE_DATA = {\n")
    for msg_id in sorted(MESSAGE_DATA.keys()):
        data = MESSAGE_DATA[msg_id]
        # 关键：在写入文件时，统一转义双引号
        safe_content = data["content"].replace('"', '\\"')
        f.write(f'    "{msg_id}": {{\n')
        f.write(f'        "content": "{safe_content}",\n')
        f.write(f'        "level": "{data["level"]}"\n')
        f.write('    },\n')
    f.write("}\n")

print(f"✅ 生成完成！共处理 {len(MESSAGE_DATA)} 条消息")
print(f"📂 文件已生成在当前目录: {os.path.abspath('messages.py')}")
