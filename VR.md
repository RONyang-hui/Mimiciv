-- 计算每个ICU住院的最初6小时内的液体输入量
WITH fluid_intake_6h AS (
  SELECT stay_id, SUM(amount) AS fluid_amount_6h
  FROM mimiciv_icu.inputevents
  -- 只选择最初6小时内的输入事件
  WHERE endtime - starttime <= INTERVAL '6 hours'
  -- 只选择单位为ml的输入项目
  AND amountuom = 'ml'
  GROUP BY mimiciv_icu.inputevents.stay_id
)

-- 筛选出液体输入量大于5000ml的患者并添加一列
SELECT *, CASE WHEN fluid_amount_6h > 5000 THEN 1 ELSE NULL END AS flag
FROM fluid_intake_6h
-- 这里添加一个WHERE条件，只保留flag=1的行
WHERE fluid_amount_6h > 5000;

-- 这里使用CREATE TABLE AS语句，将结果保存到一张表中
CREATE TABLE mimiciv_derived.kdigo_fluid AS
SELECT stay_id, fluid_amount_6h, flag
FROM fluid_intake_6h
WHERE fluid_amount_6h > 5000;




CREATE TABLE mimiciv_derived.kdigo_final AS
SELECT *
FROM mimiciv_derived.mv_kdigo_uo AS uo
LEFT JOIN mimiciv_derived.kdigo_fluid AS fluid
USING (stay_id)
WHERE flag IS NOT NULL;




-- 用RIGHT JOIN将mimiciv_derived.diureticss和mimiciv_derived.new_crrt合并，用stay_id作为对应项

SELECT *
FROM mimiciv_derived.diureticss
LEFT JOIN mimiciv_derived.new_crrt
ON CAST(mimiciv_derived.diureticss.stay_id AS INTEGER) = mimiciv_derived.new_crrt.stay_id; -- 在这里加上一个类型转换



CREATE TABLE mimiciv_derived.VR_final_crrt AS
SELECT diureticss.*
FROM mimiciv_derived.diureticss
LEFT JOIN (
    SELECT stay_id::text
    FROM mimiciv_derived.diureticss
    GROUP BY stay_id::text
    HAVING COUNT(*) = 1
) non_duplicates ON diureticss.stay_id::text = non_duplicates.stay_id
LEFT JOIN mimiciv_derived.new_crrt ON diureticss.stay_id::integer = new_crrt.stay_id::integer
WHERE non_duplicates.stay_id IS NOT NULL;


CREATE TABLE mimiciv_derived.VR_crr_diu AS
SELECT *
FROM mimiciv_derived.vr_final_crrt
LEFT JOIN mimiciv_derived.new_crrt
ON CAST(mimiciv_derived.vr_final_crrt.stay_id AS INTEGER) = mimiciv_derived.new_crrt.stay_id; 




-- 如果存在 "mimiciv_derived.VR_final_crrt" 表格，则先删除
DROP TABLE IF EXISTS mimiciv_derived.VR_final_crrt;

-- 创建 "mimiciv_derived.VR_final_crrt" 表格，并进行数据插入
CREATE TABLE mimiciv_derived.VR_final_crrt AS
SELECT diureticss.*, nc.new_stay_id, new_crrt.flag_crrt
FROM mimiciv_derived.diureticss
LEFT JOIN (
    SELECT stay_id::text AS new_stay_id
    FROM mimiciv_derived.diureticss
    GROUP BY stay_id::text
    HAVING COUNT(*) = 1
) nc ON diureticss.stay_id::text = nc.new_stay_id
LEFT JOIN mimiciv_derived.new_crrt ON diureticss.stay_id::integer = new_crrt.stay_id::integer
WHERE nc.new_stay_id IS NOT NULL;
# 之后形成了物化视图vr_group

我不希望指定某一个列标题，因为列标题太多了，我只想排除掉多余的hadm_id



DROP TABLE IF EXISTS mimiciv_derived.VR_final_icustay;
CREATE TABLE mimiciv_derived.VR_final_icustay AS 
SELECT vr_group.*, icustays.first_careunit
FROM mimiciv_derived.vr_group
LEFT JOIN mimiciv_icu.icustays ON vr_group.stay_id::integer = icustays.stay_id;
