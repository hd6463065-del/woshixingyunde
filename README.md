import pandas as pd
import os

# -------------------------- 核心配置 --------------------------
EXCEL_FILE = "別紙_9_9_メッセージ一覧.xlsx"
# 请确保这里的Sheet名称和Excel里的完全一致，可先运行打印代码确认
TARGET_SHEETS = [
    "情報メッセージ一覧",
    "ワーニング・エラーメッセージ一覧"
]
HEADER_ROW = 7  # 列名在第8行（索引从0开始）
# -----------------------------------------------------------

MESSAGE_DATA = {}

# 遍历目标Sheet
for sheet_name in TARGET_SHEETS:
    print(f"正在处理Sheet: {sheet_name}")
    # 读取Excel，按列名读取，彻底避免索引错误
    df = pd.read_excel(
        EXCEL_FILE,
        sheet_name=sheet_name,
        header=HEADER_ROW,
        usecols=["メッセージID", "メッセージ区分", "表示メッセージ"]
    )

    # 过滤空行，只保留消息ID和内容都不为空的行
    df = df.dropna(subset=["メッセージID", "表示メッセージ"])

    # 逐行处理数据
    for _, row in df.iterrows():
        # 用列名取值，不再用 row[0]/row[2]
        msg_id = str(row["メッセージID"]).strip()
        msg_type = str(row["メッセージ区分"]).strip()
        # 先strip，再转义双引号，避免语法错误
        content = str(row["表示メッセージ"]).strip().replace('"', '\\"')

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
            "content": content,
            "level": level
        }

# 生成最终的messages.py文件
with open("messages.py", "w", encoding="utf-8") as f:
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

print(f"✅ 生成完成！共处理 {len(MESSAGE_DATA)} 条消息")
print(f"📂 文件已生成在当前目录: {os.path.abspath('messages.py')}")
