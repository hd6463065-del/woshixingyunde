import pandas as pd
import os
import webbrowser

# -------------------------- 配置区 --------------------------
EXCEL_FILE_NAME = "別紙_9_9_メッセージ一覧.xlsx"
TARGET_SHEETS = [
    "情報メッセージ一覧",
    "ワーニング・エラーメッセージ一覧"
]
# -----------------------------------------------------------

# 初始化消息字典
MESSAGE_DATA = {}

# 读取Excel
xlsx = pd.ExcelFile(EXCEL_FILE_NAME)
print(f"检测到所有Sheet: {xlsx.sheet_names}")
print(f"只处理指定Sheet: {TARGET_SHEETS}")

# 遍历目标Sheet
for sheet_name in TARGET_SHEETS:
    if sheet_name not in xlsx.sheet_names:
        print(f"⚠️  警告：未找到Sheet '{sheet_name}'，跳过。")
        continue

    print(f"正在处理: {sheet_name}")
    # 读取数据：从第9行开始，只取A、B、F列
    df = pd.read_excel(
        EXCEL_FILE_NAME,
        sheet_name=sheet_name,
        usecols="A,B,F",
        header=None,
        skiprows=8  # 跳过前8行，从第9行开始读数据
    )

    # 逐行处理
    for _, row in df.iterrows():
        msg_id = str(row[0]).strip() if pd.notna(row[0]) else None
        msg_type = str(row[1]).strip() if pd.notna(row[1]) else None
        content = str(row[2]).strip() if pd.notna(row[2]) else None

        # 过滤无效数据
        if not msg_id or msg_id == "nan" or not content or content == "nan":
            continue

        # 映射level
        if "エラー" in msg_type:
            level = "ERR"
        elif "ワーニング" in msg_type:
            level = "WAR"
        elif "インフォメーション" in msg_type:
            level = "INF"
        else:
            level = "INF"

        # 加入字典
        MESSAGE_DATA[msg_id] = {
            "content": content,
            "level": level
        }

# 生成Python文件
output_file = "messages.py"
with open(output_file, "w", encoding="utf-8") as f:
    f.write("# -*- coding: utf-8 -*-\n")
    f.write("# 自动生成：仅包含指定Sheet的消息字典\n")
    f.write("MESSAGE_DATA = {\n")
    for msg_id in sorted(MESSAGE_DATA.keys()):
        data = MESSAGE_DATA[msg_id]
        safe_content = data["content"].replace('"', '\\"')
        f.write(f'    "{msg_id}": {{\n')
        f.write(f'        "content": "{safe_content}",\n')
        f.write(f'        "level": "{data["level"]}"\n')
        f.write('    },\n')
    f.write("}\n")

# 自动打开文件所在目录
output_path = os.path.abspath(output_file)
output_dir = os.path.dirname(output_path)
webbrowser.open(output_dir)

print(f"✅ 生成完成！共处理 {len(MESSAGE_DATA)} 条消息")
print(f"📂 文件位置：{output_path}")
