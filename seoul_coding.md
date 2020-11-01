# 서울시 코딩 & 교습소 개수

```python
# READ EXCEL 
import pandas as pd
import re
import os

business_type_list = ["학원", "교습소"]
default_map_svg = 'data/SEOUL_SIG.html'

total_coding_count_per_cities = pd.Series([]) # a list of names of (학원, 교습소)
final_city_color_map = [] # a list of info list 

# GET UNIQUE NAMES AND COUNT
def get_count_per_city (sheet_dfs):
    # INIT VARIABLES
    count_per_city = pd.Series([], dtype=object)
    cities = pd.Series([], dtype=object)

    print("Start reading sheet dataframe...\n")
    
    for address_type, name_type, sheet_name, df in sheet_dfs:
        #print("Address type: ", address_type)
        #print("Name type: ", name_type)
        #print("Sheet name: ", sheet_name)
        
        # get unique name and count from the data 
        city_count_df = df.groupby(address_type)[name_type].nunique()
        city_name_df = df.groupby(address_type)[name_type].unique()
        
        # none unique
        #city_count_df = df.groupby(address_type)[name_type].size()
        #city_name_df = df.groupby(address_type)[name_type]

        count_per_city = count_per_city.append(city_count_df)
        cities = cities.append(city_name_df)
    print("Finish reading sheet dataframe\n\n")

    print("Print unique cities...")
    for idx, name_list in cities.iteritems():
        print()
        print("City({}): {}".format(len(name_list), idx))
        for name in name_list:
            print ("\tName: ", name)

    print("\nPrint unique names DONE")
    return count_per_city
    
# data: series for city names
def set_color_for_cities(city_names):
    # SET COLOR FOR CITIES
    #COLORS = ['#CCE0FF', '#99C2FF', '#663AFF', '#3385FF', '#0066FF', '#0052CC', '#003D99']
    COLORS = ['#CCE0FF', '#99C2FF', '#3385FF', '#0066FF', '#0052CC','#003D99']

    print(os.path.abspath(default_map_svg))
    info_list = []
    print(city_names)

    quantiles = city_names.quantile([.5, .6, .7, .8, .9])
    print("Quantiles:\n", quantiles)

    with open(default_map_svg, 'r') as f:
        for line in f:
            if "LCD" in line:
                gu_id = re.findall('LCD\d+', line)[0][3:]
                gu_name = re.findall('\>(.*)\<', line)[-1]
                print("[{}] {}".format(gu_name, gu_id))
                city_names = city_names.fillna(0)
                try:
                    print("===============")
                    print("city names[gu_name]: ", city_names[gu_name])
                    print("===============")

                    if city_names[gu_name] >= quantiles[.9]:
                        print ("~ 10%: {}".format(gu_name))
                        gu_color = COLORS[5]
                    elif city_names[gu_name] >= quantiles[.8]:
                        print ("~ 20%: {}".format(gu_name))
                        gu_color = COLORS[4]
                    elif city_names[gu_name] >= quantiles[.7]:
                        print ("~ 30%: {}".format(gu_name))
                        gu_color = COLORS[3]
                    elif city_names[gu_name] >= quantiles[.6]:
                        print ("~ 40%: {}".format(gu_name))
                        gu_color = COLORS[2]
                    elif city_names[gu_name] >= quantiles[.5]:
                        print ("~ 50%: {}".format(gu_name))
                        gu_color = COLORS[1]
                    else:
                        print ("~ 100%: {}".format(gu_name))
                        gu_color = COLORS[0]
                except Exception as e:
                    print("Exception: ", str(e))
                    gu_color = "#CCE0FF"
                info_list.append((gu_id, gu_name, gu_color))

    print ("SET COLOR FOR CITIES DONE")
    return info_list
               
    
# CREATE HTML
def create_html(m_gu_info_list, map_output_html=None):
    id_list = [item[0] for item in m_gu_info_list]
    color_list = [item[2] for item in m_gu_info_list]
    
    print("id list: ", id_list)
    print("color list: ", color_list)
    
    idx = 0
    found_id = False
    
    # create write file
    if map_output_html is None:
        map_output_html = "data/output_{}.html".format(business_type)
        
    wf = open(map_output_html, 'w')
    
    # read original html
    with open(default_map_svg, 'r') as f:
        for line in f:
            if not found_id:
                wf.write(line)
                if line.startswith("#") and (line.split(' '))[0] == "#CD" + id_list[idx]:
                    found_id = True
            else:
                wf.write("    fill: {}\n".format(color_list[idx]))
                found_id = False
                idx+=1
    wf.close()
    f.close()

    print("CREATE HTML DONE")


# FILTER 
def filter(df):
    df[address_type] = (df[address_type].str.replace('서울특별시', '').str.lstrip().str.split(' ')).str.get(0)

    if business_type == "학원":

        df = df[ (df["교습과목(반)"].str.contains\
                  ('|'.join(["코딩", "프로그래밍", "프로그래", "C언어", "JAVA",\
                           "HTML", "컴퓨터", "COMPUTER", \
                           "CODING", "인공지능", "로봇",\
                          "엔트리", "스크래치", "아두이노"]),flags=re.IGNORECASE, na=False)\
                  & (df["학원종류"] != "평생직업교육학원"))]
        
        df = df [~df["교습과목(반)"].str.contains\
                 ('|'.join(["음악", "컴퓨터사용안함", "컴퓨터미사용"]))]
    else:
        df = df[ (df["교습과목(반)"].str.contains\
                  ('|'.join(["코딩", "프로그래밍", "프로그래", "C언어", "JAVA",\
                           "HTML", "컴퓨터", "COMPUTER", \
                           "CODING", "인공지능", "로봇",\
                          "엔트리", "스크래치", "아두이노"]),flags=re.IGNORECASE, na=False))]
        df = df [~df["교습과목(반)"].str.contains\
                 ('|'.join(["음악", "컴퓨터사용안함", "컴퓨터미사용"]))]

    return df
```

# 서울시 코딩 & 교습소 개수 총합
```python
total_file_path = "data/final_SEOUL_SIG_total.html"
#print("total names: ", total_names.to_string())
total_gu_info_list = set_color_for_cities(total_coding_count_per_cities)
print("total gu info list: ", total_gu_info_list)
create_html(total_gu_info_list, total_file_path)
```

# 구글 트렌드 (코딩에 대한 관심)
```python
# -*- coding: utf-8 -*-
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib as mpl
import matplotlib.pyplot as plt
import matplotlib.font_manager as fm
mpl.rcParams['axes.unicode_minus'] = False
%matplotlib inline

print ('버전: ', mpl.__version__)
print ('설치 위치: ', mpl.__file__)
print ('설정 위치: ', mpl.get_configdir())
print ('캐시 위치: ', mpl.get_cachedir())
print ('설정파일 위치: ', mpl.matplotlib_fname())

font_name = fm.FontProperties(fname="/Users/macbook/Downloads/NanumFontSetup_TTF_GOTHIC/NanumGothic.ttf").get_name()
plt.rcParams["font.family"] = font_name
#plt.rcParams["font.size"] = 20
plt.rcParams["figure.figsize"] = (16,6)

df = pd.read_csv("data/google_trend_coding.csv")
df["관심도"].astype("int32")
df = df.set_index("주")
print("Raw data")
print(df)

window_size = 12
# get moving average
moving_avg = df["관심도"].rolling(window_size).mean()

week_interval = 8

# create a label
pos_list = []
labels = []
for pos, idx in enumerate(df.index):
    if pos%week_interval == 0 or pos == len(df.index)-1:
        pos_list.append(pos)
        labels.append(idx)
        
sns.lineplot(data=df)
sns.lineplot(data=moving_avg, label="이동평균선 (분기)")

plt.title("Google Trends 관심도\n(키워드='코딩', 국가='대한민국', 기간='2015 ~ 2020')", fontsize=20)
plt.xticks(pos_list, labels, rotation='vertical')
plt.show()
```
