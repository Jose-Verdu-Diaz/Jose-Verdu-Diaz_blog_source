+++
date = "2022-09-18"
title = "Improved Spotify Music Selection with UMAP"
slug = "improved-spotify-music-selection-with-umap"
categories = [ "Post" , "Personal Projects" ]
tags = [ "Spotify", "UMAP", "Python" ]
+++

I'm the kind of Spotify user that adds every single song to one single playlist and then complains about not being able to find what he wants to hear. One simple solution would be grouping songs in different playlists by mood, however, this solution is not exciting enough for me... Instead, I decided to create an app that takes all tracks in a playlist, "analyzes" and groups them by similarity in an interactive 3D graph, where you can click on a song and automatically play it on Spotify. Ok, maybe that was too much to digest, let's decompose the problem:

- **Getting all tracks of a Spotify playlist**: Spotify offers an amazing [Web API][1] with an endpoint for requesting a list of tracks for a given playlist.
- **Analyze the tracks**: I might have lied here... we are not going to analyze and extract the features of the songs, as Spotify does not give access to the audio data. Instead, Spotify generates a set of song features using their :sparkles:*Corporate Secrets*:sparkles: which are accessible with the Web API.
- **Grouping tracks by similarity**: To group similar songs we will follow a [**dimensionality reduction**][2] strategy using the [UMAP][2] technique. We will use [Plotly][4] to create a 3D interactive graph of the UMAP result that will allow us to select individual songs.
- **Playing selected songs in Spotify**: We will use, again, the Spotify Web API to play the song we select in the Plotly graph.

:exclamation: Before moving forward: if you are planning to reproduce this project you might need a **premium Spotify account** to perform some of the API requests. Also, this project assumes you have an intermediate understanding of python, packet managing and other concepts.

With that being said, let's dive in!

---

## Setting up the Spotify Web API requirements

The Web API requests require **authentication** with an [OAuth][5] access token. To obtain a token we first need to log in to [Spotify for Developers Dashboard][6]:

![](/img/spotify_umap/1.png)

Once we are logged in, we will click on "create an app" and a form will appear. We will give an awesome name and an awesome description to our app, and we will accept the Developer Terms of Service and Branding Guidelines after fully reading them :eyes:.

![](/img/spotify_umap/2.png)

![](/img/spotify_umap/3.png)

After accepting we will be redirected to the app's dashboard. There we will be able to find the `Client ID` and `Client Secret`. Note that both are blurred out, **you shouldn't share these values anywhere!** :speak_no_evil:

![](/img/spotify_umap/4.png)

Write these values in a json file named `credentials.json`. If you are using git with your code and uploading the repository anywhere (GitHub, for example) **add the json file to the `.gitignore`!**

{{< highlight json "linenos=false" >}}
{
    "CLIENT_ID": "123456789abcdefghijklmnopqrstuvw",
    "CLIENT_SECRET": "123456789abcdefghijklmnopqrstuvw"
}
{{< /highlight >}}

Click the "EDIT SETTINGS" button in the dashboard and add the following redirect URI: `https://www.google.com/`. This will be used during the authentication process.

![](/img/spotify_umap/5.png)

With this done, we finished setting up the requirements for the Web API.


---

## Getting the data

First of all, we will load the credentials we stored before:

{{< highlight python "linenos=false" >}}
import json

with open('credentials.json', 'r') as f: credentials = json.load(f)
CLIENT_ID = credentials['CLIENT_ID']
CLIENT_SECRET = credentials['CLIENT_SECRET']
{{< /highlight >}}

We will use [spotipy][7], a library makes interacting with the API simpler. We define a `Spotify` object as follows:

{{< highlight python "linenos=false" >}}
import spotipy as spt

SCOPE = 'user-modify-playback-state user-library-read user-read-email user-read-private user-top-read user-modify-playback-state user-read-playback-state'
REDIRECT_URI = 'https://www.google.com/'

auth = spt.SpotifyOAuth(
    client_id=CLIENT_ID, client_secret=CLIENT_SECRET, 
    redirect_uri=REDIRECT_URI, 
    scope=SCOPE, open_browser=True, cache_path='.cache'
)
spotify = spt.Spotify(oauth_manager=auth)
{{< /highlight >}}


{{< load-plotly >}}
{{< plotly json="/plotly/umap.json" height="800px">}}


[1]: https://developer.spotify.com/documentation/web-api/
[2]: https://en.wikipedia.org/wiki/Dimensionality_reduction
[3]: https://arxiv.org/abs/1802.03426
[4]: https://plotly.com/python/
[5]: https://oauth.net/
[6]: https://developer.spotify.com/dashboard/
[7]: https://spotipy.readthedocs.io