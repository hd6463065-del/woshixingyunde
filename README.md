import pandas as pd
import os
import webbrowser

# -------------------------- 核心配置 --------------------------
EXCEL_FILE_NAME = "別紙_9_9_メッセージ一覧.xlsx"
TARGET_SHEETS = [
    "情報メッセージ一覧",
    "ワーニング・エラーメッセージ一覧"
]
# 表头行：Excel 第8行（pandas 索引从0开始，所以是 7）
HEADER_ROW = 7
# -----------------------------------------------------------

MESSAGE_DATA = {}

# 读取 Excel
xlsx = pd.ExcelFile(EXCEL_FILE_NAME)
print(f"检测到所有Sheet: {xlsx.sheet_names}")
print(f"只处理指定Sheet: {TARGET_SHEETS}")

for sheet_name in TARGET_SHEETS:
    if sheet_name not in xlsx.sheet_names:
        print(f"⚠️  警告：未找到Sheet '{sheet_name}'，跳过。")
        continue

    print(f"正在处理: {sheet_name}")
    # 关键：按列名读取，彻底避免索引错误
    df = pd.read_excel(
        EXCEL_FILE_NAME,
        sheet_name=sheet_name,
        header=HEADER_ROW,  # 用第8行作为列名
        usecols=[
            "メッセージID",
            "メッセージ区分",
            "表示メッセージ"
        ]  # 直接用你Excel里的列名
    )

    # 过滤掉空行
    df = df.dropna(subset=["メッセージID", "表示メッセージ"])

    # 逐行处理（用列名取值，不会再报 KeyError）
    for _, row in df.iterrows():
        msg_id = str(row["メッセージID"]).strip()
        msg_type = str(row["メッセージ区分"]).strip()
        content = str(row["表示メッセージ"]).strip()

        if msg_id == "nan" or content == "nan":
            continue

        # 映射 level
        if "エラー" in msg_type:
            level = "ERR"
        elif "ワーニング" in msg_type:
            level = "WAR"
        elif "インフォメーション" in msg_type:
            level = "INF"
        else:
            level = "INF"

        MESSAGE_DATA[msg_id] = {
            "content": content.replace('"', '\\"'),
            "level": level
        }

# 生成 messages.py
output_file = "messages.py"
with open(output_file, "w", encoding="utf-8") as f:
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

# 自动打开目录
output_path = os.path.abspath(output_file)
output_dir = os.path.dirname(output_path)
webbrowser.open(output_dir)

print(f"✅ 生成完成！共处理 {len(MESSAGE_DATA)} 条消息")
print(f"📂 文件位置：{output_path}")
