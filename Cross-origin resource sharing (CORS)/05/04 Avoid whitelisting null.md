# Avoid whitelisting null

* Avoid using the header ``Access-Control-Allow-Origin: null``. Cross-origin resource calls from internal documents and sandboxed requests can specify the null origin. CORS headers should be properly defined in respect of trusted origins for private and public servers. *