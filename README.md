import time

if flag and len(ss.hosei_upload["validation_errors"]) == 0:
    ss.hosei_upload["valid_data"] = df
    st.success(f"DEBUG success {time.time()}")