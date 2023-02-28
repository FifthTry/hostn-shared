# ~/.fastn/sid.json


```json
{
}
```

# <package-root>/.fastn/cw-id.json

# cw-id = ekey(package.id)

# refresh logic: <bucket>/updated.txt

On every successful package upload, an entry will be added to updated.txt file.

```txt
<cw-id>: <sha256 of list file content>
```

We store the hash so HSR need not refresh list file.

When storing the list file in memory, also store the sha256 of the file.

Every min hsr will delta-fetch the udpated.txt file. Every cw-id that has different
sha256 compared to what we have in cache, mark the cw-id as stale.

When the next request comes to a stale `cw-id`, delta fetch the list file, and then
serve it so the next request is gauranteed to contain the last uploaded content.

# how to ensure only one concurrent download of list file?

Multple requests to same domain can come simultaneously. We have to ensure that each
request does not get a cache miss and starts downloading the list file. To do that
we have to use a rw-lock per cw-id.

We also have to ensure the same thing for domain to cw-id lookup. If multiple requests
come for same domain whose cw-id is not yet known, each request handler may try to 
read from s3. We will store a inmemory map of domain to cw-id, and in the map we will
also store a rw-lock.

# what is delta fetch?

You have a file. The file is append only. You can do a range request on the file's
S3 URL passing it the length of file you already have to get the rest of the file.

# cw-id lookup

The key would `cw-id` instead of package/domain name. 

Request will contain domain, and domain has to be mapped to cw-id, by looking at 
<bucket>/<domain>.txt which will contain the `cw-id`.


