if "HOSEI_VER" in key_df.columns:
    key_df["HOSEI_VER"] = (
        pd.to_numeric(
            key_df["HOSEI_VER"],
            errors="coerce"
        )
        .astype("Int64")
    )