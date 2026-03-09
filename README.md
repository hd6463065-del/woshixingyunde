import pandas as pd
import openpyxl

def excel_to_validation_rules(excel_path, sheet_name):
    # 读取Excel
    df = pd.read_excel(excel_path, sheet_name=sheet_name)
    
    # 定义需要的列名（根据你的Excel表头）
    col_item = "補正項目名"
    col_attr = "属性"
    col_length = "Length"
    col_required = "必須チェック"
    col_precision = "PrimaryKeyClusteringKj"  # AU列：总精度（数字总位数）
    col_scale = "Length"                     # AV列：小数位
    
    rules_list = []
    idx = 0
    
    for _, row in df.iterrows():
        item_name = str(row[col_item]).strip()
        if not item_name or pd.isna(item_name):
            continue
        
        # 必填判断（○=True，×=False）
        is_required = row[col_required] == "○"
        
        # 属性判断（仅支持文字列/数字）
        attr = str(row[col_attr]).strip()
        if attr not in ["文字列", "数字"]:
            print(f"跳过不支持的类型: {item_name} ({attr})")
            continue
        
        # 数字类型：从AU和AV列获取精度和小数位
        if attr == "数字":
            precision = row[col_precision]
            scale = row[col_scale]
            if pd.notna(precision) and pd.notna(scale):
                precision = int(float(precision))
                scale = int(float(scale))
                common_check = f"number_{precision}_{scale}"
            else:
                #  fallback：如果精度/小数位为空，用Length列作为总位数，小数位0
                length = row[col_length]
                length = int(float(length)) if pd.notna(length) else 0
                common_check = f"number_{length}_0"
            
            rule = {
                "type": "number",
                "common_check": common_check
            }
            if is_required:
                rule["is_required"] = True
        
        # 文字列类型：用Length列作为最大长度
        else:  # 文字列
            length = row[col_length]
            length = int(float(length)) if pd.notna(length) else 0
            rule = {
                "type": "string",
                "max_len": length
            }
            if is_required:
                rule["is_required"] = True
        
        rules_list.append((item_name, rule, idx))
        idx += 1
    
    return rules_list

def print_and_save_rules(rules_list, output_file="validation_rules.py"):
    """
    按你要求的格式输出：
    - 注释行：# 項目名：{项目名}
    - 规则行：{序号}: {规则字典}
    """
    with open(output_file, "w", encoding="utf-8") as f:
        f.write("validation_rules = {\n")
        for item_name, rule, idx in rules_list:
            # 注释行
            f.write(f'    # 項目名：{item_name}\n')
            # 规则行
            rule_str = ", ".join([f'"{k}": {v}' if isinstance(v, (int, bool)) else f'"{k}": "{v}"' for k, v in rule.items()])
            f.write(f'    {idx}: {{{rule_str}}},\n')
        f.write("}\n")
    
    # 控制台打印
    print("=" * 60)
    print("validation_rules = {")
    for item_name, rule, idx in rules_list:
        print(f'    # 項目名：{item_name}')
        rule_str = ", ".join([f'"{k}": {v}' if isinstance(v, (int, bool)) else f'"{k}": "{v}"' for k, v in rule.items()])
        print(f'    {idx}: {{{rule_str}}},')
    print("}")
    print("=" * 60)
    print(f"\n规则已保存到: {output_file}")

# 使用示例
if __name__ == "__main__":
    # --------------------------
    # 【请修改这里的路径和表名】
    # --------------------------
    EXCEL_PATH = "你的Excel文件路径.xlsx"  # 例如: "C:/Users/xxx/Desktop/補正項目一覧.xlsx"
    SHEET_NAME = "你的工作表名"            # 例如: "Sheet1"
    
    try:
        rules = excel_to_validation_rules(EXCEL_PATH, SHEET_NAME)
        print_and_save_rules(rules)
    except Exception as e:
        print(f"运行出错: {str(e)}")
