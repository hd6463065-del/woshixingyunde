import streamlit as st
import openpyxl
from openpyxl.styles import Font
from openpyxl.utils import get_column_letter

# 页面标题
st.title("Excel 字体检查工具 (BIZ UDP Gothic)")

# 上传文件
uploaded_file = st.file_uploader("请上传 Excel 文件", type=["xlsx"])

if uploaded_file is not None:
    # 加载工作簿
    wb = openpyxl.load_workbook(uploaded_file, data_only=False)
    st.success(f"成功加载文件：{uploaded_file.name}")

    # 选择工作表
    sheet_name = st.selectbox("选择要检查的工作表", wb.sheetnames)
    ws = wb[sheet_name]

    # 开始检查
    st.subheader("检查结果")
    error_cells = []

    # 遍历所有有字体信息的单元格
    for row in ws.iter_rows():
        for cell in row:
            if cell.font:
                font_name = cell.font.name
                # 兼容两种写法：BIZ UDPGothic / BIZ UDP Gothic
                if font_name and "BIZ UDP" not in font_name:
                    cell_pos = f"{get_column_letter(cell.column)}{cell.row}"
                    error_cells.append({
                        "位置": cell_pos,
                        "当前字体": font_name,
                        "内容": cell.value
                    })

    # 展示结果
    if error_cells:
        st.warning(f"发现 {len(error_cells)} 个不符合要求的单元格：")
        # 转成 DataFrame 展示
        import pandas as pd
        df = pd.DataFrame(error_cells)
        st.dataframe(df, use_container_width=True)
    else:
        st.success("✅ 所有单元格字体均为 BIZ UDP Gothic，符合要求！")

    # 下载修正后的文件（可选功能：自动替换字体）
    if st.checkbox("自动修正为 BIZ UDP Gothic"):
        # 遍历并替换字体
        for row in ws.iter_rows():
            for cell in row:
                if cell.font:
                    new_font = Font(
                        name="BIZ UDPGothic",
                        size=cell.font.size,
                        bold=cell.font.bold,
                        italic=cell.font.italic
                    )
                    cell.font = new_font
        # 保存到内存
        from io import BytesIO
        buffer = BytesIO()
        wb.save(buffer)
        buffer.seek(0)
        # 提供下载
        st.download_button(
            label="下载修正后的 Excel",
            data=buffer,
            file_name=f"fixed_{uploaded_file.name}",
            mime="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
        )
