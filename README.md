row_count = len(rows)
     exec_time = int((time.time() - start) * 1000)  # 单位转成ms，和错误分支保持一致
     
     runner._add_log(
         f"DB存在チェック {selected_table}",
         "SUCCESS",
         f"件数: {row_count}, コスト {exec_time}ms",
         ""  # 成功时可以把错误信息留空
     )