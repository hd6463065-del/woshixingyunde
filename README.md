import pandas as pd
import json
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
# 用列表来保持顺序
ordered_msg_ids = []

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
        # 按 Excel 里的顺序记录 ID
        ordered_msg_ids.append(msg_id)

# 先生成 JSON，再转成 Python 代码，彻底避免语法错误
json_str = json.dumps(MESSAGE_DATA, ensure_ascii=False, indent=4)

with open("messages.py", "w", encoding="utf-8") as f:
    f.write("# -*- coding: utf-8 -*-\n")
    f.write("# 自动生成：按 Excel 一览顺序的消息字典\n")
    f.write("import json\n")
    f.write("MESSAGE_DATA = json.loads('''\n")
    f.write(json_str)
    f.write("\n''')\n")
    # 如果需要严格按顺序遍历，可以用这个列表
    f.write("\n# 按一览顺序的消息ID列表\n")
    f.write(f"ORDERED_MSG_IDS = {ordered_msg_ids}\n")

print(f"✅ 生成完成！共处理 {len(MESSAGE_DATA)} 条消息")
print(f"📂 文件已生成在当前目录: {os.path.abspath('messages.py')}")
