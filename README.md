# SDK Documentation

This documentation is based on MKDocs documentation generation. Please install it with this command:

```
pip install mkdocs
pip install mike
```


In order to run the documentation server, run this command on your PC terminal:

```
cd sdk-docs
mkdocs serve
```

Or, if you prefer to serve the HTML version of the documentation, run this command on your PC terminal:
```
cd sdk-docs/site
python -m http.server 8000
```

For versioning, use:
```
mike deploy [version] [alias] # for new version
mike delete [version] # for removing specific version 
```
the static versioned file is on gh-pages branch!

If the previous command doesn't show any exceptions, the server should run on the url http://127.0.0.1:8000
