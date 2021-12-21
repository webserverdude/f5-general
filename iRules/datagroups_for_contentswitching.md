# Doku zur content switching iRule

## Datagroups und Funktionen

### dg_applications
Mapping von HTTP::host und HTTP::path zu pool
Key: app01.zerotrust.works/site2
Value: pl_nginx_4000

### dg_link_rewrite
Mapping von HTTP::path zu STREAM expression
Key: /site2/
Value: @siteTwo@site2@
Regex for root /
Key: /demo/
Value: @^\/$@/demo/@

### dg_path-rewrite
Mapping von HTTP::path zu internal Server Path
Key: /site2/
Value: /siteTwo/
Example for host header to path rewrite
Key: app.demo.org
Valule: /app.demo.org/

### dg_hostheader
Mapping von HTTP::path zu internal Host header
Key: /site2/
Value: app02.internal.mydomain.com

### dg_return_path-rewrite
Mapping von intern zu externem Location Header
Key: http:/app02.internal.mydomain.com:4000/siteTwo/
Value: http://app01.mydomain.com/site2/