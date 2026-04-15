except Exception as e:
    exec_time = int((time.time() - start) * 1000)

    runner.add_log(
        f"DB存在チェック {selected_table}",
        "ERROR",
        f"コスト {exec_time}ms",
        str(e)
    )

    mu.push_messages("456ERR0064")
    return {}, False