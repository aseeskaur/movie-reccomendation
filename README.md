# movie-reccomendation
I will be make a reccomendation model based on item based collaborative filtering 

Item Based Collaborative Filtering: reccomendation system based on the similarity between the items calculated using user ratings

Sparse Matrices = Matrices that contain mostly zero entries are called sparse

Example: Whether or not a user has watched a particular movie 

import pandas as pd 
import numpy as np
from scipy.sparse import csr_matrix
from sklearn.neighbors import NearestNeighbors
import matplotlib.pyplot as plt
import seaborn as sns

#reading data

movies = pd.read_csv("movies.csv")
ratings = pd.read_csv("ratings.csv")

movies.head()

ratings.head()

ratings.columns

# pivot function is used to reshape the data to see which userId has rated which movieId

new_df = ratings.pivot(index = "movieId", columns = "userId", values = "rating")

new_df.head()

#NOw let's replace the Nan's by 0

new_df.fillna(0, inplace = True)

new_df.head()

#filtering some data

#movie qualification = at least 7 users have voted for that movie
#user qualification = at least 30 movies should have been voted by that user

num_movie_voted = ratings.groupby("userId")['rating'].agg('count')
num_user_voted = ratings.groupby("movieId")['rating'].agg('count')


new_df = new_df.loc[num_user_voted[num_user_voted > 7].index,:]
new_df = new_df.loc[:,num_movie_voted[num_movie_voted > 30].index]

new_df.head()

#removing sparsity to save space

csr_data = csr_matrix(new_df.values)
new_df.reset_index(inplace = True)

new_df.head()

#using knn model for movie reccomendations 

# using the cosine similarity(used most popularily in these kinds of cases)

model = NearestNeighbors(metric = "cosine",algorithm = "brute", n_neighbors = 25)
model.fit(csr_data)

def get_movie_recommendation(movie_name):
    n_movies_to_reccomend = 10
    movie_list = movies[movies['title'].str.contains(movie_name)]  
    if len(movie_list):        
        movie_idx = movie_list.iloc[0]['movieId']
        movie_idx = new_df[new_df['movieId'] == movie_idx].index[0]
        distances , indices = model.kneighbors(csr_data[movie_idx],n_neighbors=n_movies_to_reccomend+1)    
        rec_movie_indices = sorted(list(zip(indices.squeeze().tolist(),distances.squeeze().tolist())),key=lambda x: x[1])[:0:-1]
        recommend_frame = []
        for val in rec_movie_indices:
            movie_idx = new_df.iloc[val[0]]['movieId']
            idx = movies[movies['movieId'] == movie_idx].index
            recommend_frame.append({'Title':movies.iloc[idx]['title'].values[0],'Distance':val[1]})
        df = pd.DataFrame(recommend_frame,index=range(1,n_movies_to_reccomend+1))
        return df
    else:
        return "No movies found. Please check your input"

get_movie_recommendation("Matrix")

