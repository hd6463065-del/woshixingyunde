import pandas as pd
import os

def excel_to_validation_rules(excel_path: str, sheet_name: str = "Sheet1") -> dict:
    """
    从Excel的補正項目一覧生成校验规则字典
    规则说明：
    1. 仅支持「文字列」和「数字」两种类型
    2. 文字列：is_required + max_len + type="string"
    3. 数字：is_required + type="number" + common_check（number_长度_小数位，默认小数位0）
    4. 无日期/英数字等其他检查
    
    参数：
    excel_path: Excel文件路径
    sheet_name: 工作表名称（默认Sheet1）
    
    返回：
    validation_rules: 最终的校验规则字典
    """
    # 1. 检查文件是否存在
    if not os.path.exists(excel_path):
        raise FileNotFoundError(f"Excel文件不存在：{excel_path}")
    
    # 2. 读取Excel数据
    try:
        df = pd.read_excel(excel_path, sheet_name=sheet_name)
    except Exception as e:
        raise Exception(f"读取Excel失败：{str(e)}")
    
    # 3. 定义列名映射（适配你的Excel列名）
    col_mapping = {
        "補正項目名": "item_name",
        "属性": "attr",
        "Length": "length",
        "必須チェック": "is_required_flag",
        "小数位数": "decimal_digits"  # 可选列，无则默认0
    }
    
    # 4. 数据预处理（确保列存在）
    for excel_col, code_col in col_mapping.items():
        if excel_col not in df.columns:
            if code_col == "decimal_digits":
                df[code_col] = 0  # 小数位数列不存在则默认0
            else:
                raise ValueError(f"Excel缺少必要列：{excel_col}")
    
    # 5. 转换为校验规则
    validation_rules = {}
    for _, row in df.iterrows():
        # 提取核心字段
        item_name = row["補正項目名"].strip()
        if not item_name:
            continue  # 跳过空行
        
        # 必填检查（○=True，×=False）
        is_required = row["必須チェック"] == "○"
        
        # 长度（仅数字有效）
        length = row["Length"]
        length = int(length) if pd.notna(length) and str(length).isdigit() else 0
        
        # 属性类型（仅文字列/数字）
        attr = row["属性"].strip()
        if attr not in ["文字列", "数字"]:
            raise ValueError(f"不支持的属性类型：{attr}（仅支持文字列/数字）")
        
        # 小数位数（默认0）
        decimal_digits = row.get("小数位数", 0)
        decimal_digits = int(decimal_digits) if pd.notna(decimal_digits) else 0
        
        # 构建单条规则
        rule = {"is_required": is_required}
        
        # 文字列类型处理
        if attr == "文字列":
            rule["type"] = "string"
            if length > 0:
                rule["max_len"] = length
        
        # 数字类型处理
        elif attr == "数字":
            rule["type"] = "number"
            if length > 0:
                rule["common_check"] = f"number_{length}_{decimal_digits}"
        
        # 加入总规则
        validation_rules[item_name] = rule
    
    return validation_rules

def print_validation_rules(validation_rules: dict):
    """
    按要求格式打印最终规则（一行一条）
    """
    print("=" * 50)
    print("最终生成的校验规则（可直接复制使用）：")
    print("=" * 50)
    print("validation_rules = {")
    for item_name, rule in validation_rules.items():
        # 格式化字典为一行字符串
        rule_str = ", ".join([f'"{k}": {v}' if isinstance(v, (int, bool)) else f'"{k}": "{v}"' for k, v in rule.items()])
        print(f'    "{item_name}": {{{rule_str}}},')
    print("}")
    print("=" * 50)

if __name__ == "__main__":
    # --------------------------
    # 【仅需修改这里的配置】
    # --------------------------
    EXCEL_PATH = "補正項目一覧.xlsx"  # 替换为你的Excel文件路径
    SHEET_NAME = "Sheet1"            # 替换为你的工作表名称
    # 可选：如果Excel没有「小数位数」列，需手动指定部分字段的小数位
    CUSTOM_DECIMAL_DIGITS = {
        "スワップ金額": 7,
        "端数金額": 7,
        "スワップレート": 7,
        "端数レート": 7
    }
    
    try:
        # 1. 生成基础规则
        rules = excel_to_validation_rules(EXCEL_PATH, SHEET_NAME)
        
        # 2. 手动调整小数位数（如果Excel没有「小数位数」列）
        for item_name, decimal in CUSTOM_DECIMAL_DIGITS.items():
            if item_name in rules and rules[item_name]["type"] == "number":
                length = rules[item_name]["common_check"].split("_")[1]
                rules[item_name]["common_check"] = f"number_{length}_{decimal}"
        
        # 3. 打印最终结果
        print_validation_rules(rules)
        
        # 4. 可选：将结果保存到文件
        with open("validation_rules.py", "w", encoding="utf-8") as f:
            f.write("validation_rules = {\n")
            for item_name, rule in rules.items():
                rule_str = ", ".join([f'"{k}": {v}' if isinstance(v, (int, bool)) else f'"{k}": "{v}"' for k, v in rule.items()])
                f.write(f'    "{item_name}": {{{rule_str}}},\n')
            f.write("}\n")
        print("\n规则已保存到：validation_rules.py")
        
    except Exception as e:
        print(f"运行出错：{str(e)}")
