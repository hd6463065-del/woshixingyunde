filter_spdf = filter_spdf.with_column("管理店番", try_to_number(col("管理店番")))


from snowflake.snowpark.functions import try_to_number, col