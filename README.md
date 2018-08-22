# radarr_staticPath_dynamic
API commands to update staticPath of all movies from 'static' to 'dynamic'


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
