
# PyCity Schools Analysis

## Three observation trends based on the data

1. [School Type] Whole average values are not so different(less than 1 point/ 1%) but 5 bottom performing schools (by passing rate) are all Charter.  
2. [Spending Ranges Per Student] No positive correlation between Spending Ranges Per Student and Passing Rates, rather than there might be negative correlation.
3. [School Size] Smaller schools likely to have higher average or rate than larger ones.


```python
# Import Dependencies
import pandas as pd
import os
import numpy as np

# Path to resource files
path = "resources"

# Genarate a student df
students_df = pd.read_csv(os.path.join(path, 'students_complete.csv'))

# Generate a school df changed the column name 'name' to 'school'
school_df = pd.read_csv(os.path.join(path, 'schools_complete.csv')).rename(columns={"name": "school"})

# Merge two dfs into one df
stu_sch_df = pd.merge(students_df, school_df, how = "outer")
# stu_sch_df.head()
```

## District Summary


```python
# generate a new dataframe for schools "dist_sch_df" in whicn rows "type" is "District" are collected
dist_sch_df = school_df[school_df["type"] == "District"]

# generate a new dataframe for students "dist_stu_sch_df" in whicn rows "type" is "District" are collected
dist_stu_sch_df = stu_sch_df[stu_sch_df["type"] == "District"]

# group "dist_stu_sch_df" (for students) by type
distByType = stu_sch_df.groupby("type")

# assign the computed numbers to the fields
totalStu = len(dist_stu_sch_df.index)
math_passed = len(dist_stu_sch_df[dist_stu_sch_df["math_score"] >= 60].index)
read_passed = len(dist_stu_sch_df[dist_stu_sch_df["reading_score"] >= 60].index)

# create a new df with computed data
dist_summary = pd.DataFrame({
    "Total Schools":[len(dist_sch_df.index)],
    "Total Students":[totalStu],
    "Total Budget":[distByType.sum().loc["District","budget"]],
    "Average Math Score":[distByType.mean().loc["District","math_score"]],
    "Average Reading Score":[distByType.mean().loc["District","reading_score"]],
    "% Passing Math":[math_passed/totalStu],
    "% Passing Reading":[read_passed/totalStu],
    "% Overall Passing Rate":[(math_passed + read_passed)/(totalStu*2)]
})

# format and show the df
dist_summary[[
    "Total Schools",
    "Total Students",
    "Total Budget",
    "Average Math Score",
    "Average Reading Score",
    "% Passing Math",
    "% Passing Reading",
    "% Overall Passing Rate"
]].style.format({
    "Total Budget":"${:,d}",
    "Average Math Score":"{:.2f}",
    "Average Reading Score":"{:.2f}",
    "% Passing Math":"{:.2%}",
    "% Passing Reading":"{:.2%}",
    "% Overall Passing Rate":"{:.2%}"
})

```




<style  type="text/css" >
</style>  
<table id="T_35888d90_2335_11e8_b841_784f4376ba08" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Total Schools</th> 
        <th class="col_heading level0 col1" >Total Students</th> 
        <th class="col_heading level0 col2" >Total Budget</th> 
        <th class="col_heading level0 col3" >Average Math Score</th> 
        <th class="col_heading level0 col4" >Average Reading Score</th> 
        <th class="col_heading level0 col5" >% Passing Math</th> 
        <th class="col_heading level0 col6" >% Passing Reading</th> 
        <th class="col_heading level0 col7" >% Overall Passing Rate</th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_35888d90_2335_11e8_b841_784f4376ba08level0_row0" class="row_heading level0 row0" >0</th> 
        <td id="T_35888d90_2335_11e8_b841_784f4376ba08row0_col0" class="data row0 col0" >7</td> 
        <td id="T_35888d90_2335_11e8_b841_784f4376ba08row0_col1" class="data row0 col1" >26976</td> 
        <td id="T_35888d90_2335_11e8_b841_784f4376ba08row0_col2" class="data row0 col2" >$70,439,053,973</td> 
        <td id="T_35888d90_2335_11e8_b841_784f4376ba08row0_col3" class="data row0 col3" >76.99</td> 
        <td id="T_35888d90_2335_11e8_b841_784f4376ba08row0_col4" class="data row0 col4" >80.96</td> 
        <td id="T_35888d90_2335_11e8_b841_784f4376ba08row0_col5" class="data row0 col5" >89.03%</td> 
        <td id="T_35888d90_2335_11e8_b841_784f4376ba08row0_col6" class="data row0 col6" >100.00%</td> 
        <td id="T_35888d90_2335_11e8_b841_784f4376ba08row0_col7" class="data row0 col7" >94.52%</td> 
    </tr></tbody> 
</table> 



## School Summary


```python
# generate a new df for students grouped by "school"
groupBySch = stu_sch_df.groupby("school")

# assign the computed numbers to the fields
passedStu_mathbySch = stu_sch_df[stu_sch_df["math_score"] >= 60].groupby("school").count()["Student ID"]
passedStu_readbySch = stu_sch_df[stu_sch_df["reading_score"] >= 60].groupby("school").count()["Student ID"]

# generate a new df to show the results
sch_summ_df = groupBySch.mean()

# assign computed data to sch_summ_df
sch_summ_df["School Type"] = list(school_df["type"])
sch_summ_df["Total Students"] = groupBySch.count()["Student ID"]
sch_summ_df["Per Student Budget"] = sch_summ_df["budget"] / sch_summ_df["Total Students"]
sch_summ_df["Average Math Score"] = groupBySch.mean()["math_score"]
sch_summ_df["Average Reading Score"] = groupBySch.mean()["reading_score"]
sch_summ_df["% Passing Math"] = passedStu_mathbySch / sch_summ_df["Total Students"]
sch_summ_df["% Passing Reading"] = passedStu_readbySch / sch_summ_df["Total Students"]
sch_summ_df["% Overall Passing Rate"] = (passedStu_mathbySch + passedStu_readbySch)/(sch_summ_df["Total Students"]*2)

# format and show the result
sch_summ = sch_summ_df.rename(columns={"budget": "Total School Budget"})[[
    "School Type",
    "Total Students",
    "Total School Budget",
    "Per Student Budget",
    "Average Math Score",
    "Average Reading Score",
    "% Passing Math",
    "% Passing Reading",
    "% Overall Passing Rate"
    
]]
sch_summ.style.format({
    "Total School Budget":"${:,.0f}",
    "Per Student Budget":"${:,.0f}",
    "Average Math Score":"{:.2f}",
    "Average Reading Score":"{:.2f}",
    "% Passing Math":"{:.2%}",
    "% Passing Reading":"{:.2%}",
    "% Overall Passing Rate":"{:.2%}"
    
})

```




<style  type="text/css" >
</style>  
<table id="T_359e2718_2335_11e8_911c_784f4376ba08" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >School Type</th> 
        <th class="col_heading level0 col1" >Total Students</th> 
        <th class="col_heading level0 col2" >Total School Budget</th> 
        <th class="col_heading level0 col3" >Per Student Budget</th> 
        <th class="col_heading level0 col4" >Average Math Score</th> 
        <th class="col_heading level0 col5" >Average Reading Score</th> 
        <th class="col_heading level0 col6" >% Passing Math</th> 
        <th class="col_heading level0 col7" >% Passing Reading</th> 
        <th class="col_heading level0 col8" >% Overall Passing Rate</th> 
    </tr>    <tr> 
        <th class="index_name level0" >school</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_359e2718_2335_11e8_911c_784f4376ba08level0_row0" class="row_heading level0 row0" >Bailey High School</th> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row0_col0" class="data row0 col0" >District</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row0_col1" class="data row0 col1" >4976</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row0_col2" class="data row0 col2" >$3,124,928</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row0_col3" class="data row0 col3" >$628</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row0_col4" class="data row0 col4" >77.05</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row0_col5" class="data row0 col5" >81.03</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row0_col6" class="data row0 col6" >89.53%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row0_col7" class="data row0 col7" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row0_col8" class="data row0 col8" >94.76%</td> 
    </tr>    <tr> 
        <th id="T_359e2718_2335_11e8_911c_784f4376ba08level0_row1" class="row_heading level0 row1" >Cabrera High School</th> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row1_col0" class="data row1 col0" >District</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row1_col1" class="data row1 col1" >1858</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row1_col2" class="data row1 col2" >$1,081,356</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row1_col3" class="data row1 col3" >$582</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row1_col4" class="data row1 col4" >83.06</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row1_col5" class="data row1 col5" >83.98</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row1_col6" class="data row1 col6" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row1_col7" class="data row1 col7" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row1_col8" class="data row1 col8" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_359e2718_2335_11e8_911c_784f4376ba08level0_row2" class="row_heading level0 row2" >Figueroa High School</th> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row2_col0" class="data row2 col0" >Charter</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row2_col1" class="data row2 col1" >2949</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row2_col2" class="data row2 col2" >$1,884,411</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row2_col3" class="data row2 col3" >$639</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row2_col4" class="data row2 col4" >76.71</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row2_col5" class="data row2 col5" >81.16</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row2_col6" class="data row2 col6" >88.44%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row2_col7" class="data row2 col7" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row2_col8" class="data row2 col8" >94.22%</td> 
    </tr>    <tr> 
        <th id="T_359e2718_2335_11e8_911c_784f4376ba08level0_row3" class="row_heading level0 row3" >Ford High School</th> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row3_col0" class="data row3 col0" >District</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row3_col1" class="data row3 col1" >2739</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row3_col2" class="data row3 col2" >$1,763,916</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row3_col3" class="data row3 col3" >$644</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row3_col4" class="data row3 col4" >77.10</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row3_col5" class="data row3 col5" >80.75</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row3_col6" class="data row3 col6" >89.30%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row3_col7" class="data row3 col7" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row3_col8" class="data row3 col8" >94.65%</td> 
    </tr>    <tr> 
        <th id="T_359e2718_2335_11e8_911c_784f4376ba08level0_row4" class="row_heading level0 row4" >Griffin High School</th> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row4_col0" class="data row4 col0" >Charter</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row4_col1" class="data row4 col1" >1468</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row4_col2" class="data row4 col2" >$917,500</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row4_col3" class="data row4 col3" >$625</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row4_col4" class="data row4 col4" >83.35</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row4_col5" class="data row4 col5" >83.82</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row4_col6" class="data row4 col6" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row4_col7" class="data row4 col7" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row4_col8" class="data row4 col8" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_359e2718_2335_11e8_911c_784f4376ba08level0_row5" class="row_heading level0 row5" >Hernandez High School</th> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row5_col0" class="data row5 col0" >Charter</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row5_col1" class="data row5 col1" >4635</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row5_col2" class="data row5 col2" >$3,022,020</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row5_col3" class="data row5 col3" >$652</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row5_col4" class="data row5 col4" >77.29</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row5_col5" class="data row5 col5" >80.93</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row5_col6" class="data row5 col6" >89.08%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row5_col7" class="data row5 col7" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row5_col8" class="data row5 col8" >94.54%</td> 
    </tr>    <tr> 
        <th id="T_359e2718_2335_11e8_911c_784f4376ba08level0_row6" class="row_heading level0 row6" >Holden High School</th> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row6_col0" class="data row6 col0" >Charter</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row6_col1" class="data row6 col1" >427</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row6_col2" class="data row6 col2" >$248,087</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row6_col3" class="data row6 col3" >$581</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row6_col4" class="data row6 col4" >83.80</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row6_col5" class="data row6 col5" >83.81</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row6_col6" class="data row6 col6" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row6_col7" class="data row6 col7" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row6_col8" class="data row6 col8" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_359e2718_2335_11e8_911c_784f4376ba08level0_row7" class="row_heading level0 row7" >Huang High School</th> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row7_col0" class="data row7 col0" >District</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row7_col1" class="data row7 col1" >2917</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row7_col2" class="data row7 col2" >$1,910,635</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row7_col3" class="data row7 col3" >$655</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row7_col4" class="data row7 col4" >76.63</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row7_col5" class="data row7 col5" >81.18</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row7_col6" class="data row7 col6" >88.86%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row7_col7" class="data row7 col7" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row7_col8" class="data row7 col8" >94.43%</td> 
    </tr>    <tr> 
        <th id="T_359e2718_2335_11e8_911c_784f4376ba08level0_row8" class="row_heading level0 row8" >Johnson High School</th> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row8_col0" class="data row8 col0" >Charter</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row8_col1" class="data row8 col1" >4761</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row8_col2" class="data row8 col2" >$3,094,650</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row8_col3" class="data row8 col3" >$650</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row8_col4" class="data row8 col4" >77.07</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row8_col5" class="data row8 col5" >80.97</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row8_col6" class="data row8 col6" >89.18%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row8_col7" class="data row8 col7" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row8_col8" class="data row8 col8" >94.59%</td> 
    </tr>    <tr> 
        <th id="T_359e2718_2335_11e8_911c_784f4376ba08level0_row9" class="row_heading level0 row9" >Pena High School</th> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row9_col0" class="data row9 col0" >Charter</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row9_col1" class="data row9 col1" >962</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row9_col2" class="data row9 col2" >$585,858</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row9_col3" class="data row9 col3" >$609</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row9_col4" class="data row9 col4" >83.84</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row9_col5" class="data row9 col5" >84.04</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row9_col6" class="data row9 col6" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row9_col7" class="data row9 col7" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row9_col8" class="data row9 col8" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_359e2718_2335_11e8_911c_784f4376ba08level0_row10" class="row_heading level0 row10" >Rodriguez High School</th> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row10_col0" class="data row10 col0" >Charter</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row10_col1" class="data row10 col1" >3999</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row10_col2" class="data row10 col2" >$2,547,363</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row10_col3" class="data row10 col3" >$637</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row10_col4" class="data row10 col4" >76.84</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row10_col5" class="data row10 col5" >80.74</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row10_col6" class="data row10 col6" >88.55%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row10_col7" class="data row10 col7" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row10_col8" class="data row10 col8" >94.27%</td> 
    </tr>    <tr> 
        <th id="T_359e2718_2335_11e8_911c_784f4376ba08level0_row11" class="row_heading level0 row11" >Shelton High School</th> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row11_col0" class="data row11 col0" >District</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row11_col1" class="data row11 col1" >1761</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row11_col2" class="data row11 col2" >$1,056,600</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row11_col3" class="data row11 col3" >$600</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row11_col4" class="data row11 col4" >83.36</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row11_col5" class="data row11 col5" >83.73</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row11_col6" class="data row11 col6" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row11_col7" class="data row11 col7" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row11_col8" class="data row11 col8" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_359e2718_2335_11e8_911c_784f4376ba08level0_row12" class="row_heading level0 row12" >Thomas High School</th> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row12_col0" class="data row12 col0" >District</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row12_col1" class="data row12 col1" >1635</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row12_col2" class="data row12 col2" >$1,043,130</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row12_col3" class="data row12 col3" >$638</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row12_col4" class="data row12 col4" >83.42</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row12_col5" class="data row12 col5" >83.85</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row12_col6" class="data row12 col6" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row12_col7" class="data row12 col7" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row12_col8" class="data row12 col8" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_359e2718_2335_11e8_911c_784f4376ba08level0_row13" class="row_heading level0 row13" >Wilson High School</th> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row13_col0" class="data row13 col0" >District</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row13_col1" class="data row13 col1" >2283</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row13_col2" class="data row13 col2" >$1,319,574</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row13_col3" class="data row13 col3" >$578</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row13_col4" class="data row13 col4" >83.27</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row13_col5" class="data row13 col5" >83.99</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row13_col6" class="data row13 col6" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row13_col7" class="data row13 col7" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row13_col8" class="data row13 col8" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_359e2718_2335_11e8_911c_784f4376ba08level0_row14" class="row_heading level0 row14" >Wright High School</th> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row14_col0" class="data row14 col0" >Charter</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row14_col1" class="data row14 col1" >1800</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row14_col2" class="data row14 col2" >$1,049,400</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row14_col3" class="data row14 col3" >$583</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row14_col4" class="data row14 col4" >83.68</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row14_col5" class="data row14 col5" >83.95</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row14_col6" class="data row14 col6" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row14_col7" class="data row14 col7" >100.00%</td> 
        <td id="T_359e2718_2335_11e8_911c_784f4376ba08row14_col8" class="data row14 col8" >100.00%</td> 
    </tr></tbody> 
</table> 



## Top Performing Schools (By Passing Rate)


```python
# sort, format, and show the df
sch_summ.sort_values(by=["% Overall Passing Rate", "% Passing Math"], ascending=False).head().style.format({
    "Total School Budget":"${:,.0f}",
    "Per Student Budget":"${:,.0f}",
    "Average Math Score":"{:.2f}",
    "Average Reading Score":"{:.2f}",
    "% Passing Math":"{:.2%}",
    "% Passing Reading":"{:.2%}",
    "% Overall Passing Rate":"{:.2%}"
    
})
```




<style  type="text/css" >
</style>  
<table id="T_35a25efa_2335_11e8_8052_784f4376ba08" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >School Type</th> 
        <th class="col_heading level0 col1" >Total Students</th> 
        <th class="col_heading level0 col2" >Total School Budget</th> 
        <th class="col_heading level0 col3" >Per Student Budget</th> 
        <th class="col_heading level0 col4" >Average Math Score</th> 
        <th class="col_heading level0 col5" >Average Reading Score</th> 
        <th class="col_heading level0 col6" >% Passing Math</th> 
        <th class="col_heading level0 col7" >% Passing Reading</th> 
        <th class="col_heading level0 col8" >% Overall Passing Rate</th> 
    </tr>    <tr> 
        <th class="index_name level0" >school</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_35a25efa_2335_11e8_8052_784f4376ba08level0_row0" class="row_heading level0 row0" >Cabrera High School</th> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row0_col0" class="data row0 col0" >District</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row0_col1" class="data row0 col1" >1858</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row0_col2" class="data row0 col2" >$1,081,356</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row0_col3" class="data row0 col3" >$582</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row0_col4" class="data row0 col4" >83.06</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row0_col5" class="data row0 col5" >83.98</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row0_col6" class="data row0 col6" >100.00%</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row0_col7" class="data row0 col7" >100.00%</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row0_col8" class="data row0 col8" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_35a25efa_2335_11e8_8052_784f4376ba08level0_row1" class="row_heading level0 row1" >Griffin High School</th> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row1_col0" class="data row1 col0" >Charter</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row1_col1" class="data row1 col1" >1468</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row1_col2" class="data row1 col2" >$917,500</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row1_col3" class="data row1 col3" >$625</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row1_col4" class="data row1 col4" >83.35</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row1_col5" class="data row1 col5" >83.82</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row1_col6" class="data row1 col6" >100.00%</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row1_col7" class="data row1 col7" >100.00%</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row1_col8" class="data row1 col8" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_35a25efa_2335_11e8_8052_784f4376ba08level0_row2" class="row_heading level0 row2" >Holden High School</th> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row2_col0" class="data row2 col0" >Charter</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row2_col1" class="data row2 col1" >427</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row2_col2" class="data row2 col2" >$248,087</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row2_col3" class="data row2 col3" >$581</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row2_col4" class="data row2 col4" >83.80</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row2_col5" class="data row2 col5" >83.81</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row2_col6" class="data row2 col6" >100.00%</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row2_col7" class="data row2 col7" >100.00%</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row2_col8" class="data row2 col8" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_35a25efa_2335_11e8_8052_784f4376ba08level0_row3" class="row_heading level0 row3" >Pena High School</th> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row3_col0" class="data row3 col0" >Charter</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row3_col1" class="data row3 col1" >962</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row3_col2" class="data row3 col2" >$585,858</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row3_col3" class="data row3 col3" >$609</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row3_col4" class="data row3 col4" >83.84</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row3_col5" class="data row3 col5" >84.04</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row3_col6" class="data row3 col6" >100.00%</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row3_col7" class="data row3 col7" >100.00%</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row3_col8" class="data row3 col8" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_35a25efa_2335_11e8_8052_784f4376ba08level0_row4" class="row_heading level0 row4" >Shelton High School</th> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row4_col0" class="data row4 col0" >District</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row4_col1" class="data row4 col1" >1761</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row4_col2" class="data row4 col2" >$1,056,600</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row4_col3" class="data row4 col3" >$600</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row4_col4" class="data row4 col4" >83.36</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row4_col5" class="data row4 col5" >83.73</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row4_col6" class="data row4 col6" >100.00%</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row4_col7" class="data row4 col7" >100.00%</td> 
        <td id="T_35a25efa_2335_11e8_8052_784f4376ba08row4_col8" class="data row4 col8" >100.00%</td> 
    </tr></tbody> 
</table> 



## Bottom Performing Schools (By Passing Rate)


```python
# sort, format, and show the df
sch_summ.sort_values(by=["% Overall Passing Rate", "% Passing Math"]).head().style.format({
    "Total School Budget":"${:,.0f}",
    "Per Student Budget":"${:,.0f}",
    "Average Math Score":"{:.2f}",
    "Average Reading Score":"{:.2f}",
    "% Passing Math":"{:.2%}",
    "% Passing Reading":"{:.2%}",
    "% Overall Passing Rate":"{:.2%}"
    
})
```




<style  type="text/css" >
</style>  
<table id="T_35a581ac_2335_11e8_a60b_784f4376ba08" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >School Type</th> 
        <th class="col_heading level0 col1" >Total Students</th> 
        <th class="col_heading level0 col2" >Total School Budget</th> 
        <th class="col_heading level0 col3" >Per Student Budget</th> 
        <th class="col_heading level0 col4" >Average Math Score</th> 
        <th class="col_heading level0 col5" >Average Reading Score</th> 
        <th class="col_heading level0 col6" >% Passing Math</th> 
        <th class="col_heading level0 col7" >% Passing Reading</th> 
        <th class="col_heading level0 col8" >% Overall Passing Rate</th> 
    </tr>    <tr> 
        <th class="index_name level0" >school</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_35a581ac_2335_11e8_a60b_784f4376ba08level0_row0" class="row_heading level0 row0" >Figueroa High School</th> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row0_col0" class="data row0 col0" >Charter</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row0_col1" class="data row0 col1" >2949</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row0_col2" class="data row0 col2" >$1,884,411</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row0_col3" class="data row0 col3" >$639</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row0_col4" class="data row0 col4" >76.71</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row0_col5" class="data row0 col5" >81.16</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row0_col6" class="data row0 col6" >88.44%</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row0_col7" class="data row0 col7" >100.00%</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row0_col8" class="data row0 col8" >94.22%</td> 
    </tr>    <tr> 
        <th id="T_35a581ac_2335_11e8_a60b_784f4376ba08level0_row1" class="row_heading level0 row1" >Rodriguez High School</th> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row1_col0" class="data row1 col0" >Charter</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row1_col1" class="data row1 col1" >3999</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row1_col2" class="data row1 col2" >$2,547,363</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row1_col3" class="data row1 col3" >$637</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row1_col4" class="data row1 col4" >76.84</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row1_col5" class="data row1 col5" >80.74</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row1_col6" class="data row1 col6" >88.55%</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row1_col7" class="data row1 col7" >100.00%</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row1_col8" class="data row1 col8" >94.27%</td> 
    </tr>    <tr> 
        <th id="T_35a581ac_2335_11e8_a60b_784f4376ba08level0_row2" class="row_heading level0 row2" >Huang High School</th> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row2_col0" class="data row2 col0" >District</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row2_col1" class="data row2 col1" >2917</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row2_col2" class="data row2 col2" >$1,910,635</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row2_col3" class="data row2 col3" >$655</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row2_col4" class="data row2 col4" >76.63</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row2_col5" class="data row2 col5" >81.18</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row2_col6" class="data row2 col6" >88.86%</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row2_col7" class="data row2 col7" >100.00%</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row2_col8" class="data row2 col8" >94.43%</td> 
    </tr>    <tr> 
        <th id="T_35a581ac_2335_11e8_a60b_784f4376ba08level0_row3" class="row_heading level0 row3" >Hernandez High School</th> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row3_col0" class="data row3 col0" >Charter</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row3_col1" class="data row3 col1" >4635</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row3_col2" class="data row3 col2" >$3,022,020</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row3_col3" class="data row3 col3" >$652</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row3_col4" class="data row3 col4" >77.29</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row3_col5" class="data row3 col5" >80.93</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row3_col6" class="data row3 col6" >89.08%</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row3_col7" class="data row3 col7" >100.00%</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row3_col8" class="data row3 col8" >94.54%</td> 
    </tr>    <tr> 
        <th id="T_35a581ac_2335_11e8_a60b_784f4376ba08level0_row4" class="row_heading level0 row4" >Johnson High School</th> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row4_col0" class="data row4 col0" >Charter</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row4_col1" class="data row4 col1" >4761</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row4_col2" class="data row4 col2" >$3,094,650</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row4_col3" class="data row4 col3" >$650</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row4_col4" class="data row4 col4" >77.07</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row4_col5" class="data row4 col5" >80.97</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row4_col6" class="data row4 col6" >89.18%</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row4_col7" class="data row4 col7" >100.00%</td> 
        <td id="T_35a581ac_2335_11e8_a60b_784f4376ba08row4_col8" class="data row4 col8" >94.59%</td> 
    </tr></tbody> 
</table> 



## Math Scores by Grade


```python
# generate a new df to show the result
mathScore_sch_grade = pd.DataFrame({
    "9th":stu_sch_df[stu_sch_df["grade"] == "9th"].groupby("school").mean()["math_score"],
    "10th":stu_sch_df[stu_sch_df["grade"] == "10th"].groupby("school").mean()["math_score"],
    "11th":stu_sch_df[stu_sch_df["grade"] == "11th"].groupby("school").mean()["math_score"],
    "12th":stu_sch_df[stu_sch_df["grade"] == "12th"].groupby("school").mean()["math_score"]
})

# format and show the df
mathScore_sch_grade[[
    "9th", "10th", "11th", "12th"
]].style.format("{:.2f}")
```




<style  type="text/css" >
</style>  
<table id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >9th</th> 
        <th class="col_heading level0 col1" >10th</th> 
        <th class="col_heading level0 col2" >11th</th> 
        <th class="col_heading level0 col3" >12th</th> 
    </tr>    <tr> 
        <th class="index_name level0" >school</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08level0_row0" class="row_heading level0 row0" >Bailey High School</th> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row0_col0" class="data row0 col0" >77.08</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row0_col1" class="data row0 col1" >77.00</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row0_col2" class="data row0 col2" >77.52</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row0_col3" class="data row0 col3" >76.49</td> 
    </tr>    <tr> 
        <th id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08level0_row1" class="row_heading level0 row1" >Cabrera High School</th> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row1_col0" class="data row1 col0" >83.09</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row1_col1" class="data row1 col1" >83.15</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row1_col2" class="data row1 col2" >82.77</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row1_col3" class="data row1 col3" >83.28</td> 
    </tr>    <tr> 
        <th id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08level0_row2" class="row_heading level0 row2" >Figueroa High School</th> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row2_col0" class="data row2 col0" >76.40</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row2_col1" class="data row2 col1" >76.54</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row2_col2" class="data row2 col2" >76.88</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row2_col3" class="data row2 col3" >77.15</td> 
    </tr>    <tr> 
        <th id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08level0_row3" class="row_heading level0 row3" >Ford High School</th> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row3_col0" class="data row3 col0" >77.36</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row3_col1" class="data row3 col1" >77.67</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row3_col2" class="data row3 col2" >76.92</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row3_col3" class="data row3 col3" >76.18</td> 
    </tr>    <tr> 
        <th id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08level0_row4" class="row_heading level0 row4" >Griffin High School</th> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row4_col0" class="data row4 col0" >82.04</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row4_col1" class="data row4 col1" >84.23</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row4_col2" class="data row4 col2" >83.84</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row4_col3" class="data row4 col3" >83.36</td> 
    </tr>    <tr> 
        <th id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08level0_row5" class="row_heading level0 row5" >Hernandez High School</th> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row5_col0" class="data row5 col0" >77.44</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row5_col1" class="data row5 col1" >77.34</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row5_col2" class="data row5 col2" >77.14</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row5_col3" class="data row5 col3" >77.19</td> 
    </tr>    <tr> 
        <th id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08level0_row6" class="row_heading level0 row6" >Holden High School</th> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row6_col0" class="data row6 col0" >83.79</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row6_col1" class="data row6 col1" >83.43</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row6_col2" class="data row6 col2" >85.00</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row6_col3" class="data row6 col3" >82.86</td> 
    </tr>    <tr> 
        <th id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08level0_row7" class="row_heading level0 row7" >Huang High School</th> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row7_col0" class="data row7 col0" >77.03</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row7_col1" class="data row7 col1" >75.91</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row7_col2" class="data row7 col2" >76.45</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row7_col3" class="data row7 col3" >77.23</td> 
    </tr>    <tr> 
        <th id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08level0_row8" class="row_heading level0 row8" >Johnson High School</th> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row8_col0" class="data row8 col0" >77.19</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row8_col1" class="data row8 col1" >76.69</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row8_col2" class="data row8 col2" >77.49</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row8_col3" class="data row8 col3" >76.86</td> 
    </tr>    <tr> 
        <th id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08level0_row9" class="row_heading level0 row9" >Pena High School</th> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row9_col0" class="data row9 col0" >83.63</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row9_col1" class="data row9 col1" >83.37</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row9_col2" class="data row9 col2" >84.33</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row9_col3" class="data row9 col3" >84.12</td> 
    </tr>    <tr> 
        <th id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08level0_row10" class="row_heading level0 row10" >Rodriguez High School</th> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row10_col0" class="data row10 col0" >76.86</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row10_col1" class="data row10 col1" >76.61</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row10_col2" class="data row10 col2" >76.40</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row10_col3" class="data row10 col3" >77.69</td> 
    </tr>    <tr> 
        <th id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08level0_row11" class="row_heading level0 row11" >Shelton High School</th> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row11_col0" class="data row11 col0" >83.42</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row11_col1" class="data row11 col1" >82.92</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row11_col2" class="data row11 col2" >83.38</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row11_col3" class="data row11 col3" >83.78</td> 
    </tr>    <tr> 
        <th id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08level0_row12" class="row_heading level0 row12" >Thomas High School</th> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row12_col0" class="data row12 col0" >83.59</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row12_col1" class="data row12 col1" >83.09</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row12_col2" class="data row12 col2" >83.50</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row12_col3" class="data row12 col3" >83.50</td> 
    </tr>    <tr> 
        <th id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08level0_row13" class="row_heading level0 row13" >Wilson High School</th> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row13_col0" class="data row13 col0" >83.09</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row13_col1" class="data row13 col1" >83.72</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row13_col2" class="data row13 col2" >83.20</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row13_col3" class="data row13 col3" >83.04</td> 
    </tr>    <tr> 
        <th id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08level0_row14" class="row_heading level0 row14" >Wright High School</th> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row14_col0" class="data row14 col0" >83.26</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row14_col1" class="data row14 col1" >84.01</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row14_col2" class="data row14 col2" >83.84</td> 
        <td id="T_35b1e1cc_2335_11e8_b41c_784f4376ba08row14_col3" class="data row14 col3" >83.64</td> 
    </tr></tbody> 
</table> 




```python
# generate a new df to show the result
readScore_sch_grade = pd.DataFrame({
    "9th":stu_sch_df[stu_sch_df["grade"] == "9th"].groupby("school").mean()["reading_score"],
    "10th":stu_sch_df[stu_sch_df["grade"] == "10th"].groupby("school").mean()["reading_score"],
    "11th":stu_sch_df[stu_sch_df["grade"] == "11th"].groupby("school").mean()["reading_score"],
    "12th":stu_sch_df[stu_sch_df["grade"] == "12th"].groupby("school").mean()["reading_score"]
})

# format and show the df
readScore_sch_grade[[
    "9th", "10th", "11th", "12th"
]].style.format("{:.2f}")

```




<style  type="text/css" >
</style>  
<table id="T_35be43b8_2335_11e8_9639_784f4376ba08" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >9th</th> 
        <th class="col_heading level0 col1" >10th</th> 
        <th class="col_heading level0 col2" >11th</th> 
        <th class="col_heading level0 col3" >12th</th> 
    </tr>    <tr> 
        <th class="index_name level0" >school</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_35be43b8_2335_11e8_9639_784f4376ba08level0_row0" class="row_heading level0 row0" >Bailey High School</th> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row0_col0" class="data row0 col0" >81.30</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row0_col1" class="data row0 col1" >80.91</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row0_col2" class="data row0 col2" >80.95</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row0_col3" class="data row0 col3" >80.91</td> 
    </tr>    <tr> 
        <th id="T_35be43b8_2335_11e8_9639_784f4376ba08level0_row1" class="row_heading level0 row1" >Cabrera High School</th> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row1_col0" class="data row1 col0" >83.68</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row1_col1" class="data row1 col1" >84.25</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row1_col2" class="data row1 col2" >83.79</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row1_col3" class="data row1 col3" >84.29</td> 
    </tr>    <tr> 
        <th id="T_35be43b8_2335_11e8_9639_784f4376ba08level0_row2" class="row_heading level0 row2" >Figueroa High School</th> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row2_col0" class="data row2 col0" >81.20</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row2_col1" class="data row2 col1" >81.41</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row2_col2" class="data row2 col2" >80.64</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row2_col3" class="data row2 col3" >81.38</td> 
    </tr>    <tr> 
        <th id="T_35be43b8_2335_11e8_9639_784f4376ba08level0_row3" class="row_heading level0 row3" >Ford High School</th> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row3_col0" class="data row3 col0" >80.63</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row3_col1" class="data row3 col1" >81.26</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row3_col2" class="data row3 col2" >80.40</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row3_col3" class="data row3 col3" >80.66</td> 
    </tr>    <tr> 
        <th id="T_35be43b8_2335_11e8_9639_784f4376ba08level0_row4" class="row_heading level0 row4" >Griffin High School</th> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row4_col0" class="data row4 col0" >83.37</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row4_col1" class="data row4 col1" >83.71</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row4_col2" class="data row4 col2" >84.29</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row4_col3" class="data row4 col3" >84.01</td> 
    </tr>    <tr> 
        <th id="T_35be43b8_2335_11e8_9639_784f4376ba08level0_row5" class="row_heading level0 row5" >Hernandez High School</th> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row5_col0" class="data row5 col0" >80.87</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row5_col1" class="data row5 col1" >80.66</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row5_col2" class="data row5 col2" >81.40</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row5_col3" class="data row5 col3" >80.86</td> 
    </tr>    <tr> 
        <th id="T_35be43b8_2335_11e8_9639_784f4376ba08level0_row6" class="row_heading level0 row6" >Holden High School</th> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row6_col0" class="data row6 col0" >83.68</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row6_col1" class="data row6 col1" >83.32</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row6_col2" class="data row6 col2" >83.82</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row6_col3" class="data row6 col3" >84.70</td> 
    </tr>    <tr> 
        <th id="T_35be43b8_2335_11e8_9639_784f4376ba08level0_row7" class="row_heading level0 row7" >Huang High School</th> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row7_col0" class="data row7 col0" >81.29</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row7_col1" class="data row7 col1" >81.51</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row7_col2" class="data row7 col2" >81.42</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row7_col3" class="data row7 col3" >80.31</td> 
    </tr>    <tr> 
        <th id="T_35be43b8_2335_11e8_9639_784f4376ba08level0_row8" class="row_heading level0 row8" >Johnson High School</th> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row8_col0" class="data row8 col0" >81.26</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row8_col1" class="data row8 col1" >80.77</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row8_col2" class="data row8 col2" >80.62</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row8_col3" class="data row8 col3" >81.23</td> 
    </tr>    <tr> 
        <th id="T_35be43b8_2335_11e8_9639_784f4376ba08level0_row9" class="row_heading level0 row9" >Pena High School</th> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row9_col0" class="data row9 col0" >83.81</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row9_col1" class="data row9 col1" >83.61</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row9_col2" class="data row9 col2" >84.34</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row9_col3" class="data row9 col3" >84.59</td> 
    </tr>    <tr> 
        <th id="T_35be43b8_2335_11e8_9639_784f4376ba08level0_row10" class="row_heading level0 row10" >Rodriguez High School</th> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row10_col0" class="data row10 col0" >80.99</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row10_col1" class="data row10 col1" >80.63</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row10_col2" class="data row10 col2" >80.86</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row10_col3" class="data row10 col3" >80.38</td> 
    </tr>    <tr> 
        <th id="T_35be43b8_2335_11e8_9639_784f4376ba08level0_row11" class="row_heading level0 row11" >Shelton High School</th> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row11_col0" class="data row11 col0" >84.12</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row11_col1" class="data row11 col1" >83.44</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row11_col2" class="data row11 col2" >84.37</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row11_col3" class="data row11 col3" >82.78</td> 
    </tr>    <tr> 
        <th id="T_35be43b8_2335_11e8_9639_784f4376ba08level0_row12" class="row_heading level0 row12" >Thomas High School</th> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row12_col0" class="data row12 col0" >83.73</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row12_col1" class="data row12 col1" >84.25</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row12_col2" class="data row12 col2" >83.59</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row12_col3" class="data row12 col3" >83.83</td> 
    </tr>    <tr> 
        <th id="T_35be43b8_2335_11e8_9639_784f4376ba08level0_row13" class="row_heading level0 row13" >Wilson High School</th> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row13_col0" class="data row13 col0" >83.94</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row13_col1" class="data row13 col1" >84.02</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row13_col2" class="data row13 col2" >83.76</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row13_col3" class="data row13 col3" >84.32</td> 
    </tr>    <tr> 
        <th id="T_35be43b8_2335_11e8_9639_784f4376ba08level0_row14" class="row_heading level0 row14" >Wright High School</th> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row14_col0" class="data row14 col0" >83.83</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row14_col1" class="data row14 col1" >83.81</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row14_col2" class="data row14 col2" >84.16</td> 
        <td id="T_35be43b8_2335_11e8_9639_784f4376ba08row14_col3" class="data row14 col3" >84.07</td> 
    </tr></tbody> 
</table> 




```python
# assign bins and labels
bins_spend = [0,585,615,645,675]
group_labels_spend = ["< $585","$585 - $615","$615 - $645","$645 - $675"]

# assign a new column as labels
sch_summ["Spending Ranges (Per Student)"] = pd.cut(sch_summ["Per Student Budget"],bins_spend,labels=group_labels_spend)

# Group by ageRange
sch_spendRange = sch_summ.groupby("Spending Ranges (Per Student)").mean()

# format and show the df
sch_spendRange[[
    "Average Math Score",
    "Average Reading Score",
    "% Passing Math",
    "% Passing Reading",
    "% Overall Passing Rate"
]].style.format({
    "Average Math Score": "{:.2f}",
    "Average Reading Score": "{:.2f}",
    "% Passing Math": "{:.2%}",
    "% Passing Reading": "{:.2%}",
    "% Overall Passing Rate":"{:.2%}"   
})
```




<style  type="text/css" >
</style>  
<table id="T_35cf2c78_2335_11e8_9759_784f4376ba08" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Average Math Score</th> 
        <th class="col_heading level0 col1" >Average Reading Score</th> 
        <th class="col_heading level0 col2" >% Passing Math</th> 
        <th class="col_heading level0 col3" >% Passing Reading</th> 
        <th class="col_heading level0 col4" >% Overall Passing Rate</th> 
    </tr>    <tr> 
        <th class="index_name level0" >Spending Ranges (Per Student)</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_35cf2c78_2335_11e8_9759_784f4376ba08level0_row0" class="row_heading level0 row0" >< $585</th> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row0_col0" class="data row0 col0" >83.46</td> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row0_col1" class="data row0 col1" >83.93</td> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row0_col2" class="data row0 col2" >100.00%</td> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row0_col3" class="data row0 col3" >100.00%</td> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row0_col4" class="data row0 col4" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_35cf2c78_2335_11e8_9759_784f4376ba08level0_row1" class="row_heading level0 row1" >$585 - $615</th> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row1_col0" class="data row1 col0" >83.60</td> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row1_col1" class="data row1 col1" >83.89</td> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row1_col2" class="data row1 col2" >100.00%</td> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row1_col3" class="data row1 col3" >100.00%</td> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row1_col4" class="data row1 col4" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_35cf2c78_2335_11e8_9759_784f4376ba08level0_row2" class="row_heading level0 row2" >$615 - $645</th> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row2_col0" class="data row2 col0" >79.08</td> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row2_col1" class="data row2 col1" >81.89</td> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row2_col2" class="data row2 col2" >92.64%</td> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row2_col3" class="data row2 col3" >100.00%</td> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row2_col4" class="data row2 col4" >96.32%</td> 
    </tr>    <tr> 
        <th id="T_35cf2c78_2335_11e8_9759_784f4376ba08level0_row3" class="row_heading level0 row3" >$645 - $675</th> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row3_col0" class="data row3 col0" >77.00</td> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row3_col1" class="data row3 col1" >81.03</td> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row3_col2" class="data row3 col2" >89.04%</td> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row3_col3" class="data row3 col3" >100.00%</td> 
        <td id="T_35cf2c78_2335_11e8_9759_784f4376ba08row3_col4" class="data row3 col4" >94.52%</td> 
    </tr></tbody> 
</table> 




```python
# assign bins and labels
bins_size = [0,1000,2000,5000]
group_labels_size = ["Small(< 1000)","Medium(1000-2000)","Large(2000-5000)"]

# assign a new column as labels
sch_summ["School Size"] = pd.cut(sch_summ["Total Students"],bins_size,labels=group_labels_size)

# Group by sizeRange
sch_sizeRange = sch_summ.groupby("School Size").mean()

# format and show the df
sch_sizeRange[[
    "Average Math Score",
    "Average Reading Score",
    "% Passing Math",
    "% Passing Reading",
    "% Overall Passing Rate"
]].style.format({
    "Average Math Score": "{:.2f}",
    "Average Reading Score": "{:.2f}",
    "% Passing Math": "{:.2%}",
    "% Passing Reading": "{:.2%}",
    "% Overall Passing Rate":"{:.2%}"   
})
```




<style  type="text/css" >
</style>  
<table id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Average Math Score</th> 
        <th class="col_heading level0 col1" >Average Reading Score</th> 
        <th class="col_heading level0 col2" >% Passing Math</th> 
        <th class="col_heading level0 col3" >% Passing Reading</th> 
        <th class="col_heading level0 col4" >% Overall Passing Rate</th> 
    </tr>    <tr> 
        <th class="index_name level0" >School Size</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08level0_row0" class="row_heading level0 row0" >Small(< 1000)</th> 
        <td id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08row0_col0" class="data row0 col0" >83.82</td> 
        <td id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08row0_col1" class="data row0 col1" >83.93</td> 
        <td id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08row0_col2" class="data row0 col2" >100.00%</td> 
        <td id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08row0_col3" class="data row0 col3" >100.00%</td> 
        <td id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08row0_col4" class="data row0 col4" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08level0_row1" class="row_heading level0 row1" >Medium(1000-2000)</th> 
        <td id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08row1_col0" class="data row1 col0" >83.37</td> 
        <td id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08row1_col1" class="data row1 col1" >83.86</td> 
        <td id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08row1_col2" class="data row1 col2" >100.00%</td> 
        <td id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08row1_col3" class="data row1 col3" >100.00%</td> 
        <td id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08row1_col4" class="data row1 col4" >100.00%</td> 
    </tr>    <tr> 
        <th id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08level0_row2" class="row_heading level0 row2" >Large(2000-5000)</th> 
        <td id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08row2_col0" class="data row2 col0" >77.75</td> 
        <td id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08row2_col1" class="data row2 col1" >81.34</td> 
        <td id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08row2_col2" class="data row2 col2" >90.37%</td> 
        <td id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08row2_col3" class="data row2 col3" >100.00%</td> 
        <td id="T_35d4c7c8_2335_11e8_b9a8_784f4376ba08row2_col4" class="data row2 col4" >95.18%</td> 
    </tr></tbody> 
</table> 




```python
# group by school type, format, and show the df
sch_summ.groupby("School Type").mean()[[
    "Average Math Score",
    "Average Reading Score",
    "% Passing Math",
    "% Passing Reading",
    "% Overall Passing Rate"
]].style.format({
    "Average Math Score": "{:.2f}",
    "Average Reading Score": "{:.2f}",
    "% Passing Math": "{:.2%}",
    "% Passing Reading": "{:.2%}",
    "% Overall Passing Rate":"{:.2%}"   
})
```




<style  type="text/css" >
</style>  
<table id="T_35d8323a_2335_11e8_acfa_784f4376ba08" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Average Math Score</th> 
        <th class="col_heading level0 col1" >Average Reading Score</th> 
        <th class="col_heading level0 col2" >% Passing Math</th> 
        <th class="col_heading level0 col3" >% Passing Reading</th> 
        <th class="col_heading level0 col4" >% Overall Passing Rate</th> 
    </tr>    <tr> 
        <th class="index_name level0" >School Type</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_35d8323a_2335_11e8_acfa_784f4376ba08level0_row0" class="row_heading level0 row0" >Charter</th> 
        <td id="T_35d8323a_2335_11e8_acfa_784f4376ba08row0_col0" class="data row0 col0" >80.32</td> 
        <td id="T_35d8323a_2335_11e8_acfa_784f4376ba08row0_col1" class="data row0 col1" >82.43</td> 
        <td id="T_35d8323a_2335_11e8_acfa_784f4376ba08row0_col2" class="data row0 col2" >94.41%</td> 
        <td id="T_35d8323a_2335_11e8_acfa_784f4376ba08row0_col3" class="data row0 col3" >100.00%</td> 
        <td id="T_35d8323a_2335_11e8_acfa_784f4376ba08row0_col4" class="data row0 col4" >97.20%</td> 
    </tr>    <tr> 
        <th id="T_35d8323a_2335_11e8_acfa_784f4376ba08level0_row1" class="row_heading level0 row1" >District</th> 
        <td id="T_35d8323a_2335_11e8_acfa_784f4376ba08row1_col0" class="data row1 col0" >80.56</td> 
        <td id="T_35d8323a_2335_11e8_acfa_784f4376ba08row1_col1" class="data row1 col1" >82.64</td> 
        <td id="T_35d8323a_2335_11e8_acfa_784f4376ba08row1_col2" class="data row1 col2" >95.38%</td> 
        <td id="T_35d8323a_2335_11e8_acfa_784f4376ba08row1_col3" class="data row1 col3" >100.00%</td> 
        <td id="T_35d8323a_2335_11e8_acfa_784f4376ba08row1_col4" class="data row1 col4" >97.69%</td> 
    </tr></tbody> 
</table> 


