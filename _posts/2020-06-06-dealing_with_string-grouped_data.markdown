---
layout: post
title:      "Seperating String in a Column into Multiple Rows"
date:       2020-06-06 18:17:08 -0400
permalink:  dealing_with_string-grouped_data
---


**Blog still in Draft......**

Sometimes, items in  table is catogarized into multiple groups, denoted as a string corresponding to it in a table. Consider for example, a table of various movies, with one of the columns being 'genre'. There are several movies nowadays that belong to multiple genres, for example, The Avengers belong to genres Action, Adventure, and Sci-Fi. But what if we want to do an analysis on single genre? We will have to split this into differnt genres with 3 rows belonging to each. This blog post will guide you through building a function that separates each categories into rows and return the values or columns that you are interested in analysing.

Let us continue with the example of movies and genres. The main table we will be working with is `df_bud_genre`, shown in the table below.

We are interested in analysing the movie budget and gross revenue based on their genres. But first, it is important to understand and list out all the genres that these movies are separated into. We will do this by first stripping the string into a list of genres and then using the `set()` method to find the unique genres.

```
#Creating a list of unique Genres
all_genres = []
for genre in df_bud_genre['genres']:
    #Avoid null values of genre by using if statement
    if genre:
        #Avoid running into error where type(Genre) != string using another if statement
        if type(genre)==str:
            genres = genre.split(',')
            #add all genres for each movie in list all_genres
            for i in range(len(genres)):
                all_genres.append(genres[i])
#list out the unique genres
unique_genres = set(all_genres)
unique_genres
```

Now, here comes the fun part! We start creating our function, which works similar to VLOOKUP. We will basically look for a genre in the table, go right to find the index of that row, and read out the desired value, here, budget or gross, in that row.

```
def genre_bg(a_genre = ''):
    """Takes a genre and returns a DF with data of budget and gross values"""
    
    budget_gross = {}
    
		budget_gross['Genre']=a_genre
		
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

#### Breaking down the code:
First, we create an empty dictionary which takes in the keys 'Genre' - for the unique genre, 'Budget' - list of all budgets corresponding to that genre, and 'Gross' - list of all gross values corresponding to that genre. We then use for loop to create the value lists for each key. The `enumerate()` method splits the series into rows of tuples (index, value). We then check if the genre string contains the genre of interest. If it does, it looks at the index of that row, and then returns the value of budget and gross corresponding to that index.

Lets check out the output.

```
genre_bg(a_genre = 'Action')
```

You may then either analyse each table individually, or, to make a comparative analysis, join these tables using the `concat()` method.

```
tables_list = []
for x in unique_genres:
    df_1 = genre_bg(a_genre = x)
    
    #rearranging the columns in the dataframe
    df_1 = df_1[['Genre', 'Budget', 'Worldwide Gross', 'Domestic Gross']]
    tables_list.append(df_1)
		
		#combining each data frame in "table_list" to form a single data table
table_1 = genre_bg(a_genre = 'Action')
for table in tables_list:
    table_1 = pd.concat([table_1, table])
```

And heres what the final outcome looks like.
