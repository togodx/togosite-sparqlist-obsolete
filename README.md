# togosite-sparqlist
SPARQList for TogoSite

Sparqlet markdown files are located under the repository directory.

## install and run

1. git clone https://github.com/biosciencedbc/togosite-sparqlist/
2. cd togosite-sparqlist
3. git clone https://github.com/dbcls/sparqlist
4. Add ENV ROOT_PATH=/togosite/sparqlist/ to sparqlist/Dockerfile
5. Place a file named togosite.env under togosite-sparqlist
6. Set ADMIN_PASSWORD=&lt;&lt;PASSWORD&gt;&gt; to togosite.env
7. Set VERSION and SPARQLIST_PORT in .env
8. Run docker-compose up -d. Building a sparqlist image takes around 20 minutes.
9. Run docker-compose ps to confirm if the status is up
```
[mitsuhashi@smp05 sparqlist20210322]$ docker-compose ps
            Name                           Command               State            Ports
------------------------------------------------------------------------------------------------
sparqlist20210322_sparqlist_1   docker-entrypoint.sh /bin/ ...   Up      0.0.0.0:11001->3000/tcp
```
10. Access http://localhost:11001/togosite/sparqlist when SPARQList_PORT is 11001
```
[mitsuhashi@smp05 sparqlist20210322]$ curl -L http://localhost:11001/togosite/sparqlist
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>SPARQList</title>
    <meta name="description" content="">
    <meta name="viewport" content="width=device-width, initial-scale=1">


<meta name="sparqlist/config/environment" content="%7B%22modulePrefix%22%3A%22sparqlist%22%2C%22environment%22%3A%22production%22%2C%22rootURL%22%3A%22%2Ftogosite%2Fsparqlist%2F%22%2C%22locationType%22%3A%22auto%22%2C%22EmberENV%22%3A%7B%22FEATURES%22%3A%7B%7D%2C%22EXTEND_PROTOTYPES%22%3A%7B%22Date%22%3Afalse%7D%2C%22_APPLICATION_TEMPLATE_WRAPPER%22%3Afalse%2C%22_DEFAULT_ASYNC_OBSERVERS%22%3Atrue%2C%22_JQUERY_INTEGRATION%22%3Afalse%2C%22_TEMPLATE_ONLY_GLIMMER_COMPONENTS%22%3Atrue%7D%2C%22APP%22%3A%7B%22name%22%3A%22sparqlist%22%2C%22version%22%3A%220.0.0%2Bc4690ece%22%7D%2C%22ember-simple-auth%22%3A%7B%22routeAfterAuthentication%22%3A%22sparqlets%22%7D%2C%22exportApplicationGlobal%22%3Afalse%7D" />

    <link integrity="" rel="stylesheet" href="/togosite/sparqlist/assets/vendor-18a0ccf26629e199a3e54c43703042f5.css">
    <link integrity="" rel="stylesheet" href="/togosite/sparqlist/assets/sparqlist-0505cc8f6bf91a8e65638ad59b7271fa.css">


  </head>
  <body>


    <script src="/togosite/sparqlist/assets/vendor-fc91a35b5cbc4ca8e5f3af8d4ffaca94.js"></script>
    <script src="/togosite/sparqlist/assets/sparqlist-2e18be7359041c2b25ced02a8c26d84e.js"></script>


  </body>
</html>
```

## scheduled backup and service availability monitoring

See the cron jobs below.
```
[mitsuhashi@smp05 sparqlist20210322]$ cat /etc/cron.d/sparqlist-for-togosite
# scheduled rsync backup to NAS
0 3 * * * root /ssd/togosite/scripts/sparqlist-togosite-integbio.jp-backup > /dev/null
# scheduled git push
0 2 * * * root /ssd/togosite/scripts/sparqlist-togosite-integbio.jp-github-push > /dev/null
# service availability monitoring
*/5 * * * * root /ssd/togosite/scripts/sparqlist-togosite-integbio.jp-monitor > /dev/null
[mitsuhashi@smp05 sparqlist20210322]$
```
