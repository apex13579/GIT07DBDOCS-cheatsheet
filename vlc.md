|purpose|command|
|---|---|
|play youtube in vlc|streamlink "https://www.youtube.com/watch?v=VIDEO_ID" best --player vlc|
|integration with vlc|vlc $(yt-dlp -f best -g "https://www.youtube.com/watch?v=VIDEO_ID")|
