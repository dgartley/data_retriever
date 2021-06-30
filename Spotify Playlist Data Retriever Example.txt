### ADDITIONAL PYTHON LIBRARIES USED ########################################
import requests
import json
import pandas as pd
import csv
import timeit


### SPOTIFY API AUTHORIZATION ###############################################
login_url = 'https://accounts.spotify.com/authorize' #login url per spotify docs


client_id = ' ' #id provided by creating spotify app on spotify for developers site
client_secret = ' ' #secret code provided by creating spotify app on spotify for developers site
auth_url = 'https://accounts.spotify.com/api/token' #spotify url that checks my id and secret code


### Variable using 'posts' method of 'requests' package to access API. Compiles necessary auth info into dictionary format.
auth_response = requests.post(auth_url, {
    'grant_type' : 'client_credentials', 
    'client_id' : client_id, 
    'client_secret' : client_secret
})

auth_response_data = auth_response.json() #converts auth_response variable to json format
access_token = auth_response_data['access_token'] # 
headers = {'Authorization' : 'Bearer {token}'.format(token = access_token)}



### LISTS OF PLAYLIST URI'S USED BY FUNCTIONS TO RETRIEVE DATA ##############

# List for testing/debugging - 2 playlists. Shorter run time 
ccm_playlists = ['37i9dQZF1DXcb6CQIjdqKy', '6MOsesb7nyoCk1jXPTKG8F'] 


# List of 50+ Spotify-owned playlists in US labelled with a "Christian" genre.
# Retrieved from: https://everynoise.com/worldbrowser.cgi?section=inspirational&hours=0
spotify_ccm_playlists = ['37i9dQZF1DWYhr4P5Boce5', '37i9dQZF1DX0N57moxx9BL', '37i9dQZF1DX82qPOvdCxxq', '37i9dQZF1DX3ZaylsK87YU', '37i9dQZF1DWUEbH7oMQunS', \
                         '37i9dQZF1DWVtgG63SDdt8', '37i9dQZF1DX2sJGkrvCPgm', '37i9dQZF1DX00QnRFT3a0O', '37i9dQZF1DXcb6CQIjdqKy', '37i9dQZF1DWVYgpMbMPJMz', \
                             '37i9dQZF1DX5SzTPIoCKiv', '37i9dQZF1DX35T8tss1Gxt', '37i9dQZF1DWU2LcZVHsTdv', '37i9dQZF1DX7Bi6W3YuUlA', '37i9dQZF1DXc4BD3pzYdKY', \
                                 '37i9dQZF1DWZoR0U5SzE1r', '37i9dQZF1DX7OIddoQVdRt', '61xuizm8At6DCwGHJZTmB7', '37i9dQZF1DX0IXk7nnh7Jx', \
                                     '37i9dQZF1DXdrxKdrXE2Vk', '37i9dQZF1DXbQ1kpdsa9FU', '37i9dQZF1DXdgckExLlG1g', '37i9dQZF1DWUMIjnZuaulx', \
                                         '37i9dQZF1DX4levbzTG2FX', '37i9dQZF1DWVlWpJblBvap', '37i9dQZF1DWUileP28ODwg', '37i9dQZF1DWUSDYg6JTKFA', \
                                             '37i9dQZF1DWUjxqgjSiQ9K', '37i9dQZF1DWYcaB2B11tq2', '37i9dQZF1DXaJDplEJVP32', '37i9dQZF1DWTkyVpP5GNAO', \
                                                 '37i9dQZF1DX5pEiFLSS7sX', '37i9dQZF1DWZCJsgK4Sw8Y', '37i9dQZF1DWVrSccL9KVUt', '37i9dQZF1DX7fPhUztqfIX', \
                                                     '37i9dQZF1DWVh9guDyUECQ', '37i9dQZF1DWZ2ShEhw72H5', '37i9dQZF1DX3XfcfEnrDRE', '37i9dQZF1DWUUPO0Sbx2CM', \
                                                         '37i9dQZF1DWZnA0FshBt4S', '37i9dQZF1DWX1VaLD1v09s', '37i9dQZF1DWXKWi9FunemC', '37i9dQZF1DX0gF89fU9TTk', \
                                                             '37i9dQZF1DXbK4SCc53BQf', '37i9dQZF1DX0OG4PTOnovS', '37i9dQZF1DX4kqCz1xEn1w', '37i9dQZF1DWW37fHr0rhOh', \
                                                                 '37i9dQZF1DWZIzeeN1t2Mn', '37i9dQZF1DWUzcqvqvxPQA', '37i9dQZF1DX4nGm7tTyA78', '37i9dQZF1DWTSWzScBKziy',]

print("START") # Starting point 


### FUNCTION 1 ###
### ITERATES THROUGH LIST OF SPOTIFY PLAYLIST URI'S ABOVE.
### FORMATS API ENDPOINT LINKS TO RETRIEVE DATA IN UPCOMING FUNCTIONS.

def playlist_getter(playlist_uri_list, headers = headers): 
    playlist_list = []
    base_url = 'https://api.spotify.com/v1/playlists/'
    for uri in playlist_uri_list:
        get_request = requests.get(base_url + '{playlisturi}'.format(playlisturi = uri), \
                                   headers = headers, params = {'include_groups' : 'total'})
        playlist_json = get_request.json()
        playlist_list.append(playlist_json)
    return playlist_list
    
#print(playlist_getter(ccm_playlists, headers)) 
# Remove hashtag above to run function alone. For testing/debugging purposes
    

# Converts playlist_getter function into a variable. For shorter scripts below.
ccm = playlist_getter(ccm_playlists, headers = headers) 
# change "ccm_playlists" to "spotify_ccm_playlists" for the larger dataset. 
# Note: last function below is not currently set to save to csv. It will only show a condensed dataframe.


### FUNCTION 2 ###
### TAKES LIST OF PLAYLIST URIS. RETURNS 5-TUPLE:
### (PLAYLIST NAME, URI, OWNER, SONG COUNT, TOTAL FOLLOWERS) 

def playlist_info(playlist_getter):     
    playlist_name_list = [] # list of playlist names iterated through
    playlist_uri_list = [] # list of playlist uri's iterated through
    playlist_owner_list = [] # list of playlist owner name for each playlist 
    playlist_songcount_list = [] # list of how many songs are in each playlist
    playlist_followers_list = [] # list of total followers for a playlist
    
# Iterates through playlists. Retrieves data. Appends to lists above.    
    for playlist in playlist_getter:
        playlist_name_list.append(playlist['name']) # 
        playlist_uri_list.append(playlist['uri'][17:])
        playlist_owner_list.append(playlist['owner']['display_name'])
        playlist_songcount_list.append(playlist['tracks']['total'])
        playlist_followers_list.append(playlist['followers']['total'])

# Combines separated lists above into elements of a tuple.
# Returns a list of tuples with 5 fields of data in each.
    return list(zip(playlist_name_list, playlist_uri_list, playlist_songcount_list, playlist_owner_list, playlist_followers_list, ))


#print(playlist_info(ccm)) 
# Remove hashtag above to run function alone. For testing/debugging purposes


### FUNCTION 3 ###
### TAKES LIST OF PLAYLIST URI'S. RETURNS A 2-TUPLE: (DATA, ERRORS)
### 1ST TUPLE ELEMENT: LIST OF 10-TUPLES:
### (SONG ORDER INDEX, ARTIST NAME, ARTIST URI, SONG NAME, SONG URI, POPULARITY #, DATE ADDED, ALBUM NAME, ALBUM URI, PLAYLIST INFO 5-TUPLE)
### 2ND TUPLE ELEMENT: FOR DEBUGGING - LIST OF ERROR CAUSING PLAYLIST "ITEMS" - USUALLY ITEMS CONTAINING NO DATA

def artist_info(playlist_getter):
# Lists to store data retrieved from playlist uri's:
    playlist_song_index_list = []
    artist_name_list = []
    artist_uri_list = []
    album_name_list = []
    album_uri_list = []
    song_name_list = []
    song_uri_list = []
    added_at_list = []
    popularity_list = []
    playlist_list = []
    error_list = []
    
    song_sum_counter = 0 # sums songs in batches by playlist length - for debugging, monitoring progress
    song_counter = 0 # running song counter - for debugging/monitoring progress
    playlist_song_index = 0 # Used to retain song order for each playlist
    playlist_item_counter = 0 # used to iterate through playlist_getter fx inside artist_info fx iterations
    
# Iterates through playlist json generated by playlist_getter fx
    for playlist in playlist_getter:
        playlist_song_index = 0
        song_sum_counter += playlist['tracks']['total']

# connects to correct dictionary items to pull data:
        for item in playlist['tracks']['items']: # 'item' in spotify_json contains data for each SONG in the playlist
            playlist_song_index += 1
            if item['track']: # if statement used to separate data from errors

# appends appropriate data to corresponding lists above
                playlist_song_index_list.append(playlist_song_index)
                artist_name_list.append(item['track']['artists'][0]['name'])
                artist_uri_list.append(item['track']['artists'][0]['uri'][15:])
                song_name_list.append(item['track']['name'])
                song_uri_list.append(item['track']['uri'][14:])
                added_at_list.append(item['added_at'])
                popularity_list.append(item['track']['popularity'])
                album_name_list.append(item['track']['album']['name']) 
                album_uri_list.append(item['track']['album']['uri'][14:])
                playlist_list.append(playlist_info(playlist_getter)[playlist_item_counter])
                print('artist_info:', song_counter, item['track']['artists'][0]['name'], item['track']['name']) # monitors progress when running code
                song_counter += 1
            else:
                error_list.append(item)
        playlist_item_counter += 1
        
    
# combines 10-tuple and error_list into a 2-tuple
    data_error_tupl = (list(zip(playlist_song_index_list, artist_name_list, artist_uri_list, song_name_list, song_uri_list, popularity_list, added_at_list, album_name_list, album_uri_list, playlist_list)), error_list)
    
    return data_error_tupl # returns the fully-combined 2-Tuple
# Note all data is stored in the first element of the tuple.
# When calling function for data, request only 1st element - "ccm[0]"
# when calling function for errors, request 2nd element - "ccm[1]" 

#print(artist_info(ccm)[0]) 
# Remove hashtag above to run function alone. For testing/debugging purposes


### FUNCTION 4 ###
### TAKES ALBUM URI COLLECTED FROM ARTIST_INFO FUNCTION ABOVE. RETRIEVES RECORD LABEL DATA
### RETURNS LIST OF 3-TUPLES FOR EACH SONG IN PLAYLISTS:
### ({ARTIST:URI}, ALBUM NAME, RECORD LABEL) ### dictionary to debug/assert correct data is being pulled

def label_getter(artist_info):
    label_counter = 0 # for debugging/monitoring progress

# formats incoming data for use in function
    album_uri_tupl = []
    for tupl in artist_info:
        album_uri_tupl.append((tupl[7], tupl[8]))

# Lists to store data retrieved from ALBUM ENDPOINT (not playlist endpoint)
    artist_list = []
    album_list = []
    label_list = []
    album_error_list = []

# Iterates through newly formated list above. 
# Connects to correct album endpoint for each song in playlist
    for uri in album_uri_tupl:
        album_url = 'https://api.spotify.com/v1/albums/{albumuri}/'.format(albumuri = uri[1])
        album_info = requests.get(album_url, headers = headers, params = {'include_groups' : 'total'})
        album_info = album_info.json()

# appends appropriate data to corresponding lists above
        if album_info['artists'][0]['name'] and album_info['artists'][0]['uri']: # if statement used to separate data from errors
            print('label_getter:', label_counter, album_info['artists'][0]['name'], album_info['artists'][0]['uri']) # for debugging/monitoring progress
            label_counter += 1
            artisturi_tupl = (album_info['artists'][0]['name'], album_info['artists'][0]['uri'][15:])
            artist_list.append(artisturi_tupl)
            album_list.append(album_info['name'])
            label_list.append(album_info['label'])
        else:
            album_error_list.append(album_info)

# combines data into 3-tuple   
    label_info = list(zip(artist_list, album_list, label_list))
    return label_info 

#print(label_getter(artist_info(ccm)[0])) 
# Remove hashtag above to run function alone. For testing/debugging purposes



### FUNCTION 5 ###
### TAKES ARTIST URI COLLECTED FROM ARTIST_INFO FUNCTION ABOVE. RETRIEVES ARTIST DATA
### RETURNS LIST OF 2-TUPLES FOR EACH SONG IN PLAYLISTS:
### ({ARTIST:URI}, ARTIST NAME, ARTIST FOLLOWERS) ### dictionary to debug/assert correct data is being pulled
def artist_followers_getter(artist_info):
    uri_counter = 0 # for debugging/monitoring progress

# formats incoming data for use in function
    artist_uri_tupl = []
    for tupl in artist_info:
        artist_uri_tupl.append((tupl[1], tupl[2]))

# Lists to store data retrieved from ARTIST ENDPOINT (not playlist or album endpoint)
    artist_list = []
    follower_list = []
    artist_error_list = []
    
# Iterates through newly formated list above. 
# Connects to correct artist endpoint for each song in playlist
    for uri in artist_info:
        artist_url = 'https://api.spotify.com/v1/artists/{artisturi}/'.format(artisturi = uri[2])
        artist_info = requests.get(artist_url, headers = headers, params = {'include_groups' : 'total'})
        artist_info = artist_info.json()

# appends appropriate data to corresponding lists above
        if artist_info['name'] and artist_info['uri']:
            print('artist_followers_getter:', uri_counter,  artist_info['name'], artist_info['uri'])
            uri_counter += 1
            artisturi_tupl = (artist_info['name'], artist_info['uri'][15:])
            artist_list.append(artisturi_tupl)
            follower_list.append(artist_info['followers']['total']) #artist name and followers
        else:
            artist_error_list.append(artist_info)

# combines data into 2-tuple   
    follower_info = list(zip(artist_list, follower_list))
    return follower_info #, len(artist_list), len(follower_list), len(artist_error_list)

#print(artist_followers_getter(artist_info(ccm)[0]))
# Remove hashtag above to run function alone. For testing/debugging purposes


### FUNCTION 6 ###
### CALLS ALL PREVIOUS FUNCTIONS.
### COMPILES DATA INTO DATAFRAME & ADDS A DATETIME COLUMN.
### APPENDS TO CSV

def artist_album_df(artist_info, label_getter, artist_followers_getter):
# formatting data from previous functions for use in this function
    artist_name_tupl = []
    song_name_tupl = []
    album_name_tupl = []
    playlist_tupl = []
    playlist_song_index = []
    
    for song in artist_info:
        artist_name_tupl.append((song[1], song[2]))
        song_name_tupl.append((song[3], song[4], song[5], song[6]))
        album_name_tupl.append((song[7], song[8]))
        playlist_tupl.append((song[9][0], song[9][1], song[9][2], song[8][3], song[9][4]))
        playlist_song_index.append(song[0])

# converts lists above into dataframes. Print below statements are to monitor progress
    artist_df = pd.DataFrame(artist_name_tupl) #, columns = ['Artist Name', 'Artist URI'])
    print('artist_df: \n')
    print(artist_df)
    song_df = pd.DataFrame(song_name_tupl) #, columns = ['Song Name', 'Song URI', 'Song Popularity', 'Added to playlist']
    print('song_df: \n')
    print(song_df)
    album_df = pd.DataFrame(album_name_tupl) #, columns = ['Album Name', 'Album URI'])
    print('album_df: \n')
    print(album_df)
    playlist_df = pd.DataFrame(playlist_tupl) #, columns = ['Playlist Name', 'Playlist URI', 'Playlist Length', 'Playlist Owner', 'Playlist Followers']
    print('playlist_df: \n')
    print(playlist_df)
    label_df = pd.DataFrame(label_getter) #, columns = ['Artist URI Dict', 'Album Name', 'Record Label (Album)'])
    print('label_df: \n')
    print(label_df)
    followers_df = pd.DataFrame(artist_followers_getter) #, columns = ['Artist URI Dict', 'Total Spotify Followers'])
    print('followers_df: \n')
    print(followers_df)
    playlist_song_index_df = pd.DataFrame(playlist_song_index) #, columns = ['Playlist Song Index'])
  
 
    pd.set_option('max_columns', None) # see all columns in printout

# Merge all dataframes into 1 final data frame using indexes as keys
    new_df = artist_df.merge(song_df, left_index = True, right_index = True)
    new_df = new_df.merge(label_df, left_index = True, right_index = True)
    new_df = new_df.merge(followers_df, left_index = True, right_index = True)
    new_df = new_df.merge(album_df, left_index = True, right_index = True)
    new_df = new_df.merge(playlist_df, left_index = True, right_index = True)
    new_df = new_df.merge(playlist_song_index_df, left_index = True, right_index = True)
    new_df['Last Refresh'] = pd.to_datetime('now')
    new_df['Artist Name Count'] = new_df.groupby('Artist Name')['Last Refresh'].transform('count')
    new_df['Label Name Count'] = new_df.groupby('Record Label (Album)')['Last Refresh'].transform('count')  
    new_df['Track Name Count'] = new_df.groupby('Song Name')['Last Refresh'].transform('count')

    
# Remove hashtags and add file path below to save to save to csv. 
#    new_df.to_csv('### FILE PATH ON LOCAL MACHINE.csv', \
#                  header = False, index = False, mode = 'a')
    return new_df
    
print(artist_album_df(artist_info(ccm)[0], label_getter(artist_info(ccm)[0]), artist_followers_getter(artist_info(ccm)[0])))


print('FINISHED') # To indicate the program has finished.