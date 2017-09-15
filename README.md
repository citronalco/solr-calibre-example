# Calibre solr Web Search

Calibre and Calibre's web server are quite slow for bigger libraries. Searching in a library with a few 10000 books takes several minutes 
But if you index Calibre's metadata in a Apache solr server and use a web site for searching, you get your results within a second.

This an index tool for parsing Calibre's metadata.opf files and store in in solr, the required schema file for solr's index, and a neat website for searching in the index and downloading books.

![screenshot 1](screenshot1.png?raw=true)
![screenshot 2](screenshot2.png?raw=true)

### Installation (on Debian 9):

#### What's where?
- In the "indextool" directory is the indexer, a python script that looks through your Calibre library, parses all metadata.opf files it can find and stuffs the result in the solr server, which again creates the index and answers to queries.
- The "website" directory contains a web site for searching (querying) the solr server and for downloading books.
- The "solr" directory contains a schema.xml file which tells solr how to store what in the index and a solrconfig.xml file, which tells solr where to save the index and how to talk to clients.

#### How to install and configure solr

I assume you already have a webserver up and running. I tested this on Debian 9, I'm pretty sure it works nearly the same way on other operating systems.
Debian 9 comes with solr 3.6.2, if you're using another version same ports in schema.xml and/or solrconfig.xml might need to be changed.

1. Put the content of the "website" directory on your web server (e.g. /var/www/html/ebooksearch/)
1. Configure your webserver to make your Calibre library available in a subdirectory of the website (e.g. in /ebooksearch/calibrelibrary/)
1. Install solr: `apt-get install solr-tomcat`
1. Create a directory `/var/lib/solr/ebooksearch/conf/ and put the content of the "solr" directory into.
1. `chown -R tomcat8:tomcat8 /var/lib/solr/ebooksearch`
1. Edit /etc/solr/solr.xml and add `<core name="ebooksearch" instanceDir="/var/lib/solr/ebooksearch" />` between `<core>` and `</core>`
1. Now map solr/Tomcat's search interface (''http://127.0.0.1:8080/solr/ebooksearch/select'') to /ebooksearch/solr, e.g. by using Apache's mod_proxy module:```<Location /ebooksearch/solr/select>
	ProxyPass http://127.0.0.1:8080/solr/ebooksearch/select
	ProxyPassReverse /ebooksearch/solr/select
    </Location>```
1. Start tomcat8: `systemctl start tomcat8`
1. Edit the website's index.html and set `calibre_url_prefix` to the webserver subdirectory from above (e.g. /ebooksearch/calibrelibrary/) and `solr_url_prefix` to the mapped solr location (e.g. /ebooksearch/solr/select/)
1. Edit the indexer from the "indextool" directory and set the ``BASEDIR`` to point to your calibre library directory on the file system.
1. Start the indexer and wait until it finishes.
1. You might need to reload the solr ebooksearch core to be able to search in the newly created index: either restart tomcat8 or execute `curl "http://localhost:8080/solr/admin/cores?action=RELOAD&core=ebooksearch"`
1. Done. Open the website in a browser and you're supposed to be able to search.

The schema.xml is losely based on the schema of [OPUS4](https://github.com/OPUS4/search), the web interface inspired by [calibre-web](https://github.com/janeczku/calibre-web).
The idea itself is from https://github.com/chrigl/solr-calibre-example
