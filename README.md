# USDA-Food-Database
this project's aim is to apply some data cleaning and to induce some changes in the way the data is structured in order to make it more usable

### Data source and description
The US Department of Agriculture (USDA) makes available a database of food nutrient informations, each food has a list of identifying attributesalong with two lists of nutrients and
portion sizes. Data in this form is not particularly amenable to analysis, so we need to do some work to wrangle the data into a better form the data is in JSON format
data source [download here](https://fdc.nal.usda.gov/download-datasets)

### Data Wrangling and Preparation
, first of all i import the database
```python
improt json
db=json.load(open("database.json"))
```
since this data is not so much usable in it's current state, we have to induce some changes as each entry in the database is a dictionnary containing informations about
 a certain food and in the food's dictionnary there's a field "nutrients" it's a list containing dictionnaries one for each nutrient, so we create separate databases from our db one for the nutrients another one for the
 foods and i'm not gonna take the whole fields existing in the list but only the food names, group, ID, and manufacturer
 ```python
nutrients=pd.DataFrame(db[0]["nutrients"])
info_keys = ["description", "group", "id", "manufacturer"]
info = pd.DataFrame(db, columns=info_keys)
info
```

now to do some analysis on the nutrients table it would be easier to do so if i had all the nutrients for each food in a single table, some steps need to be taken in order to achieve this
first of all : I’ll convert each list of food nutrients to a DataFrame, add a column for the food id, and append the DataFrame to a list. Then, these can be concatenated with concat , since originally there's no
id in the nutrient's dictionnaries
```python
nutrients = []
for rec in db:
    fnuts = pd.DataFrame(rec["nutrients"])
    fnuts["id"] = rec["id"]
    nutrients.append(fnuts)
nutrients = pd.concat(nutrients, ignore_index=True)
```
there is some duplicates in the nutrients so it would be better to jsut drop them
```python
nutrients.duplicated().sum() 
nutrients = nutrients.drop_duplicates()
nutrients.duplicated().sum()
```
there is two columns which have identical names in the two table (food table and nutrients table ) so i change the names of these two columns for the sake of clarity, before i procede to merge the two tables
``` python
col_mapping={description:food,
                   group:fgroup}
info=info.rename(columns=col_mapping,copy=False)
col_mapping={description:nutrient,
                    group:nutgroup}
nutrient.rename(columns=col_mapping,copy=False)
ndata=pd.merge(nutrients,info,on="id")
```
i then do a simple graphe that shows the mediane values of a specific nutrient(zinc in this graph) in each food group for this to be done i group by the
nutrient i want to know in which food groups it's abundant and by the food group of course
```python
result = ndata.groupby(["nutrient", "fgroup"])["value"].quantile(0.5)
result["Zinc, Zn"].sort_values().plot(kind="barh")
```
the resulting figure is this :

figure 1 : median zinc values by food group : 

<img width="866" height="418" alt="cp5" src="https://github.com/user-attachments/assets/f430fcfd-2f85-4914-a983-4ee9a72a885a" />

or if i'm interested in the most dense foods for a specific type of nutrient i can use the idxmax or argmax Series methods 

```python
by_nutrient = ndata.groupby(["nutgroup", "nutrient"])
def get_maximum(x):
    return x.loc[x.value.idxmax()]
max_foods = by_nutrient.apply(get_maximum,include_groups=False)[["value", "food"]]
max_foods["food"] = max_foods["food"].str[:50]
```
with the above operation ne we can get tables of the foods having the densiest amounts of whatever nutrient we want 
```python
max_foods.loc["Amino Acids"]["food"]
max_foods.loc["Vitamins"]["food"]















