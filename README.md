from snowflake.snowpark.functions import col, lit, coalesce

target_spdf = (
    session.table(f"{ref_db}.{ref_schema}.{ref_table}")
    .with_column("_hit", lit(1))
)

joined_spdf = (
    filter_spdf.join(
        target_spdf,
        on=target_keys_en,
        how="left"
    )
    .with_column("IS_EXIST", coalesce(col("_hit"), lit(0)))
    .drop("_hit")
)