# Sent-To-Users
with Sent_to_Users as (
  
select STR_NBR, count (SKU_NUMBER) as Shelf_Out

from MET_BASE.SHELF_OUT_DATA

where thd_fiscal_week = '202235'

and source in('ML_MODEL','REACTIVE_PACKDOWN')

group by STR_NBR

order by STR_NBR),

Packdown as (select STR_NBR, COUNT(SKU_NUMBER) as Packdown

FROM `analytics-met-thd.MET_BASE.SHELF_OUT_DATA` SO

 CROSS JOIN UNNEST(SO.SURVEY) AS QA
 
 where QA.Question_ID = 34 and QA.Answer_Id= 2 and thd_fiscal_week = '202235'

and source in('ML_MODEL','REACTIVE_PACKDOWN')

group by STR_NBR
order by STR_NBR),

Worked as ( select STR_NBR, COUNT(SKU_NUMBER) as Worked

FROM `analytics-met-thd.MET_BASE.SHELF_OUT_DATA` SO

 CROSS JOIN UNNEST(SO.SURVEY) AS QA

where QA.Question_ID = 1 and QA.Answer_Id in (1,2) and thd_fiscal_week = '202235'

and source in('ML_MODEL','REACTIVE_PACKDOWN')

group by STR_NBR
order by STR_NBR), 

Combine1 as (select Sent_to_Users.STR_NBR, Sent_to_Users.Shelf_Out, Packdown.Packdown, Worked.Worked
from Sent_to_Users

left join Packdown on

Sent_to_Users.STR_NBR = Packdown.STR_NBR

left join Worked on

 Sent_to_Users.STR_NBR = Worked.STR_NBR)

 Select combine1.*,`analytics-met-thd.MET_BASE.MV_MET_STR_HIER`.RGN_NM as Regions, (Combine1.Worked/Combine1.Shelf_Out)*100 as Worked_Percentage, (Combine1.Packdown/Combine1.Worked)* 100 as Packdown_Percentage

from combine1

join `analytics-met-thd.MET_BASE.MV_MET_STR_HIER` on 

combine1.STR_NBR = `analytics-met-thd.MET_BASE.MV_MET_STR_HIER`.STR_NBR

group by Regions, Combine1.Shelf_out, combine1.Worked, Combine1.Packdown, Worked_Percentage, Packdown_Percentage,combine1.STR_NBR;






![image](https://user-images.githubusercontent.com/17092274/208141328-634ccb19-7005-4594-9581-ba9ea9fdc369.png)
