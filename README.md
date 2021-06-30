Concept:

Feed a list of Spotify Playlist URIs to the program. The program accesses these playlists, retrieves data, formats it into a dataframe, and stores it as a csv file. This csv file can then be used as the source for data visualizations in Google Data Studio (PowerBI, Excel, Tableau, Domo, etc) for reports and dashboards. Running the program daily appends the newest data to the bottom of the csv and gives ability to monitor trends over time.

Sections of Code:

1. Spotify API Authorization: Code to access Spotify API. Note: I referred to this site to assist with formatting this section: https://stmorse.github.io/journal/spotify-api.html

2. Lists of Playlist URIs: 2 lists.
        ccm_playlists: Short list for testing code quickly.
        spotify_ccm_playlists: Long list as data source. Takes a few minutes to run.

3. Functions: Six functions.

    First five functions used to gather data.
    
    Final function is the only one to run. It initiates previous functions as needed.
    
        1. "Playlist_getter": takes URI list. Returns “playlist API” endpoint links.
        2. "Playlist_info": Takes links above and gathers data about each playlist
        3. "Artist_info": Takes links above and gathers data about every song in each playlist
        4. "Label_getter": Takes Album URI from “Artist_info” function above, accesses “album API” endpoint to retrieve “record label” for every song in each playlist.
        5. "Artist_followers_getter": Takes Artist URI from “Artist_info” function, accesses “artist API” endpoint to retrieve “Total Followers” for every artist of every song
        6. "Artist_album_df": Initiates all functions above, gathers all data, formats into a dataframe, appends to csv file each time the program runs.

Methodology:

Approach is to iterate through each song in every playlist and append desired datapoints to lists dedicated to that type of datapoint. For example: “Artist Name” for each song in every playlist will be appended to a list containing only Artist Names. “Record Label” for each song will be appended to a list containing only Record Labels. “Total Followers” appended to a list dedicated for that field, and so on.

These lists are then “zipped” together into a new list of “tuples” for each song. Order is retained, so this groups the different data points together by song (e.g. artist name, label, followers, etc). These tuples become the input used by the final function.
The final function is the only one to “print.” It initiates the other functions, accesses the tuples, and formats them into dataframes. Its last step is to merge these dataframes into a single, larger dataframe and save it to a csv file.

LINKS:

Code as a text file: https://github.gatech.edu/pages/dgartley3/data_retriever/Spotify_Playlist_Data_Retriever_Example.txt

Code as a Python file: https://github.gatech.edu/pages/dgartley3/data_retriever/Playlist_Data_Retriever

Example CSV: https://github.gatech.edu/pages/dgartley3/data_retriever/Spotify_Playlist_Data_by_Label_v2.csv
