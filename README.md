=IF(OR(H2="○",I2="○",J2="○"),
 "{" &
 TEXTJOIN(", ", TRUE,
  IF(H2="○","""is_required"": True",""),
  IF(AND(J2="○",F2="NUMBER"),
   """common_check"": ""number_" & 
   IF(ISNUMBER(SEARCH(",",E2)),
    SUBSTITUTE(E2,",","_"),
    E2 & "_0"
   ) & """",
   ""
  ),
  IF(AND(J2="○",F2<>"NUMBER"),"""max_len"": "&E2,""),
  IF(I2="○","""type"": """&LOWER(F2)&"""","")
 ) &
 "}",
 ""
)
