This is a small config example for how to do exact match on a tokenized field.
The typical use case is to match or boost exact matches over partial matches, such as:
 * Exact match of person name in a whitepages, "John Doe" won't match "John Doe Johnson"
 * Crawling anchor-texts to web-pages. Exact matches much more valueable than partial

The technique is s simple text based field-type which automatically inserts special tokens
at beginning and end of the text, both on index and query side. The special tokens may be
anything you like, but they should be unique so you don't get false matches. In this example,
if you index the test "John Doe", the field type will transform that to:
  "ǣ John Doe Ǣ"
This means that when you search "John", you'll not match because "ǣ John Ǣ" is not a match.

The nice thing about this technique, apart from that it's very simple, is that it can be used
directly in eDisMax's "pf" parameter to automatically cause a boost for an exact match.

The attached schema and solrconfig examples show how you can setup a schema with the field type,
a copyField from name -> exactname, and how to use eDisMax query parser to boost exactname over
the normal name field.

The example can be tested right away, if you have Solr v>=3.4:
1. Download and unpack Solr 3.4 or newer
2. cd solr/example
3. java -Dsolr.solr.home=path-to-exactmatch-folder -jar start.jar
4. Feed some test documents, e.g. using curl:
   curl "http://localhost:8983/solr/update/csv?separator=,&commit=true" -H "Content-Type: text/plain" --data-binary @names.csv
   Or use post.jar if you don't have curl:
   java -Durl=http://localhost:8983/solr/update/csv -Dtype=text/csv -jar path-to-post.jar names.csv
5. Query for "John Doe" and see that the exact one ends up on top
   http://localhost:8983/solr/select/?q=John%20Doe