# Only allow trusted sites

* It may seem obvious but origins specified in the `` Access-Control-Allow-Origin`` header should only be sites that are trusted. In particular, dynamically reflecting origins from cross-origin requests without validation is readily exploitable and should be avoided. *