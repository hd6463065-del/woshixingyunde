if not simple_flag or not full_flag:
    if not simple_flag:
        errors.append(simple_err_msg)
    if not full_flag:
        errors.append(full_err_msg)
    return pd.DataFrame(), "{}", {}, False, errors