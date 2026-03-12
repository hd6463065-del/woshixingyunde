=IF(OR(H3="○",J3="○",L3="○"),
    "{" &
    TEXTJOIN(", ", TRUE,
        IF(H3="○", """" & IF(L3="○", "is_required", "is_required") & """: True""", ""),
        IF(AND(J3="○", F3="NUMBER"), ""common_check"": ""number_" & SUBSTITUTE(E3, ",", "_") & """", ""),
        IF(L3="○", """" & IF(J3="○", "type", "type") & """: ""number""", "")
    ) &
    "}",
""
)
