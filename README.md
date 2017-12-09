# Pandas_HW

#observations

#1. There seems to be no coorelation between age and the $ spent, little worrysome for parents.

#2. Although the sample size is small, the normalized average purchase being higher for females is interesting. Since its a F2P game, this may hint that women are less likely than men to play for items

#3. It is concerning that the top spenders are ones who purchase either 1 or 2 items, doesnt seem to have too much repeatability.


#dependencies
import pandas as pd
import os

json_path = os.path.join('purchase_data2.json')
file = pd.read_json(json_path)


#Player Count
#* Total Number of Players

player_counts = file["SN"].nunique()
player_df = pd.DataFrame([player_counts])
player_df

# **Purchasing Analysis (Total)**

# * Number of Unique Items
unique_items = file["Item Name"].nunique()

# * Average Purchase Price
#avgprice = file["Price"].groupby(file["Item Name"]).mean()
avgprice = '${:,.2f}'.format(file["Price"].mean())

# * Total Number of Purchases
purchases = file[file["Price"] > 0].count()["Price"]

# * Total Revenue
total_rev = '${:,.2f}'.format(file["Price"].sum())
#total_rev_format = '${:,.2f}'.format(total_rev)

# print('There are',str(unique_items), 'unique items.')
# print('The average price per item is',str(avgprice) + '.')
# print('The total revenue is', str(total_rev)+'.')
# total_rev_format

pa = {'#ofUnq': [unique_items], 'Avgprice': [avgprice], 'total#purchase': [purchases],'total_rev': [total_rev]  }

purchaseA = pd.DataFrame(pa)
purchaseA


#SexDF delete duplicates
test_df = file[["Gender", "SN"]]
nodup_test_df = test_df.drop_duplicates()

#Population total
Total = nodup_test_df["Gender"].count()
# * Percentage and Count of Male Players
Males = nodup_test_df[nodup_test_df["Gender"] == 'Male'].count()["Gender"]
Percentage_Male = Males / Total
pmale = '{:,.2f}%'.format(Percentage_Male * 100)

# * Percentage and Count of Female Players
Females = nodup_test_df[nodup_test_df["Gender"] == 'Female'].count()["Gender"]
Percentage_Female = Females / Total
pfemale = '{:,.2f}%'.format(Percentage_Female * 100)

# * Percentage and Count of Other / Non-Disclosed
Neither = Total - Males - Females
Percentage_other = Neither / Total
pother = '{:,.2f}%'.format(Percentage_other * 100)

# nodup_test_df = test_df.drop_duplicates()
# group_test = nodup_test_df.groupby(['Gender']).count()


ga = {'Gender': ['Males', 'Females', 'Neither'], 'Percentage of players': [pmale, pfemale, pother], 'Total_Count': [Males, Females, Neither]}

gendA = pd.DataFrame(ga)
gendA


# * The below each broken by gender
#   * Purchase Count
Gender_df = file[["Gender", "Price"]]
gender_count = Gender_df.groupby(['Gender']).count
Total_C = Gender_df["Price"].count()
Male_C = (Gender_df[Gender_df["Gender"] == 'Male'].count()["Price"])
Female_C = (Gender_df[Gender_df["Gender"] == 'Female'].count()["Price"])
Other_C = Total_C - Male_C - Female_C

#   * Average Purchase Price
avgprice_g = Gender_df["Price"].groupby(Gender_df["Gender"]).mean()
Total_C = Gender_df["Price"].count()
Male_A = (Gender_df[Gender_df["Gender"] == 'Male'].mean()["Price"])
Female_A = (Gender_df[Gender_df["Gender"] == 'Female'].mean()["Price"])
Other_A = (Gender_df[~Gender_df["Gender"].isin(['Male', 'Female'])]).mean()["Price"]
Male_AF = '${:,.2f}'.format(Male_A)
Female_AF = '${:,.2f}'.format(Female_A)
Other_AF = '${:,.2f}'.format(Other_A)

#   * Total Purchase Value
# sumprice_g = Gender_df["Price"].groupby(Gender_df["Gender"]).sum()

Male_S = (Gender_df[Gender_df["Gender"] == 'Male'].sum()["Price"])
Female_S = (Gender_df[Gender_df["Gender"] == 'Female'].sum()["Price"])
Other_S = (Gender_df[~Gender_df["Gender"].isin(['Male', 'Female'])]).sum()["Price"]
#   * Normalized Totals (normalizing for the # of people in each gender group)
Male_P = (Gender_df[Gender_df["Gender"] == 'Male'].sum()["Price"])/ Males
Female_P = (Gender_df[Gender_df["Gender"] == 'Female'].sum()["Price"])/Females
Other_P = (Gender_df[~Gender_df["Gender"].isin(['Male', 'Female'])]).sum()["Price"]/Neither

Male_PF = '${:,.2f}'.format(Male_P)
Female_PF = '${:,.2f}'.format(Female_P)
Other_PF = '${:,.2f}'.format(Other_P)

gga = {'1. Gender': ['Males', 'Females', 'Undisclosed/Other'],
      '2. Purchase Count': [Male_C, Female_C, Other_C], '3. Avg Purchase Price': [Male_AF, Female_AF, Other_AF],
       '4. Purchase Sum': [Male_S, Female_S, Other_S],'5. Avg Purchase Norm': [Male_PF, Female_PF, Other_PF],}

ggendA = pd.DataFrame(gga)
ggendA


# * The below each broken into bins of 4 years (i.e. &lt;10, 10-14, 15-19, etc.) 
bins = [0, 4, 8, 12, 16, 20, 24, 28, 32, 36, 40]
group_names = ['a', 'b', 'c', 'd','e','f','h', 'i', 'j', 'k']
Age_df = file[["Age", "Price", "SN"]]


pd.cut(Age_df["Age"], bins, labels=group_names)
Age_df["Age_Group"] = pd.cut(Age_df["Age"],
                                           bins, labels=group_names)

Age_df2 = Age_df.drop_duplicates(subset = ['SN'])
Age_nom = Age_df2['Age_Group'].groupby(Age_df2['Age_Group']).count()

#number of real players per age group

# #   * Purchase Count
age_count = Age_df['Price'].groupby(Age_df['Age_Group']).count()

# #   * Average Purchase Price
age_mean = Age_df['Price'].groupby(Age_df['Age_Group']).mean()
# #   * Total Purchase Value
age_sum = Age_df['Price'].groupby(Age_df['Age_Group']).sum()
# #   * Normalized Totals (normalizing for the # of people in each age group)
age_norm = age_sum / Age_nom

age_pr = (Age_nom / Age_nom.sum())



dfag = pd.concat([Age_nom,age_pr, age_mean, age_sum, age_norm], axis=1)

dfag.columns = ['Count','Percentage', 'Avg purchase price', 'Total purchase price', 'Normalized']

dfag.Percentage = dfag.Percentage.astype(float).fillna(0.0) * 100
dfag['Percentage'] = dfag['Percentage'].map('{:,.2f}%'.format)
dfag['Avg purchase price'] = dfag['Avg purchase price'].astype(float).fillna(0.0)
dfag['Total purchase price'] = dfag['Total purchase price'].astype(float).fillna(0.0)
dfag['Normalized'] = dfag['Normalized'].astype(float).fillna(0.0)
dfag['Avg purchase price'] = dfag['Avg purchase price'].map('${:,.2f}'.format)
dfag['Total purchase price'] = dfag['Total purchase price'].map('${:,.2f}'.format)
dfag['Normalized'] = dfag['Normalized'].map('${:,.2f}'.format)

dfag

# * Identify the the top 5 spenders in the game by total purchase value, then list (in a table):
#   * SN
#   * Purchase Count
#   * Average Purchase Price
#   * Total Purchase Value
topspend = file.groupby(["SN"])["Price"].agg("sum")
topspendc = file.groupby(["SN"])["Price"].agg("count")
topspendm = file.groupby(["SN"])["Price"].agg("mean")
# topspendsort = topspend.iloc[np.lexsort([topspend.values], order = "desc")]
# topspendsort
tdf = pd.concat([topspend, topspendc, topspendm], axis=1)
# data.iloc[:,0]


tdf.columns = ['Total','Count', 'Avg']
tdf['Total'] = tdf['Total'].map('${:,.2f}'.format)
tdf['Avg'] = tdf['Avg'].map('${:,.2f}'.format)

tdf.sort_values(by=['Total'], ascending=False).head()


# * Identify the 5 most popular items by purchase count, then list (in a table):
#   * Item ID
#   * Item Name
#   * Purchase Count
#   * Item Price
#   * Total Purchase Value
itopspend = file.groupby(["Item Name"])["Price"].agg("sum")
itopspendc = file.groupby(["Item Name"])["Price"].agg("count")
itopspendm = file.groupby(["Item Name"])["Price"].agg("mean")
# topspendsort = topspend.iloc[np.lexsort([topspend.values], order = "desc")]
# topspendsort
itdf = pd.concat([itopspend, itopspendc, itopspendm], axis=1)
# data.iloc[:,0]

itdf.columns = ['Total','Count', 'Price']
itdf['Total'] = itdf['Total'].map('${:,.2f}'.format)
itdf['Price'] = itdf['Price'].map('${:,.2f}'.format)
itdf.sort_values(by=['Count'], ascending=False).head()


# * Identify the 5 most profitable items by total purchase value, then list (in a table):
#   * Item ID
#   * Item Name
#   * Purchase Count
#   * Item Price
#   * Total Purchase Value
itdf.sort_values(by=['Total'], ascending=False).head()

