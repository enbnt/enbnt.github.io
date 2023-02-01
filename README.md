This project makes use of Hugo and the [gokarna](https://github.com/526avijitgupta/gokarna) theme.

If you run into issues generating the site and see errors due to missing themes, the themes are
loaded via Git Submodules. Try to run

```
$ git submodule update --init --recursive
```

You can update a theme via

```
$ git submodule update --remote --merge
```

You can serve the website locally or via a GitHub codespace by running

```
$ hugo serve --baseURL=""
```

If you are in a GitHub codespace, use

```
 hugo serve --baseURL="https://<CODESPACE-URL-PORT>.preview.app.github.dev/" --appendPort=false
 ```

 Where the base url is the codespace URL from the PORTS tab.