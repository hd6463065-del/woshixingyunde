import pandas as pd
import re

def generate_validation_rules_from_excel(excel_path: str, sheet_name: str) -> dict:
    """
    从Excel的補正項目一覧生成校验规则字典，严格遵循三种check的圈叉规则
    规则：
    - 必須チェック: ○=True, ×=不包含该键
    - 最大桁数チェック: ○=包含长度校验, ×=不包含
      - 文字列: max_len = Length列的值
      - 数字: common_check = "number_整数位_小数位"，小数位默认0
    - データ型チェック: ○=包含type校验, ×=不包含
      - 文字列: type="alphanumeric"
      - 数字: type="number"
    """
    # 读取Excel
    df = pd.read_excel(excel_path, sheet_name=sheet_name)
    
    validation_rules = {}
    idx = 0
    
    for _, row in df.iterrows():
        item_name = str(row["補正項目名"]).strip()
        if not item_name or item_name == "nan":
            continue
        
        rule = {}
        
        # 1. 必須チェック
        required_flag = str(row["必須チェック"]).strip()
        if required_flag == "○":
            rule["is_required"] = True
        
        # 2. データ型チェック
        data_type_flag = str(row["データ型チェック"]).strip()
        attr = str(row["属性"]).strip()
        if data_type_flag == "○":
            if attr == "文字列":
                rule["type"] = "alphanumeric"
            elif attr == "数字":
                rule["type"] = "number"
        
        # 3. 最大桁数チェック
        max_digit_flag = str(row["最大桁数チェック"]).strip()
        length_str = str(row["Length"]).strip()
        if max_digit_flag == "○":
            if attr == "文字列":
                # 文字列：提取纯数字作为max_len
                max_len = int(re.sub(r"[^0-9]", "", length_str))
                rule["max_len"] = max_len
            elif attr == "数字":
                # 数字：处理整数位和小数位
                if "," in length_str:
                    int_part, dec_part = length_str.split(",")
                    int_len = int(int_part)
                    dec_len = int(dec_part)
                else:
                    int_len = int(length_str)
                    dec_len = 0
                rule["common_check"] = f"number_{int_len}_{dec_len}"
        
        # 只有当规则不为空时才加入字典
        if rule:
            validation_rules[idx] = rule
            idx += 1
    
    return validation_rules

# 使用示例
if __name__ == "__main__":
    # 替换为你的Excel路径和工作表名
    excel_path = "補正項目一覧.xlsx"
    sheet_name = "受信契約明細_月次"
    
    rules = generate_validation_rules_from_excel(excel_path, sheet_name)
    
    # 输出为图片三的格式
    print("validation_rules = {")
    for k, v in rules.items():
        rule_str = str(v).replace("'", '"')
        print(f'  {k}: {rule_str},')
    print("}")
