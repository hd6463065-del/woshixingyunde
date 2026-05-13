from snowflake.snowpark.types import (
    StructType,
    StructField,
    StringType,
    LongType
)

schema = StructType([
    StructField(
        key,
        LongType() if key == "HOSEI_VER" else StringType()
    )
    for key in target_keys_en
])