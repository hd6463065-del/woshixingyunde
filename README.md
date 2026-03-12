# ===================== 【登録処理】基础データ補正管理テーブル写入 =====================
def insert_hosei_management_data(df, selected_table_en, correction_reason, user_info):
    """
    写入基礎データ補正管理テーブル（设计书要求）
    :param df: 校验通过的Excel数据
    :param selected_table_en: 选中表英文名
    :param correction_reason: 修正事由
    :param user_info: 用户信息（dict，包含user_id, user_name, shop_no等）
    :return: (是否成功, 消息/错误)
    """
    # 1. 定义補正管理テーブル字段（按设计书）
    HOSEI_TABLE = "基礎データ補正管理"  # 替换为你的实际表名（英文/日文）
    # 2. 建立DB连接
    try:
        db_conn = common.get_db_connection()
        cursor = db_conn.cursor()
        db_conn.autocommit = False  # 开启事务
    except Exception as e:
        return False, f"DB连接失败：{str(e)}"

    try:
        # 3. 生成補正ID（示例：年月日+随机数，可替换为你的规则）
        import random
        hosei_id = f"{datetime.now().strftime('%Y%m%d%H%M%S')}{random.randint(1000, 9999)}"
        
        # 4. 遍历数据批量插入
        table_ja_name = {  # 表英文名→日文名映射（按你的实际配置）
            "recv_contract_month": "受信契約明細_月次",
            "loan_contract_month": "貸出契約明細_月次",
            # ... 补充其他表的日文名 ...
        }.get(selected_table_en, selected_table_en)

        for idx, row in df.iterrows():
            # 4.1 组装登録数据（按设计书字段）
            shori_kbn = str(row.get("SHORI_KBN", "")).strip()
            # 補正方式（追加/更新/削除）
            hosei_hoshiki = "追加" if shori_kbn == "1" else "更新/削除" if shori_kbn == "0" else "不明"
            # 主键值（拼接复合主键）
            primary_keys = TABLE_PRIMARY_KEY_MAP[selected_table_en]
            key_values = "_".join([str(row[pk]).strip() for pk in primary_keys])
            
            # 4.2 插入SQL（按你的实际表字段调整）
            insert_sql = f"""
            INSERT INTO `{HOSEI_TABLE}` (
                補正ID, 対象テーブル_英名, 対象テーブル_和名, 補正方式, 
                レコードキー値, 処理区分, 補正事由, 
                補正者ID, 補正者名, 補正者管理店番, 
                補正日時, 削除フラグ
            ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
            """
            # 4.3 执行插入
            cursor.execute(insert_sql, [
                hosei_id,
                selected_table_en,
                table_ja_name,
                hosei_hoshiki,
                key_values,
                shori_kbn,
                correction_reason,
                user_info["user_id"],
                user_info["user_name"],
                user_info["shop_no"],
                datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                "0"  # 0=未删除，1=已删除
            ])

        # 5. 提交事务
        db_conn.commit()
        return True, f"登録成功！補正ID：{hosei_id}"

    except Exception as e:
        # 失败回滚
        db_conn.rollback()
        error_msg = f"456EBR0070: 登録失敗 → {str(e)}"  # 设计书错误ID
        return False, error_msg
    finally:
        # 关闭连接
        cursor.close()
        db_conn.close()



        @st.dialog("", dismissible=False, width="small")
def confirm():
    with e2:
        st.write("\u3000\u3000\u3000\u3000補正データの登録を行いますか？")
    a1, a2, a3 = st.columns([0.5, 0.4, 1])
    with a2:
        if st.button("はい", type="primary"):
            # --------------------------
            # 新增：调用登録函数（核心）
            # --------------------------
            user_info = {
                "user_id": ss.get("user_id", "UNKNOWN"),
                "user_name": ss.get("user_name", "匿名"),
                "shop_no": ss.get("shop_no", "0000")
            }
            success, msg = insert_hosei_management_data(
                df=ss.valid_data,
                selected_table_en=selected_table_en,
                correction_reason=ss.correction_reason,
                user_info=user_info
            )
            
            if success:
                # 登録成功：保持你原来的成功提示
                ss.showlog = False
                st.success("補正データの登録が完了しました。精査者へ精査依頼をしてください。")
                # 重置状态（清空数据）
                ss.valid_data = None
                ss.validation_errors = []
                ss.correction_reason = ""
                st.rerun()
            else:
                # 登録失败：显示错误
                st.error(f"登録失敗：{msg}")
                ss.showlog = False  # 关闭弹窗
                st.rerun()

    with a3:
        if st.button("いいえ"):
            ss.showlog = False
            st.rerun()
