---
layout: post
title:      "Seperating String Values in a Column into Multiple Rows"
date:       2020-06-06 18:17:08 -0400
permalink:  dealing_with_string-grouped_data
---


Sometimes, the different elements of a data table are categorized into multiple groups indicated corresponding to each element as a string in a column. Consider, for example, an informative data table about various movies over a period of time. One of the data table columns indicates the 'genre' of a movie. Movies nowadays belong to multiple genres. For instance, "The Avengers" can be categorized under action, adventure, and sci-fi. The 'genre' column of the data-table may use a comma-separated string to define this. 

But what if we want to analyze individual genres? One way is to split the string and separate the genres into multiple rows. A function that separates each genre/category into multiple rows and returns the corresponding values of interest could go a long way when dealing with several categories.

Let us continue with the example of movies and their corresponding genres. The main table we will be working with is `df_bud_genre`, shown in the table below.

We are interested in analyzing how the movie budgets and gross revenue trends differ with each genre. We start by first understanding and listing out all the genres that the given movies are separated into by stripping the string of genres into a list, thereafter, using the `set()` method to find all the unique genres.

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

Now for the fun part, we start creating our function, which works similar to VLOOKUP, that will look for a genre in the table, go right to find the index of that row, and read out the desired value ( here, budget, and gross revenue) corresponding to the index.

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



**Breaking down the code:**
An empty dictionary is created that takes in the keys 'Genre' - for the unique genre of interest, 'Budget' - list of all budgets corresponding to that genre, and 'Gross' - list of all gross values corresponding to that genre. We then used a for-loop to generate the dictionary values as a list of corresponding budget and gross values.

The `enumerate()` method splits the series into rows of tuples (index, value). We utilized this method to check if the genre string contains the genre of interest. If it does, it looks at the index of that row and then returns the value of budget and gross corresponding to that index.



Lets have a look at what the output looks like.

```
genre_bg(a_genre = 'Action')
```

![](https://raw.githubusercontent.com/NehaP92/dsc-mod-1-project-v2-1-onl01-dtsc-pt-041320/master/genre_bg(a_genre%20Action)_output.png)

You may then either analyze each table individually or, make a comparative analysis by joining these tables using the `concat()` method.

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

And thats what the final outcome looks like.

![](https://raw.githubusercontent.com/NehaP92/dsc-mod-1-project-v2-1-onl01-dtsc-pt-041320/master/table_1%20all%20genre%20output.png)

Happy analysing! 
