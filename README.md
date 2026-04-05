if success:
    ss.hosei_upload["showlog"] = False
    st.success("補正データの登録が完了しました。精査者へ精査依頼をしてください。")
    ss.hosei_upload["valid_data"] = None
    ss.hosei_upload["validation_errors"] = []
    ss["correction_reason"] = ""
    st.rerun()
else:
    st.error("登録失敗")
    ss.hosei_upload["showlog"] = False
    st.rerun()