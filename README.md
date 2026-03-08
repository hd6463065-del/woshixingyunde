
import pandas as pd
import os
import webbrowser

# -------------------------- 仅需修改这里 --------------------------
EXCEL_FILE_NAME = "消息一覧.xlsx"  # 改成你的Excel文件名
# -----------------------------------------------------------------

# 初始化消息字典
MESSAGE_DATA = {}

# 读取Excel，获取所有工作表
xlsx = pd.ExcelFile(EXCEL_FILE_NAME)
print(f"检测到 {len(xlsx.sheet_names)} 个工作表，开始处理...")

# 遍历每个工作表
for sheet_name in xlsx.sheet_names:
    # 读取数据：从第9行开始（skiprows=8，因为索引从0开始），只取A、B、F列
    df = pd.read_excel(
        EXCEL_FILE_NAME,
        sheet_name=sheet_name,
        usecols="A,B,F",  # 对应：消息ID、消息区分、表示メッセージ
        header=None,      # 不把第一行当表头
        skiprows=8        # 跳过前8行，从第9行开始读数据
    )

    # 逐行处理
    for _, row in df.iterrows():
        # 提取数据，过滤空值
        msg_id = str(row[0]).strip() if pd.notna(row[0]) else None
        msg_type = str(row[1]).strip() if pd.notna(row[1]) else None
        content = str(row[2]).strip() if pd.notna(row[2]) else None

        # 过滤无效数据（空ID、空内容、纯nan）
        if not msg_id or msg_id == "nan" or not content or content == "nan":
            continue

        # 映射level
        if "エラー" in msg_type:
            level = "ERR"
        elif "ワーニング" in msg_type:
            level = "WAR"
        elif "情報" in msg_type:
            level = "INF"
        else:
            level = "INF"

        # 加入字典（重复ID会覆盖，保留最后一个）
        MESSAGE_DATA[msg_id] = {
            "content": content,
            "level": level
        }

# 生成Python文件
output_file = "messages.py"
with open(output_file, "w", encoding="utf-8") as f:
    f.write("# -*- coding: utf-8 -*-\n")
    f.write("# 自动生成：包含所有工作表的消息字典\n")
    f.write("MESSAGE_DATA = {\n")
    # 按ID排序，格式更整洁
    for msg_id in sorted(MESSAGE_DATA.keys()):
        data = MESSAGE_DATA[msg_id]
        # 处理双引号转义，避免语法错误
        safe_content = data["content"].replace('"', '\\"')
        f.write(f'    "{msg_id}": {{\n')
        f.write(f'        "content": "{safe_content}",\n')
        f.write(f'        "level": "{data["level"]}"\n')
        f.write('    },\n')
    f.write("}\n")

# 打开生成文件所在的文件夹（不用手动找了！）
output_path = os.path.abspath(output_file)
output_dir = os.path.dirname(output_path)
webbrowser.open(output_dir)

print(f"✅ 生成成功！共处理 {len(MESSAGE_DATA)} 条消息")
print(f"📂 文件位置：{output_path}")
