  if changed_cols:
        changed_cols_text = "、".join(changed_cols)

        msg = mu.get_messages_data(
            "456ERR3094",
            changed_cols_text
        )

        business_errors.append(
            f"レコード {idx + 1}: {msg}"
        )