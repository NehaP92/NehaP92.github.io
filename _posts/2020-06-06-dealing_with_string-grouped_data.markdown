---
layout: post
title:      "Dealing with String-Grouped Data"
date:       2020-06-06 22:17:07 +0000
permalink:  dealing_with_string-grouped_data
---


**Blog still in Draft......**

```
#Separating the Genre String into separate genres
df_bud_genre['genres'][1].split(',')
```
['Action', 'Adventure', 'Fantasy']

```
def genre_bg(a_genre = ''):
    """Takes a genre and returns a DF with data of budget and gross values"""
    
    budget_gross = {}
    
    budget_gross['Budget']=[]
    for index, genre in enumerate(df_bud_genre['genres'].str.contains(a_genre)):
        if genre:
            budget_gross['Budget'].append(df_bud_genre['production_budget'][index])


    budget_gross['Worldwide Gross']=[]
    for index, genre in enumerate(df_bud_genre['genres'].str.contains(a_genre)):
        if genre:
            budget_gross['Worldwide Gross'].append(df_bud_genre['worldwide_gross'][index])


    budget_gross['Domestic Gross']=[]
    for index, genre in enumerate(df_bud_genre['genres'].str.contains(a_genre)):
        if genre:
            budget_gross['Domestic Gross'].append(df_bud_genre['domestic_gross'][index])


    budget_gross = pd.DataFrame(budget_gross)
    return budget_gross
```
