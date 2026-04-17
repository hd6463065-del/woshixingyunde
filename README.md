if "hosei_upload" not in ss:
    ss.hosei_upload = {
        "show_error": False,
        "showlog": False,
        "validation_errors": [],
        "valid_data": None,
        "display_errors": [],
        "messages": {
            "error": "",
            "success": ""
        },
        "last_selected_table": None
    }


def clear_message():
    ss.hosei_upload["messages"] = {
        "error": "",
        "success": ""
    }

def clear_validation_state():
    ss.hosei_upload["show_error"] = False
    ss.hosei_upload["validation_errors"] = []
    ss.hosei_upload["display_errors"] = []
    ss.hosei_upload["valid_data"] = None


messages = ss.hosei_upload["messages"]

if messages.get("success"):
    st.success(messages["success"])

if messages.get("error"):
    st.error(messages["error"])


selected_option = st.selectbox(
    "補正対象テーブル",
    options=table_options,
    format_func=lambda x: x[0],
    index=0,
    width=280
)
selected_table_name = selected_option[0]
selected_table = selected_option[1]

if ss.hosei_upload["last_selected_table"] != selected_table:
    clear_message()
    clear_validation_state()
    ss.hosei_upload["last_selected_table"] = selected_table



