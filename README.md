if "HOSEI_VER" in key_df.columns:
    key_df["HOSEI_VER"] = key_df["HOSEI_VER"].apply(
        lambda x: x if pd.isna(x) or str(x).isdigit() else None
    )
