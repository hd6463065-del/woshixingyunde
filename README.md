# 【補充7】処理区分（SHORI_KBN）列存在性チェック
if "SHORI_KBN" not in df.columns:
    flash.push("エラー：ファイルに「SHORI_KBN（処理区分）」列が存在しません。\nファイルの先頭列に「SHORI_KBN」が必要です。ファイル形式を確認してください。", "error")
    ss.show_error = True
    flash.show()
    continue
