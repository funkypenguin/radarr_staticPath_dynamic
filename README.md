# radarr_staticPath_dynamic
API commands to update staticPath of all movies from 'static' to 'dynamic'


So one of the cool alpha features (as of current version 0.2.0.1120) is to allow renaming of the movie directory. You can change the default to be 'dynamic' in 'Settings > Media Management > Advanced > Automatically Rename Folders' to 'yes', but it does not change existing movies. I cobbled together some API stuff to pull the existing movie data, update the json output, and then PUT it back.


```
#variables
HOST="radarr_fqdn"
PORT="radarr_port"
APIKEY="radarr_apikey"
TMPDIR="/tmp/radarr/"

#jq package required for manipulating json output
yum install -y jq

#create temp directory
mkdir -p $TMPDIR

#gather list of ids, then pull json output, then change pathState to dynamic
for id in $(curl --request GET "http://$HOST:$PORT/movies/api/movie?apikey=$APIKEY" | jq '.[] | .id'); 
do 
 curl --request GET "http://$HOST:$PORT/movies/api/movie/id=$id?apikey=$APIKEY" \
 | sed 's/"pathState": "static"/"pathState": "dynamic"/' > $TMPDIR/$id
done

#put new data back via api
for idfile in $(ls $TMPDIR/*);
do
 curl --request PUT "http://$HOST:$PORT/movies/api/movie?apikey=$APIKEY" -d @$idfile
done

#cleanup files
rm -rf $TMPDIR
```

The intial script isn't the most efficient and the PUT section will definitely take some time if you have a large library, but much better than editting tons of individual movies!

Then run a mass rename from the UI or via API and enjoy!
