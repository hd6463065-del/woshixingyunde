# 从DB取用户信息（按user_id查询，后续登录后替换为真实user_id）
current_user_id = "USER001"  # 登录后替换为实际user_id
cursor.execute("SELECT user_name, shop_no FROM user_info WHERE user_id = %s", (current_user_id,))
user_data = cursor.fetchone()

# 存入session（给后续登录/业务逻辑用），不画面显示
ss.user_name = user_data["user_name"] if user_data else "未知ユーザー"
ss.shop_no = user_data["shop_no"] if user_data else ""
