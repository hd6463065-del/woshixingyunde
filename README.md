if "処理区分" not in df.columns:
                    flash.push("エラー：ファイルに「処理区分」列が存在しません。ファイル形式を確認してください。", "error")
                    ss.show_error = True
                    # 中断后续所有处理
                    flash.show()
                    continue
