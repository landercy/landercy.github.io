# 公式

## 身体证

- 生日：=--TEXT(MID(A1,7,8),"0-00-00")
- 年月：=CONCAT(MID(A1,7,4),"-",MID(I3,11,2))
- 年龄：=YEAR(TODAY())-MID(A1,7,4)
- 性别：=IF(ISODD(RIGHT(LEFT(A1,17))),"男","女")